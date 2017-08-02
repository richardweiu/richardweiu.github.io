---
title: pptp 启动失败
date: 2017-08-02 21:26:41
tags: " pptp "
categories: " 运维 "
---

> ** 前言 **
　工作一年了一直没有写博客的习惯，一共有两个原因：
- 感觉写博客太花时间的，因为每次都要查很多资料，准备的很充分，想写的很详细；
- 偷懒一两次之后需要总结的内容就越堆越多，后来写总想补之前的，但是太多了就放弃了；
- 太懒了。
　今天又开始写博客是因为被同事问到一个曾经遇到过的问题，但是忘记怎么解决了，查资料又半天没有查到，所以又决定开始写。无论大、小问题，简单记录即可

## PPTP 问题

### 简介

PPTP（Point to Point Tunneling Protocol），即点对点隧道协议。该协议是在PPP协议的基础上开发的一种新的增强型安全协议，支持多协议虚拟专用网（VPN），可以通过密码验证协议（PAP）、可扩展认证协议（EAP）等方法增强安全性。可以使远程用户通过拨入 ISP、通过直接连接 Internet 或其他网络安全地访问企业网。（此段来自[百度百科](https://baike.baidu.com/item/PPTP)）

### 启动出错

之前有位同事准备写一个关于搭建虚拟专用网的教程，本来打算使用 pptp 协议来做演示，哪知在 ubuntu 中启动时遇到这样的问题：

使用这样的命令启动 pptp 服务：

```shell
sudo service pptpd start
```

启动完之后自以为启动成功，便继续操作，哪知后面的操作失败，返回来用 `service` 命令检查服务的状态：

```shell
sudo service pptpd status
```

却得到这样的返回： `*/usr/sbin/pptpd is not running`

原来这是打包的时候 pptp 的启动脚本写漏了一个参数导致的（[launchpad 说明页](https://bugs.launchpad.net/ubuntu/+source/pptpd/+bug/1296835)）：

```shell
The "status" target in /etc/init.d/pptpd doesn't work, because the call to status_of_proc lacks a "-p" before "$PIDFILE". 

# /etc/init.d/pptpd start
 * Starting PoPToP Point to Point Tunneling Server pptpd [ OK ]
# ps aux | grep pptpd
root 16211 0.0 0.0 10684 680 ? Ss 17:51 0:00 /usr/sbin/pptpd
root 16305 0.0 0.0 18236 904 pts/0 S+ 17:51 0:00 grep --color=auto pptpd
# /etc/init.d/pptpd status
 * /usr/sbin/pptpd is not running

This happens because status_of_proc takes $PIDFILE as $DAEMON, and tries to use the file "/var/run/pptpd.pid.pid" as source for the PID of the program. Adding "-p" solves the problem.
```

### 解决方法

1.通过 `vim` 打开 `/etc/init.d/pptpd` 文件：

```shell
sudo vim /etc/init.d/pptpd
```

2.然后查找这行代码 `status_of_proc "$PIDFILE"`,并添加 `-p` 参数，修改成这样：

```shell
status_of_proc -p "$PIDFILE" "$DAEMON""$NAME" && exit 0 || exit $?
```

3.保存文件并启动 pptpd

## 参考

- [PPTP 简介](https://baike.baidu.com/item/PPTP)
- [zhaoyu7777777 博客](http://blog.csdn.net/zhaoyu7777777/article/details/50724280)