---
title: PHP 依赖管理
date: 2017-08-02 21:56:07
tags: " PHP "
categories: " 运维 "
---

> ** 前言 **
　有个同事在写关于 PHP 相关的课程，用到了 Composer 这个工具，需要将其更新至镜像中，便记录了一下这个工具的一些介绍与安装。此文来自于 [Composer 中文网](www.phpcomposer.com/)，若是方法失效，查看官网。

## Composer 的介绍

Composer 是 PHP 用来管理依赖（dependency）关系的工具。 你可以在自己的项目中声明所依赖的外部工具库（libraries），Composer 会帮你安装这些依赖的库文件。

Composer 不是一个包管理器。虽然它涉及 "packages" 和 "libraries"，但它在每个项目的基础上进行管理，在你项目的某个目录中（例如 vendor）进行安装。默认情况下它不会在全局安装任何东西。因此，这仅仅是一个依赖管理。

这种想法并不新鲜，Composer 受到了 node's npm 和 ruby's bundler 的强烈启发。而当时 PHP 下并没有类似的工具。

Composer 将这样为你解决问题：

a) 你有一个项目依赖于若干个库。

b) 其中一些库依赖于其他库。

c) 你声明你所依赖的东西。

d) Composer 会找出哪个版本的包需要安装，并安装它们（将它们下载到你的项目中）。

## Composer 的安装

Composer 有两种安装方式：

- 局部安装
- 全局安装

### 局部安装

要真正获取 Composer，我们需要做两件事。首先安装 Composer （同样的，这意味着它将下载到你的项目中）：

```shell
curl -sS https://getcomposer.org/installer | php
```

> 注意： 如果上述方法由于某些原因失败了，你还可以通过 php >下载安装器：

```php
php -r "readfile('https://getcomposer.org/installer');" | php
```

这将检查一些 PHP 的设置，然后下载 composer.phar(Composer 的二进制文件) 到你的工作目录中。这是一个 PHAR 包（PHP 的归档），PHP 的归档格式可以帮助用户在命令行中执行一些操作。

你可以通过 --install-dir 选项指定 Composer 的安装目录（它可以是一个绝对或相对路径）：

curl -sS https://getcomposer.org/installer | php -- --install-dir=bin

### 全局安装

你可以将此文件放在任何地方。如果你把它放在系统的 PATH 目录中，你就能在全局访问它。 在类Unix系统中，你甚至可以在使用时不加 php 前缀。

你可以执行这些命令让 composer 在你的系统中进行全局调用：

```shell
curl -sS https://getcomposer.org/installer | php

mv composer.phar /usr/local/bin/composer
```

> 注意： 如果上诉命令因为权限执行失败， 请使用 `sudo` 再次尝试运行 mv 那行命令。

现在只需要运行 composer 命令就可以使用 Composer 而不需要输入 php composer.phar。


### 全局安装 (on OSX via homebrew)

Composer 是 homebrew-php 项目的一部分。

```shell
brew update
brew tap josegonzalez/homebrew-php
brew tap homebrew/versions
brew install php55-intl
brew install josegonzalez/php/composer
```

## 参考

- [Composer 中文网](www.phpcomposer.com/)