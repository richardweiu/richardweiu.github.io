---
title: Linux_help
date: 2016-07-07 14:32:10
tags: " Linux "
categories: " 运维 "
---

>**前言**
　今天写了一门算是课程的文章，既然回顾了一下 Linux 中的帮助命令，也就一起转移上来吧。

### 内建命令与外部命令

什么是内建命令，什么是外部命令呢？这和帮助命令又有什么关系呢？

因为有一些查看帮助的工具在内建命令与外建命令上是有区别对待的。

> **内建命令**实际上是 shell 程序的一部分，其中包含的是一些比较简单的 Linux 系统命令，这些命令是写在bash源码的builtins里面的，并由 shell 程序识别并在 shell 程序内部完成运行，通常在 Linux 系统加载运行时 shell 就被加载并驻留在系统内存中。而且解析内部命令 shell 不需要创建子进程，因此其执行速度比外部命令快。比如：history、cd、exit 等等。

> **外部命令**是 Linux 系统中的实用程序部分，因为实用程序的功能通常都比较强大，所以其包含的程序量也会很大，在系统加载时并不随系统一起被加载到内存中，而是在需要时才将其调用内存。虽然其不包含在 shell 中，但是其命令执行过程是由 shell 程序控制的。外部命令是在 Bash 之外额外安装的，通常放在/bin，/usr/bin，/sbin，/usr/sbin等等。比如：ls、vi等。

简单来说就是一个是天生自带的天赋技能，一个是后天得来附加技能。我们可以使用　type 命令来区分命令是内建的还是外部的。例如这两个得出的结果是不同的，不信你试试

```
type exit

type service
```

没有骗你吧，得到的是两种结果吧，若是对ls你还能得到第三种结果
![实验楼](https://dn-simplecloud.qbox.me/1135081467870301890-wm)

```
#得到这样的结果说明是内建命令，正如上文所说内建命令都是在 bash 源码中的 builtins 的.def中
xxx is a shell builtin
#得到这样的结果说明是外部命令，正如上文所说，外部命令在/usr/bin or /usr/sbin等等中
xxx is /usr/sbin/xxx
#若是得到alias的结果，说明该指令为命令别名所设定的名称；
xxx is an alias for xx --xxx
```

### 帮助命令的使用

#### help 命令

若是本地机有Linux环境可以在试试下面这个命令(因为本实验环境中用的是zsh，而不是bash。所以无法执行该命令)

```
#请看截图本实验环境中用的是zsh，而不是bash。所以无法执行该命令
help ls
```

得到的结果如图所示，是不是很奇怪。为什么是这样的结果呢？

![Help_for_nei_wai]()

因为 help 命令是用于显示 shell 内建命令的简要帮助信息。帮助信息中显示有该命令的简要说明以及一些参数的使用以及说明，一定记住 help 命令只能用于显示内建命令的帮助信息，不然就会得到你刚刚得到的结果。

那如果是外部命令怎么办，不能就这么抛弃它呀。其实外部命令的话基本上都有一个参数--help,这样就可以得到相应的的帮助，看到你想要的东西了。试试下面这个命令是不是能看到你想要的东西了。

```
ls --help
```

![实验楼](https://dn-simplecloud.qbox.me/1135081467871419660-wm)

#### man 命令

你可以尝试下这个命令

```
man ls
```

![实验楼](https://dn-simplecloud.qbox.me/1135081467871829217-wm)

得到的内容是不是比用 help 更多更详细，而且man没有种族歧视(没有内建与外部命令的区分)，因为 man 工具是显示系统手册页中的内容，也就是一本电子版的字典，这些内容大多数都是对命令的解释信息，还有一些相关的描述。通过查看系统文档中的 man 页可以得到程序的更多相关信息和 Linux 的更多特性。

是不是好用许多，当然也不代表 help 就没有存在的必要，当你非常紧急只是忘记改用那个参数什么的时候，help 这种显示简单扼要的信息就特别使用，若是不太紧急的时候就可以用 man 这种详细描述的查询方式

在尝试上面这个命令是我们会发现最左上角显示“LS（1）”，在这里，“LS”表示手册名称，而“（1）”表示该手册位于第一章节。这个章节又是什么呢？在 man 手册中一共有这么几个章节:

|参数 | 说明 |
|-----|-----|
| 1 | Standard commands （标准命令）|
| 2 | System calls （系统调用）|
| 3 | Library functions （库函数）|
| 4 | Special devices （设备说明）|
| 5 | File formats （文件格式）|
| 6 | Games and toys （游戏和娱乐）|
| 7 | Miscellaneous （杂项）|
| 8 | Administrative Commands （管理员命令）|
| 9 | 其他（Linux特定的）， 用来存放内核例行程序的文档。|

打开手册之后我们可以通过 pgup 与 pgdn 或者上下键来上下翻看，可以按 q 退出当前页面

#### info 命令

要是你觉得man显示的信息都还不够，满足不了你的需求，那只好放大招了，试试这个命令

```
#该命令与help一样，zsh没有，bash有。在本地尝试
info ls
```

![Info_for_ls]()

得到的信息是不是比 man 还要多了，info 来自自由软件基金会的 GNU 项目，是 GNU 的超文本帮助系统，能够更完整的显示出 GNU 信息。所以得到的信息当然更多拉

Man 和 info 就像两个集合，它们有一个交集部分，但与 man 相比，info 工具可显示更完整的GNU工具信息。若 man 页包含的某个工具的概要信息在 info 中也有介绍，那么 man 页中会有“请参考 info 页更详细内容”的字样。