---
title: ubuntu 开机修复
date: 2019-10-10 09:52:34
tags: [ " Ubuntu ", "Linux" ]
categories: " 运维 "
---

> ** 前言 **
　手上的笔记本装的是 ubuntu，一次开机就再也进不去的修复记录

## 问题解决

下午的时候重启了一下电脑，结果就一直卡在开机画面，然后再也进入不了桌面了，字符模式中一直开在这个位置

```bash
[ OK ] Started irqbalance daemon
[ OK ] System Logging Service.
[ OK ] Started GNOME DisplayManager. Dispatcher Service....upport.hanges.pp link was shut down.
```

一按电源键关机就卡在这个位置：

```bash
[   .] A start job is running for Hold until boot process finishes up (28s / no limit)[ OK ] Started Show Plymouth BootScreen.
```

与这个的情况非常的类似：

- https://askubuntu.com/questions/1051555/after-security-update-to-4-15-0-24-generic-26-ubuntu-screen-shows-log-content

具体原因不是很清楚处理过的操作有：

- 重启开启在进入系统前，按 esc
- 选择 recovery mode 模式中
- 然后一堆升级的操作

```bash
apt-get update
apt-get upgrade
apt-get dist-upgrade
```

- 升级之后发现并没有什么乱用，然后看到有人说是内核问题，装工具

```bash
apt install haveged
systemctl enable haveged
```

- 依旧没有什么用，中途有几次开机之后卡在 mysql 的启动和 redis 的启动，网络，snapd启动，

```bash
Starting mysql tails any system changes cklight
Started waiting until snapd is fully seeded
Starting LSB: automatic crash report generation...
```

- 最后然后重装桌面工具就好了

```bash
apt purge gdm gdm3 libgdm1
apt install gdm3 ubuntu-desktop
```

- https://askubuntu.com/questions/641642/gui-does-not-start


在删除 redis，mysql 等的时候发现这样的报错 `/usr/bin/mandb permission denied` 当时没有处理，该链接表示可以通过这样的命令处理 `mandb -csp`

- https://www.vmvps.com/how-to-solve-the-error-caused-by-usr-bin-mandb.html


## 参考
