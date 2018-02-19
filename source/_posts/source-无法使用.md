---
title: source 无法使用
date: 2018-01-01 15:34:31
tags: " Linux "
categories: " 运维 "
---

> ** 前言 **
　有时候在脚本中需要使用 source 命令，但是脚本却执行失败，无法使用 source 命令，`command not found`

## 问题

在 Jenkins 中的 pipeline 使用 `sh` 可以让其在过程中执行一些 shell 命令。

在 ansible 中使用 shell module 的时候也可以执行 shell 命令。

但是这样的调用只会使用 `sh`，就会出现使用 source 命令的时候报错 `command not found`，这是因为默认的 `sh` 使用的是 dash 来解析命令，而 `source` 命令是 `bash` 的内置命令。

我们可以通过这样的命令来验证：

```bash
 ls -l `which sh`

# 一般的结果应该是这样的
lrwxrwxrwx 1 root root 4 Feb 19  2014 /bin/sh -> dash
```

## 解决

所以解决问题的方式就是将默认的解释器从 dash 切换到 bash 中即可：

```bash
sudo dpkg-reconfigure dash

# 会出现一个图形界面，我们直接选择 no 即可
```

此时我们再通过 `ls -l $(which sh)` 来查看我们是否成功修改成 bash 来做默认的解析器。

修改成功之后 `source` 与 `.` 即可直接使用了。