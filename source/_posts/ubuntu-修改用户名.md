---
title: ubuntu 修改用户名
date: 2017-10-18 13:43:34
tags: " Linux "
categories: " 运维 "
---

> ** 前言 **
　这个命令这的是用的频次太少了，以至于我都忘了这个命令的存在 `usermode`。感觉现在博客都成了我的回忆录了，为了不把这些旧的知识累计，求快都没有总结，尽量简短。

## 修改用户名

由于增加了新的站点，以至于镜像中的用户名不能再沿用。所以有了修改用户名的需求（其实我的第一反应比较蠢，是删除原来的用户增加新的用户，但是发现修改的地方太多才想到修改用户名的）

修改镜像中的用户名，难道我还重新 build 镜像，无异于增加了维护镜像的工作量，还好我前端时间非常机制的提出了建议将 init 脚本挂载起来，这样就可以非常灵活的控制镜像了，什么修改源啊、修改中英文啊都不用在像以前一样重新 build 镜像了。

修改用户命令，有些人想到的方法是修改 `/etc/passwd` 与 `/etc/group` 文件，这种方法简直就是坑，那家目录怎么办，会引发一些不必要的问题，正统的解决方法应该是使用 `usermod` 命令与 `groupmod` 命令来修改相关用户的名字与组的命令。

`usermode` 的介绍：

```bash
The usermod command modifies the system account files to reflect the changes that are specified on the command line.
```

`groupmod` 的介绍：

```bash
The groupmod command modifies the definition of the specified GROUP by modifying the appropriate entry in the group database.
```

所以修改用户名只需要两个步骤：

- 修改用户的名字：

```bash
usermod -l new_username -d /home/new_username -m old_username
```

- 修改用户的组名字：

```bash
groupmod -n new_username old_username
```

这样原来的用户就跟不存在一样。

## 添加到其他组

有这个两个命令让我想起在安装 docker 的时候为了不用每次都在命令前面添加 `sudo` 才能执行 docker 相关的命令，我们会将用户添加到 docker 的用户组中去，这样我们才有了 docker 组相关的权限：

```bash
sudo gpasswd -a nagios docker
```
