---
title: sudo 无法使用
date: 2017-08-03 08:58:26
tags: " Ubuntu "
categories: " 运维 "
---

> ** 前言 **
　原来刚上班的时候每天都会写 everyday note，这是在 2016 年 8 月 23 日遇到的一个事情，用的监控是 nagios，nagios 在每台被监控的会装 nrpe 插件用于接受 server 端的请求与发送被监控机器的信息，为了方便 nrpe 在执行一些特殊脚本的时候 sudo 不用输入密码，便去修改了 sudoers 文件，哪知改出问题来。

## sudo 简介

首先简单介绍一下 `sudo`:

在 `sudo` 于 1980 年前后被写出之前，一般用户管理系统的方式是利用 `su` 切换为超级用户。但是使用 `su` 的缺点之一在于必须要先告知超级用户的密码。

sudo 使一般用户不需要知道超级用户的密码即可获得权限。首先超级用户将普通用户的名字、可以执行的特定命令、按照哪种用户或用户组的身份执行等信息，登记在特殊的文件中（通常是/etc/sudoers），即完成对该用户的授权（此时该用户称为“sudoer”）；在一般用户需要获取特殊权限时，其可在命令前加上“sudo”，此时 sudo 将会询问该用户自己的密码（以确认终端机前的是该用户本人），回答后系统即会将该命令的进程以超级用户的权限运行。之后的一段时间内（默认为5分钟，可在/etc/sudoers自定义），使用 sudo 不需要再次输入密码。
由于不需要超级用户的密码，部分 Unix 系统甚至利用 sudo 使一般用户取代超级用户作为管理账号，例如 Ubuntu、Mac OS X 等。
(此段来自于[维基百科](https://zh.wikipedia.org/wiki/Sudo))

## sudo 权限

拥有了 sudo 权限便可以使用 sudo 来执行 root 才可以执行的相关命令，如何让用户拥有 sudo 的权限？

通过 useradd 添加的用户,并不具备 sudo 权限。在 ubuntu/centos 等系统下,需要将用户加入 admin 组, 才能使其能够输入自己的账号密码来使用 sudo。

```shell
usermod -a -G admin [用户]
```

当然这里说明一下，在 ubuntu 的 11.10 之前的版本，使用 sudo 工具的管理员访问权限需要加入 `admin group`，而在 12.04 的版本中加入了 sudo group，此时便是通过加入 sudo 组来授予相关权限，但是为了兼容并没有废弃 admin 组，所以存在两个权限类似的组。

```
Up until Ubuntu 11.10, administrator access using the sudo tool was granted via the "admin" Unix group. In Ubuntu 12.04, administrator access will be granted via the "sudo" group. This makes Ubuntu more consistent with the upstream implementation and Debian. For compatibility purposes, the "admin" group will continue to provide sudo/administrator access in 12.04.
```
(此段来自于 [Ubuntu wiki](https://wiki.ubuntu.com/PrecisePangolin/ReleaseNotes/UbuntuDesktop#PrecisePangolin.2BAC8-ReleaseNotes.2BAC8-CommonInfrastructure.Common_Infrastructure))

Debian 只拥有 sudo 组，默认并没有创建 wheel 组，wheel 组的概念继承自 UNIX。

Fedora 与 centos 这样的红帽子系列一般都是用的 wheel 组，来添加 sudo 权限（最开始是添加 su 权限的）。

这里扩展一下，在 Mac 中我们还可以看到 staff 组，staff 组所有普通用户所在的组

还有一个就是 admin 与 wheel 组，一般情况下管理员所在的组就是 admin 组，wheel 组诗让其他用户有 sudo、su 的组（好像是这么理解的）

## sudo 配置

### 编辑器

一般情况下加入 `sudo` 组的用户都会做登记。普通用户的名字、可以执行的特定命令、按照哪种用户或用户组的身份执行等信息，登记在特殊的文件中。

总的配置文件是 `/etc/sudoers`，一般新添加的其他用户会在 `/etc/sudoers.d/` 目录中。

sudoers 这个总的配置文件只能使用 `visudo` 这个工具来修改，这个工具会锁住 sudoers 文件，保存修改到临时文件，然后检查文件格式，确保正确后才会覆盖 sudoers 文件。必须保证 sudoers 格式正确，否则 sudo 将无法运行。

你若是用 `sudo vim /etc/sudoers` 的话会是这样的警告：

```
W10: Warning: Changing a readonly file
```

默认情况下 visudo 工具用的是 nano 编辑器（我是觉得这个工具难用的要死），官方仓库里的 sudo 编译时开启了 `--with-env-editor`，会采用环境变量 `VISUAL` 和 `EDITOR` 的设置。如果设置了 `VISUAL` 就不会使用 `EDITOR`。

如果要临时使用其他编辑器，在该命令前加上 `EDITOR` 环境变量即可。例如：

```
# EDITOR=vim visudo
```

若是像永久修改可以在 sudoers 中添加这样一句话:

```
Defaults      editor=/usr/bin/vim, !env_editor
```

在 ubuntu 中还可以设置默认的 editor：

```shell
update-alternatives --config editor
```

然后选择 vim 即可。

### 语法

sudoers 文件的格式一般是这样的：

1. 以 `#` 开头的行为注释行

2. 授权的格式是这样的：

```
授权用户/组 主机=[(切换到哪些用户或组)] [是否需要输入密码验证] 命令1,命令2,...
字段1      字段2 =[(字段3)] [字段4] 字段5
```

例如：

```shell
root   ALL=(ALL:ALL) ALL
%admin ALL=(ALL) ALL
%sudo  ALL=(ALL:ALL) ALL
```

- 凡是[ ]中的内容, 都能省略; 命令和命令之间用,号分隔;
- 字段一：以%号开头的表示"将要授权的组", 比如例子中的%admin 和 %sudo
- 字段二：表示允许登录的主机, ALL 表示所有; 如果该字段不为 ALL,表示授权用户只能在某些机器上登录本服务器来执行 sudo 命令
- 字段三：如果省略,表示切换到root用户; 如果为ALL, 表示能够切换到任何用户; 例子中的(ALL:ALL) 是允许任何 (用户:组) 的意思.
- 字段四：可取值是NOPASSWD:。表示执行sudo时可以不需要输入密码
- 字段五：使用逗号分开一系列命令,这些命令就是授权给用户的操作; ALL表示允许所有操作
(此段来自于参考二、四)

查看用户有哪些权限可以使用 `sudo -l` 查看，或者 `sudo -ll`,命令 `sudo -lU user` 可以查看某个特定用户的设置:

```shell
 $ sudo -l

Matching Defaults entries for simplecloud on docker1:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User simplecloud may run the following commands on docker1:
    (ALL : ALL) ALL

 $ sudo -ll
Matching Defaults entries for user on host:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User simplecloud may run the following commands on docker1:

Sudoers entry:
    RunAsUsers: ALL
    RunAsGroups: ALL
    Commands:
	ALL
```

sudoers 编辑的案例，例如之前所提到的 nrpe：

```shell
nagios ALL=(root) NOPASSWD: /etc/init.d/nagios-nrpe-server,/sbin/iptables-save,/sbin/restart
Defaults:nagios !requiretty
```

亦或者是 shiyanlou 上账户的设置：

```
shiyanlou ALL=(ALL) NOPASSWD: ALL
Defaults:shiyanlou !requiretty
```

### 技巧

在上面的案例中我们看到有个 Defaults 的配置项，其作用就是来做一些特定的限制，比如一下案例：

#### 设置密码过期时间

```
Defaults:用户名 timestamp_timeout=20
```

对该用户，sudo 将记录密码 20 分钟，时间值也可以是小数。若是值为 0 则每次都需要询问

#### 跨终端 sudo 

如果不想每次启动新终端都重新输入密码，在配置文件中禁止 tty_tickets 即可：

```
Defaults !tty_tickets
```

#### 环境变量

当前用户的环境变量不会应用到 sudo 启动的程序，除非使用 `-E` 选项：

```shell
$ sudo -E pacman -Syu
```

如果经常需要这样做，可以在 `~/.bashrc`（或其他shell配置文件）中加入命令别名：

```shell
alias sudo="sudo -E"
```

在 `/etc/sudoers` 中添加以下内容作用相同：

```
Defaults !env_reset
```

可以把需要传递环境变量的命令设置到env_keep：

```
Defaults env_keep += "ftp_proxy http_proxy https_proxy no_proxy"
```

#### 传递命令别名
当前用户的命令别名不会应用到 sudo。如果需要这样，只需在 `~/.bashrc` 或者 `/etc/bash.bashrc` 中加入：

```
alias sudo='sudo '
```

默认sudo询问用户密码，若想让用户 sudo 使用 root 密码。添加 `targetpw` 或 `rootpw` 到配置文件的 `Defaults` 部分，可以让 sudo 询问 root 密码：

```
Defaults targetpw
```

可以限定特定的组使用 root 密码：

```
Defaults:%wheel targetpw
%wheel ALL=(ALL) ALL
```

#### SSH TTY 问题

远程执行命令时，SSH 默认不会分配 tty。没有 tty，sudo 就无法在获取密码时关闭回显。使用 `-tt` 选项强制 SSH 分配 tty（使用两次-tt）。
另一方面，sudoers 中的 Defaults 选项 requiretty 要求只有拥有 tty 的用户才能使用 sudo。 禁用这个选项：

```
Defaults    !requiretty
```
(此段来自参考二)

#### 日志记录

通过在 `/etc/syslog.conf` 文件中使用以下设置，可以使用 syslog 在 `/var/adm/messages` 中记录通过 sudo 运行的所有命令：

```
*.debug   /var/adm/messages
```

但是，我认为 sudo 命令应该记录在一个单独的文件中，这样便于查看已经运行的 sudo 命令。当然，这也有助于监视失败的 sudo 活动。创建 /var/adm/sudo.log 文件，然后在 /etc/sudoers 文件中添加以下条目：

```
Defaults logfile=/var/adm/sudo.log
Defaults !syslog
```

现在，无论执行成功还是失败，所有 sudo 活动都会记录在 /var/adm/sudo.log 中。

(此段来自于参考三，还有变量的使用，以及一些使用的经验)

## 问题与解决

年少的我，没有测试机，只好在生产服务器中修改 sudoers 文件，但是在为 nagios 增加免密码命令的时候，大意少写了一个逗号还是路径的 `/`，导致 sudoers 出错，使得 sudo 命令没有办法使用，使用就报错:

```
>>> /etc/sudoers.d/nrpe: syntax error near line 1 <<<
sudo: parse error in /etc/sudoers.d/nrpe near line 1
sudo: no valid sudoers sources found, quitting
sudo: unable to initialize policy plugin
```

但是要修改 sudoers 文件又必须要 sudo 命令，好像陷入了死循环，baidu 出来的结果全部必须重启自己，进入recovery mode，到有root界面，然后进入 root，更改 /etc/sudoers 权限，然后修改。但是我在阿里云上没有办法啊，而且不敢也不能随便重启生产机器，后来找到了这样一个命令 `pkexec`。

这是 `pkexec` 的介绍：

```
pkexec allows an authorized user to execute PROGRAM as another user. If username is not specified, then the
program will be executed as the administrative super user, root.
```

通过这个命令我得以再次修改 sudoers 文件，将 sudo 命令恢复。

当时真是吓到一身汗。

## 参考

- [sudo 与 admin 区别](https://askubuntu.com/questions/43317/what-is-the-difference-between-the-sudo-and-admin-group)
- [编辑器修改](https://wiki.archlinux.org/index.php/Sudo_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
- [IBM sudo 博客](https://www.ibm.com/developerworks/cn/aix/library/au-sudo/index.html)
- [旺酱在路上 sudoers 介绍](https://segmentfault.com/a/1190000007394449)