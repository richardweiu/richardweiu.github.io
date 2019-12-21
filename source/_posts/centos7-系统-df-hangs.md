---
title: centos7 系统 df hangs
date: 2019-12-18 23:23:18
tags: ["Linux", "Centos7"]
categories: "运维"
---

> ** 前言 **
　在线上发现一次各个进程的 CPU 使用都不高，但是 load average 却达到了 1000+

## 问题

在线上排查问题的时候突然发现有一台机器的 load average 居然到了 1000+，但是内存用的不多，CPU 占用的不多，然后使用 iostat 查看磁盘的读写也是在正常的范围里，很奇怪

但是就在此时希望通过 `df` 查看磁盘是否写满的时候发现该命令 hangs 了，卡主不动，并且通过 `ps` 查看监控的 agent 定时运行的 `df -l` 也全部 hangs，借鉴参考中的博客，尝试用 `strace` 追踪一下 hangs 的原因(因为处理线上机器的时候没有记录现场所以借鉴了参考中的内容)：

```bash
>strace df
execve("/usr/bin/df", ["df"], [/* 29 vars */]) = 0
brk(0)                                  = 0x1731000
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fa7720a7000
access("/etc/ld.so.preload", R_OK)      = 0
open("/etc/ld.so.preload", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=24, ...}) = 0
......
stat("/sys/fs/cgroup/memory", {st_mode=S_IFDIR|0755, st_size=0, ...}) = 0
stat("/sys/kernel/config", {st_mode=S_IFDIR|0755, st_size=0, ...}) = 0
stat("/", {st_mode=S_IFDIR|0555, st_size=4096, ...}) = 0
stat("/proc/sys/fs/binfmt_misc", 
```

具体导致问题的详细原因与排查的过程查看参考，不过怀疑的原因是 systemd 和 kernel 之间存在竞争而引起的, 导致其它程序访问挂载点的时候出现 hang 住, 解决方式是：

```bash
systemctl restart proc-sys-fs-binfmt_misc.automount
```

## 参考

- https://blog.arstercz.com/centos7-%E7%B3%BB%E7%BB%9F-df-hang-%E9%97%AE%E9%A2%98%E5%A4%84%E7%90%86%E8%AF%B4%E6%98%8E/
