---
title: git commit 修改
date: 2017-10-16 10:21:52
tags: " Git "
categories: " Git "
---

> ** 前言 **
　不用就会忘，记录一下已经提交的 commit 信息写错了的修改。

## 修改 commit 信息

修改最近一次的 commit 信息只需要用这个命令：

```
git commit --amend
```

执行命令之后会打开一个文本编辑工具（默认是 vi），然后直接修改保存即可

紧接着只需要就本次修改提交到线上即可：

```
git push -f origin master
```

这里增加 `-f` 的原因是最近的一次 commit 的信息与 server 上的不同，不 force 的话提交不上去。

因为在本地我配置了 vim 的自定义环境，而此时默认调用的是 vi，所以我出现了这样的报错：

```
error: There was a problem with the editor 'vi'.
Please supply the message using either -m or -F option.
```

我通过修改默认的编辑工具为 vim 来解决这个报错：

```
git config --global core.editor $(which vim)
```

## 参考

- [Vaprobash issue list](https://github.com/fideloper/Vaprobash/issues/494)