---
title: zsh 注意
date: 2018-01-01 16:26:59
tags: [" Linux "," zsh "]
categories: " 运维 "
---

> ** 前言 **
　在 zsh 中有些很强大的功能，高级的自动补全，输入部分命令就可以通过上下箭头搜索相关的命令等等，但是还是有一些习惯与 bash 中的不同

## zsh 的 path

在 Bash 中我们知道通过 `echo $PATH` 可以查看我们的环境变量，也就是命令搜索的位置。

使用这个命令的时候必须使用全大写的 `PATH`,若是小写的 `path` 并没有任何的作用。

而在 zsh 中大写 `PATH` 与小写 `path` 两个变量的作用都是存在的，大写的作用与在 Bash 中的相同申明路径，而小写的作用是将 `PATH` 中的路径转换成一个路径列表

在官方的邮件列表中发现有人提问，官方人员是这样回答的（忘记保存当时查到的连接了）：

```
That is not a bug. It's a feature. See the discussion of "typeset -T" in
"man zshbuiltins". The lower-case version of PATH is an array parameter
bound to the scalar upper-case parameter. The CDPATH parameter has a
similar binding. You can create your own with "typeset -T".
```

## zsh 的快捷键

在 Bash 中我们可以通过 `alt + 方向键` 来根据单词移动我们的光标，但是在 zsh 中却做不到，这样输入只会得来这样的输出:

```bash
[D[C
```

我们需要在 `.zshrc` 中增加这样的配置：

```bash
bindkey "\e\e[D" backward-word  
bindkey "\e\e[C" forward-word
```

这样功能便恢复了。

## zsh 的 export

在 zsh 中不能使用 `export -f` 来导出函数，使得子 shell 中使用导出的函数

## zsh 的 history

在 Bash 中我们可以通过 `history -c` 来清理我们的历史命令，但是在 zsh 中的 `history` 并没有这个参数，若是想要清除 history 需要删除 `.zsh_history` 中的内容。