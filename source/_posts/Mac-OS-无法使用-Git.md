---
title: Mac OS 升级导致的问题
date: 2017-11-20 09:10:39
tags: " Mac "
categories: " Mac "
---

> ** 前言 **
　前阵子升级了一下 Mac 到 Mac OS High Sierra(10.13.1)，因为最近都在搞环境没有使用本地的 git，也就没有发现问题，今天一用突然发现用不了了。

## Git 无法使用

### 问题

问题的产生是将 Mac 系统升级了一下，当然 xcode 也升级了，然后使用 git 的时候出现了这样的报错：

```bash
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
```

### 解决

通过这个命令重新安装一下 xcode 开发工具就可以解决：

```bash
xcode-select --install
```

## Telnet 无法使用

### 问题

同样是因为升级系统造成的问题，telnet、ftp 等一些基础的网络命令不见了

### 解决

通过 homebrew 重新安装一下，可以只安装 telnet 也可以装网络工具集：

```bash
brew install telnet

or

brew install inetutils
```

## 参考

- [Oh The Huge Manatee](https://ohthehugemanatee.org/blog/2015/10/01/how-i-got-el-capitain-working-with-my-developer-tools/)
- [StackExchange](https://apple.stackexchange.com/questions/299758/how-to-get-bsd-ftp-and-telnet-back-in-10-13-high-sierra)
