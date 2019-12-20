---
title: systemd 启动服务增加环境变量
date: 2019-12-18 22:40:26
tags: [ "systemd", " Linux " ]
categories: "运维"
---

> ** 前言 **
   帮同事解决启动程序没有读取到环境变量的问题 

## 正文

同事希望通过 systemd 来控制自己的程序开机自启动与程序的启动、停止、重启，而程序启动读取的配置文件依赖于启动时获取到的环境变量，所以有了这个需求。

当前的问题情况是：

- 写好了 systemd 的配置文件可以启动但是获取不到环境变量
- 环境变量写在 `/etc/environment` 中

### 解决

- 将变量写至 `/etc/sysconfig/circus` （文件名请与服务名相同）
- 环境变量文件以 `key=value` 的形式存在切勿写 `export` 命令
- 在 `/etc/systemd/system/circus.service` 中的 `Service` 中写入 `EnvironmentFile=/etc/sysconfig/circus` 的指定

最后在 `systemctl daemon-reload && systemctl restart circus` 即可

出现上述的错误是原因与注意点：

- `/etc/environment` 并不是全局环境变量配置文件，他只是用户级的（参看参考中的说明关系）
- systemd是所有进程的父进程或者祖先进程，它的pid是1，它的配置文件/etc/systemd/system/default.target
- 指定的配置文件中的格式只需要指定 `key=value` 就好

## 参考

- 解决问题：https://serverfault.com/questions/828999/systemd-environment-and-environmentfile-not-working?rq=1
- 文档说明：https://www.freedesktop.org/software/systemd/man/systemd.exec.html#EnvironmentFile=
- 说明关系：https://blog.csdn.net/m0_38063172/article/details/83784443
