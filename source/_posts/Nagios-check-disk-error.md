---
title: Nagios check_disk error
date: 2017-10-19 15:15:15
tags: " Nagios "
categories: " Linux "
---

> ** 前言 **
　这几天因为各种扩展不同的节点忙的不可开胶，增加了新的节点也就意味着需要增加监控，因为各种特殊的原因导致以前的镜像不可用（因为内核版本的更替，架构的更替，还有一些定制化的需求），所以在新节点中增加 check_disk 监控时出现问题了。

## 问题

在增加监控的时候其他没有什么问题，但是在 check_disk 在运行的时候确实出现这样的报错：

```bash
var/lib/ureadahead/debugfs/tracing is not accessible: No such file or directory
```

一脸蒙啊，从来没有去看过 nagios 自带的这个 check_disk 的源码，自然也就不知道其原理是什么，也就是不知道爆出这样的错误是为什么。

`ureadahead` 是个什么东西也不知道。

第一反应是手动执行该脚本看是否有相同的问题，在自定义的用户同样的问题，修改 `/etc/passwd` 是 nagios 可以登陆，以 nagios 用户执行也是相同的问题，而当我切换至 root 账户来执行时却没有相关的问题，那可以定位这是一个权限的问题。

那么接下来的思路就应该是：

- 如何给 nagios 增加相关的权限
- ureadahead 是什么？有什么？

## 解决

google 了一下，查看到 [Ubuntu Sharing](http://ubuntuguide.net/howto-fix-ureadahead-problem-after-upgrading-to-ubuntu-10-04) 的文章解释的非常的全面，同时也给出了相应的解释。

我可以通过这样的一个命令可以将 ureadahead 关闭：

```
sudo mv /etc/init/ureadahead.conf /etc/init/ureadahead.conf.disable
```

关闭之后重启机器，切换至 nagios 用户，运行之前的命令，发现一切正常了。

那么 ureadahead 到底是一个什么东西呢？是否可以关闭？

文章中这一段解释的非常的清楚：

```
What does ureadahead do?

In order to boot Ubuntu, we need to read somewhere between 100MB and 200MB of data from the disk and into memory. Unfortunately the slowest part of your otherwise awesome machine is its hard disk — that’s why we want this data into memory in the first place.

Hard disks aren’t just slow to read data, they’re slow to find it as well! So we can lose a time of time during boot just waiting for the hard disk to find the data we need, and then read it into memory.

What ureadahead does is figure out what pieces of which files we actually need, and read everything from the disk into memory in one go. By doing it at once we don’t need to spend so much time finding everything, and because it’s already in memory, we don’t waste anywhere near as much time during boot.
```

也就是说它可以帮助我们快速的找到那些文件需要在启动的时候读入到内存中，从而大大的减少开机所需要的时间，若是关闭掉它的话造成的影响就是我们的开机就会变慢。

同时我也发现了这样的一个 [bug 描述](https://bugs.launchpad.net/ubuntu/+source/mountall/+bug/736512)，也就是说 moutall 的一个 bug 导致在启动之后还存在与 `df` 查看信息中。但是这个 bug 早就修复了呀。

我也在 launchpad 中看到与我遇到的同样的[问题描述](https://bugs.launchpad.net/ubuntu/+source/nagios-plugins/+bug/1516451),给出的解决方式是在 check_disk 命令后面增加这样的一些参数：

```
--exclude-type=tracefs --exclude-type=cgroup --exclude_device=/run/lxcfs/controllers
```

这个方式我也不知道有没有用，因为看到这个问题描述的时候我已经解决了问题，问题没法复现了，除非重新启动一个机器。

我解决问题的方式除了上面所说的修改配置文件，还有一个阴差阳错的办法，因为这个方式是用在需要挂载其他硬盘的方式，我在挂载之后 check_disk 没法运行，然后我就将 `/etc/fstab` 相关的配置项删除之后，重启发现就没有问题了，然后我把相关的磁盘挂载配置项改回去重启，依然没有问题，从而解决了，而用之前的方式是因为我没有外界硬盘，这个系统只有根目录那个磁盘挂载，所以只有另辟蹊径。

因为之前出问题的时候也没有在意去看 `df -all` 或者 `/etc/mtab` 中是否有 `/var/lib/ureadahead/debugfs` 的信息存在，所以也不知道是不是 mount 的问题。

## 参考

- [Ubuntu Sharing](http://ubuntuguide.net/howto-fix-ureadahead-problem-after-upgrading-to-ubuntu-10-04)