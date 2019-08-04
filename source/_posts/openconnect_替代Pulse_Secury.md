---
title: openconnect 替代 Pulse Secury
date: 2019-04-19 13:20:33
tags: " Linux "
categories: " 运维 "
---

> ** 前言 **
　公司用的 Pulse Secury 做 vpn，但是在 Linux 中已经不支持浏览器中嵌套 Java 的方式了，并且客户端也放弃升级了所以导致 Linux 中无法直接使用的 client，因此通过 openconnect 来替代

## 使用 openconnect

openconnect 是一个开源工具，我们选择它来替换 Pulse Secury

- 首先安装 openconnect

```bash
# 注意 openconnect v7.06 有 bug，务必安装 7.08 及以上的版本

sudo apt-get install openconnect # Ubuntu
sudo yum install openconnect # Centos
```

- 下载 vpnc 脚本（vpnc-script 作用 To set the routing and name service up, 单独下载是因为老版本少了一个监控什么的）

```bash
mkdir -p /etc/vpnc/
curl http://git.infradead.org/users/dwmw2/vpnc-scripts.git/blob_plain/HEAD:/vpnc-script > /etc/vpnc/vpnc-script
chmod a+x /etc/vpnc/vpnc-script
```

- 然后启动 vpn:

```bash
openconnect --juniper --script /etc/vpnc/vpnc-script https://vpn.xxxx.xxx

# 运行结果
WARNING: Juniper Network Connect support is experimental.
It will probably be superseded by Junos Pulse support.
GET https://vpn.xxxx.xxx/
Connected to xxx.xxx.xxx.xxx:443
SSL negotiation with vpn.xxxx.xxx
Server certificate verify failed: signer not found

Certificate from VPN server "vpn.xxxx.xxxx" failed verification.
Reason: signer not found
To trust this server in future, perhaps add this to your command line:
Enter 'yes' to accept, 'no' to abort; anything else to view: yes
Connected to HTTPS on vpn.xxxx.xxxx
Got HTTP response: HTTP/1.1 302 Found
GET https://vpn.xxxxx.xxxx/dana-na/auth/url_default/welcome.cgi
SSL negotiation with vpn.xxxx.xxx
Server certificate verify failed: signer not found
Connected to HTTPS on vpn.xxxx.xxx
frmLogin
realm [MOTP动态令牌登陆|传统登陆]:MOTP动态令牌登陆
frmLogin
username:xxx
password:

```

- ubuntu 插件可以帮我们在 Network Manager 中简单配置即可，不用每次都运行命令(支持 openconnect、openfortivpn 等等)

```bash
# network-manager-openconnect-gnome 这个似乎也行，没用过
sudo apt-get install network-manager-fortisslvpn
```

> 注意：
> 若是用到老版本的 openconnect 会有这样的报错

```bash
Connected to HTTPS on {{myserver}}
Got HTTP response: HTTP/1.1 400 Bad Request
Unexpected 400 result from server
Creating SSL connection failed
```

在同事一台服务器（ubuntu16.04 系统）上因为找不到 v7.08 的安装包所以只能尝试编译安装，遇到了这样的问题:

```bash
# 这个问题主要是还是因为缺少依赖导致的
# ./autogen.sh 
autoheader: warning: missing template: LIBPROXY_HDR
autoheader: Use AC_DEFINE([LIBPROXY_HDR], [], [Description])
```

编译安装的流程:

```bash
# 安装依赖
sudo apt-get install build-essential gettext autoconf automake libproxy-dev libxml2-dev libtool vpnc-scripts pkg-config libgnutls-dev

# 下载源码
git clone git://git.infradead.org/users/dwmw2/openconnect.git

# 下载最新的脚本
mkdir -p /etc/vpnc/
curl http://git.infradead.org/users/dwmw2/vpnc-scripts.git/blob_plain/HEAD:/vpnc-script > /etc/vpnc/vpnc-script
chmod a+x /etc/vpnc/vpnc-script

# 编译安装
cd openconnect/
git checkout v7.08
sudo ./autogen.sh
sudo ./configure
sudo make && sudo make install

# 然后检查版本即可
sudo ./openconnect --version
```

## 参考

- 官网： https://www.infradead.org/openconnect/packages.html
- vpnc-script 说明: http://www.infradead.org/openconnect/vpnc-script.html
- 低版本问题: https://serverfault.com/questions/835918/openconnect-and-pulse-stopped-working
- 安装说明: https://github.com/dlenski/openconnect#building-from-source-on-linux
