---
title: Mac OS 无法使用 Git
date: 2017-11-20 09:10:39
tags: " Mac "
categories: " Mac "
---

> ** 前言 **
　前阵子升级了一下 Mac，因为最近都在搞环境没有使用本地的 git，也就没有发现问题，今天一用突然发现用不了了。

## 问题

问题的产生是将 Mac 系统升级了一下，当然 xcode 也升级了，然后使用 git 的时候出现了这样的报错：

```bash
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
```

## 解决

通过这个命令重新安装一下 xcode 开发工具就可以解决：

```bash
xcode-select --install
```

## 参考

- [Oh The Huge Manatee](https://ohthehugemanatee.org/blog/2015/10/01/how-i-got-el-capitain-working-with-my-developer-tools/)