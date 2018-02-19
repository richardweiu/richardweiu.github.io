---
title: PPA 的介绍
date: 2017-08-02 22:18:56
tags: " Linux "
categories: " 运维 "
---

> ** 前言 **
　PPA 的使用于添加可以说是基础到不能再基础的问题了，这是大学毕业没多久遇到的一个问题。

## PPA 简介

PPA 是 Personal Package Archives 的缩写，也就是个人软件包集。

有很多软件因为种种原因，不能进入官方的 Ubuntu 软件仓库。 为了方便 Ubuntu 用户使用，launchpad.net（由 Ubuntu 母公司 Canonical 有限公司所架设的Launchpad 网站） 提供了 ppa，允许用户建立自己的软件仓库， 自由的上传软件。PPA 也被用来对一些打算进入 Ubuntu 官方仓库的软件，或者某些软件的新版本进行测试。

PPA 上的软件极其丰富，如果 Ubuntu 官方仓库中缺少您需要的某款软件，可以去 PPA 上找找看。

一般为求稳定，官方源提供的软件安装包版本都比较老，或者根本就没有，这个时候我们就可以使用 PPA 找到最新的版本，或者相关软件。

## PPA 使用

要拿到最新的软件版本，就必须将相关的仓库加入到源。方法有两种：

- 手动修改 source 文件
- 用 `add-apt-repository` 命令

### 命令添加

一般系统是没有自带 add-apt-repository 命令，需要自己安装。

有三个安装包可以添加这个命令，三个安装针对不同的 ubuntu 版本与 python 的版本，具体 ubuntu 的版本以哪个版本作为分界线众说纷云有说 11.04 的，有说 12.04，我的方法便是多尝试，都安装试试：

```shell
#针对较早版本的
sudo apt-get install python-software-properties

#针对较新版本的
sudo apt-get install software-properties-common

#针对 python3 的
sudo apt-get install python3-software-properties
```

安装完成之后我们就拥有 `add-apt-repository` 这个命令了。这个命令的作用便是为我们自动添加源的地址文件，并自动为我们导入公钥。

例如我们为了获取到最新版的 python2.7 的版本，输入这样的命令：

```shell
sudo add-apt-repository -y ppa:fkrull/deadsnakes-python2.7
```

该命令会自动帮我们做两件事

1.在 `/etc/apt/sources.list.d/` 目录下创建 `fkrull-deadsnakes-python2_7-trusty.list` 文件，该文件的内容是：

```shell
deb http://ppa.launchpad.net/fkrull/deadsnakes-python2.7/ubuntu trusty main
# deb-src http://ppa.launchpad.net/fkrull/deadsnakes-python2.7/ubuntu trusty main
```

我们知道 `sources.list.d` 目录中文件会被包含读取，也就是说是 `sources.list` 文件的扩展，在 `apt-get update` 的时候同样会读取里面文件的源地址

2.导入公钥

只有导入公钥之后才能够安全的通信，下载源中的内容。

```shell
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys DB82666C
```

这便是 `add-apt-repository` 做的两件事情。

### 手动添加

由上述的过程我们可以很容易的得出我们该如何手动添加。

首先访问 https://launchpad.net 获取需要源的地址，然后在 `source.list` 文件中添加相关链接，或者在 `source.list.d` 中新建文件添加链接，建议新建文件，这样方便后期管理维护，例如上述例子：

```shell
sudo vim /etc/apt/source.list.d/python-version.list

deb http://ppa.launchpad.net/fkrull/deadsnakes/ubuntu trusty main 
deb-src http://ppa.launchpad.net/fkrull/deadsnakes/ubuntu trusty main 
```

然后添加其公钥：

```shell
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys DB82666C
```

接下来只需要 `sudo apt-get update` 即可，然后我们通过 `apt-cache policy python2.7` 命令可以看到出现了 python2.7 最新的几个版本。

## PPA 删除

通过上述过程我们了解到了如何添加 PPA，但是或许我们在后期并不会用到的时候我们需要将其删除，将其清除有这样的一些方法：

- 通过 `add-apt-repository`
- 手动删除
- 通过其他工具

### 通过 `add-apt-repository`

这种方法非常的简单，执行这样的命令即可，当然后面的 ppa 根据自己的情况修改：

```shell
sudo add-apt-repository --remove ppa:whatever/ppa
```

### 手动删除

1.删除 `/etc/source.list.d/` 目录下的文件，一般会有两个，一个是增加的源的 .list 文件，一个 `apt-get update` 之后的 .save 缓存文件

2.清理公钥

```shell
#列出所有公钥
sudo apt-key list

#删除指定公钥
sudo apt-key del B455BEF0

```

### 其他工具

使用 `ppa-purge` 工具来清理

```shell
#安装该工具
sudo apt-get install ppa-purge

#删除相关 ppa
sudo ppa-purge ppa_name

#更新 apt 的缓存
sudo apt-get purge package_name
```

## 参考

- [ubuntu 文档](http://people.ubuntu.com/~happyaron/udc-cn/lucid-html/ch11s02.html)
- [PPA help](https://help.launchpad.net/Packaging/PPA/InstallingSoftware)
- [PPA delete](http://askubuntu.com/questions/307/how-can-ppas-be-removed)