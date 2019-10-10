---
title: centos rpmdb 损坏修复
date: 2019-10-10 10:05:54
tags: [ "Centos", " Linux " ]
categories: " 运维 "
---

> ** 前言 **
　在处理老机器的时候遇到 rpmdb 损坏导致不可用的情况

# 问题修复

进行任何rpm操作时提示：

```bash
rpm -qa|grep openssl|grep openssl-1.0.1e|wc -l

error: db3 error(12) from dbenv->open: Cannot allocate memory
error: db3 error(12) from dbenv->close: Cannot allocate memory
error: cannot open Packages index using db3 - Cannot allocate memory (12)
error: cannot open Packages database in /var/lib/rpm
error: db3 error(12) from dbenv->open: Cannot allocate memory
error: db3 error(12) from dbenv->close: Cannot allocate memory
error: cannot open Packages database in /var/lib/rpm
```

这是由于rpm数据库文件损坏所致

解决办法：

```shell
rm -rf /var/lib/rpm/__db*
rpm --rebuilddb
```

将rpm数据库文件删除再重建，以root来运行上面的两条命令，问题解决。`

## 参考

- http://www.51niux.com/?id=181
