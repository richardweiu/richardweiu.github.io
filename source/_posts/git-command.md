---
title: git 常用用法
date: 2017-12-29 16:53:45
tags: " Git "
categories: " Git "
---

> ** 前言 **
　零散的记录一下一些 git 的命令的使用

## git 加速

这里将提到两种加速方式：

- 浅下载：所谓的浅下载就是控制项目的大小来变向的加速我们的下载
- 代理下载：通过设置代理节点从下载速度上解决下载慢的问题

### 浅下载

有时候我们 clone 项目非常的慢是因为我们的项目太大了，所以导致我们需要下载很久，这个时候我们会考虑到浅下载。

有时候项目大不仅仅是因为项目的内容很大，还有可能是因为 commit 与 branch 很多所导致的，这个时候们就会用到 `--depth` 的参数。

通过 `--depth` 参数我们可以指定我们要下载 commit 的数量，例如：

```bash
# 只下载 linux 项目中的一个 commit
git clone --depth=1 https://github.com/torvalds/linux.git
```

在使用 `--depth` 参数的时候也就意味默认使用了 `--single-branch` 参数（只会下载一个分支，除非使用 `--no-single-branch` 参数来指明需要下载所有的分支）。

还需要注意的是若是我们的项目中有一些子项目也同时需要 clone 下来的话需要使用 `--shallow-submodules` 参数。

若是项目成功下载之后发现我们还是需要完成的 commit 信息的话，我们可以通过这样的命令来下载完整版：

```bash
git fetch --unshallow
```

当然相关的参数还有很多，例如 `--shallow-since` 可以指定下载从什么时间段开始的 commit，这里就在了解了，用到的情况很少了。

### 代理下载

若是我们的网络差到 `1 kb/s` 速度下载的话，上面的方式也不会起到什么实质上的作用，这个时候我们考虑设置 git 的代理，通过代理来提高我们的下载速度。

例如我们在本地使用的 ss 做代理，我们可以使用这样的命令来设置 git 的代理：

```bash
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
```

>注意：
>- 此时一般项目都没有下下来，所以这里使用 `--global` 参数设置全局的代理参数
>- 在 Mac 中 ss 客户端默认用的端口是 1086，根据自己使用的端口设置
>- 这是 ss 的代理方式，若是通过用户名密码登录的方式来连接 proxy 的话，格式是：`http://user:password@domain:port`
>- 加代理的方式很灵活了，我们还可以设置 ssh 的 proxy 同样能够达到目的。

## git 重现 

所谓的 git 重现便是将其他分支中的 commit 修改应用在当前的分支中，例如当前的分支与其他的分支相差较大时，此时我们想将其他分支中的某一段修改应用在当前分支的时候我们就会用这种方式。

所谓的重现是我根据其作用起的名字，主要用到的是 `cherry-pick` 命令，其用法是：

```bash
# 老得 commit hash 值在前，新的在后
git cherry-pick old-commit-hash..new-commit-hash

# 也可以直接使用分支名指定 head 的前几个 commit，这就是前两个到 head 的所有 commit
git cherry-pick test-b~2..test-b

# 若只是某一个 commit 的话，同样可以用 hash 和 branch-test 两种方式
git cherry-pick hash

# 若是需要多个 commit 的话就指定多个即可
git cherry-pick hash hash
```

这就是重现，以前才开始用 pull request 方式提交 bug 的时候很常用。

## git 暂存

有时候写代码突然需要切换分支，这个时候需要将工作区清空，否则当前做的修改会应用到其他的分支中去，但是我们这些修改都是不可丢弃的，此时我们就会用暂存。

暂存就是帮当前的所有修改、删除都保存起来，但是新建的文件并不会暂存起来，因为他没有历史版本。

暂存的命令主要是用 `stash`，例如：

```bash
# 暂存当前的所有修改
git stash

# 将暂存的内容都应用
git stash pop
```

当然还有一些相关的命令例如 `show`、`list`、`clear`、`drop` 等等。

## git 忽略

我们在提交文件的时候有时候会有所选择，有时候一些缓存或者编译文件等一些没有必要上传的文件我们会让 git 忽略它，不用检查其版本问题或者修改例如 `.pyc`。

让 git 忽略一些特定的文件，主要是通过在项目根目录中创建 `.gitignore` 文件来实现。

书写该文件的规则是：

```bash
# 忽略所有 .pyc 结尾的文件
*.pyc     

# 但 lib.pyc 除外
!lib.pyc    

# 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
/TODO

# 忽略 build/ 目录下的所有文件
build/    

# 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
doc/*.txt 
```

主要就是这样的五种情况，规则很简单，但是有时候在项目开发过程中，突然想把某些目录或文件加入忽略规则，按照上述方法定义后发现并未生效，原因是 `.gitignore` 只能忽略那些原来没有被 track 的文件，如果某些文件已经被纳入了版本管理中，则修改 `.gitignore` 是无效的。

那么解决方法就是先把本地缓存删除（改变其成未 track 状态），然后再提交：

```bash
git rm -r --cached .
git add .
git commit -m 'update .gitignore'
```

## 参考

- [stackoverflow](https://stackoverflow.com/questions/128035/how-do-i-pull-from-a-git-repository-through-an-http-proxy)