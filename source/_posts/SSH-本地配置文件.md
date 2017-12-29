---
title: SSH 本地配置文件
date: 2017-10-18 17:57:25
tags: [" SSH "," Linux "
categories: " 运维 "
---

> ** 前言 **
　在扩展节点的时候又需要修改 ssh 的配置文件，不禁让我想起我这电脑上最最宝贵的莫过于我的 `hosts` 与 `./ssh/config` 文件了，由此想记录一下我如何配置我的 ssh 配置文件的

## SSH 配置

### 需求

在我的认识中若是有一下两个需求的同学，建议配置你的 ssh config，这样可以给你省不少的麻烦：

- 要想在一台电脑上使用不同的 github 账号；
- 需要登陆多个服务器但是配置的端口、key 不同

既然都要讲本地的 ssh 配置，那就把服务端建议配置与创建密钥一起讲了。

### 服务器端配置

服务器端的配置文件位于 `/etc/ssh/sshd_config`。

```bash
Port 22  # 默认端口建议修改，并且建议设置的值高一些
PasswordAuthentication no  # 不允许任何的密码登陆，统一都用密钥
PermitRootLogin no # 不允许 root 账户登陆
PermitEmptyPasswords no #不允许空密码登陆
AllowUsers # 白名单（可以不设置），如果白名单有用户只有白名单的用户可以登陆
DenyUsers  # 黑名单（可以不设置），被拒绝的用户，如果即允许又拒绝则拒绝生效
```

一般重要的服务器我都会修改端口号，因为我们能登录服务器的人并不多，创建的账户也不多所以一般我不用白名单与黑名单，其他三项是必备的。

### 生成密钥

生成密钥就比较简单了，一个命令的问题：

```
ssh-keygen -t rsa -b 4096
```

然后就会问你创建的密钥要放在那里，使用密钥的时候密码是什么。

以前的我这里会不输入密码但是现在的我会输入一个简单的密码，这样即使我的密钥不小心被偷了，也比没有密码的好，虽然没有无密码的爽，但是心里稍微踏实一些。

### 客户端配置

本地的 ssh 配置一般会放在家目录的 `.ssh/config` 里面。

一般常用的配置项有：

```bash
HOST：配置机器的别名就是你 ssh 后面需要跟的名字，若是 * 的话其下面的配置对所有的机器都生效
	HostName: 配置机器的域名或者是 IP 地址，我一般 IP 地址写在 host 里面，然后这里写别名
	User：登录的用户名
	Port：登录机器的端口，默认使用的是 22，这里可以特定设置
	preferredauthentications：登录认证的方式选择，有这样的一些选项hostbased(在SSH-2中考虑Rhosts存在安全漏洞，废弃了这种方式), publickey（公钥）, keyboard-interactive（交互式）, password（直接输入密码）
	identityfile:认证公钥的路径
	ServerAliveInterval：设置如果在多少秒之后没有接受到服务端的数据就 timeout，断开链接
```

例如：

```bash
 Host test
    HostName test
    User test
    preferredauthentications publickey
    identityfile ~/key/weizj
    ServerAliveInterval 30

```

若是配置 github 的多账号管理的话主要就是为了用不同的 key，所以区别在于别名与密钥，例如：

```bash
Host richardwei1
    User git
    Hostname github.com
    identityfile ~/key/richardwei1


Host richardwei2
    User git
    Hostname github.com
    identityfile ~/key/richardwei2
```

配置之后只需要在相关的项目中 `.git/config` 里面的 `github.com` 更换成我们的 Host 的别名就可以。例如：

```bash
git@richardwei1:richardwei1/richardwei1.git
```
