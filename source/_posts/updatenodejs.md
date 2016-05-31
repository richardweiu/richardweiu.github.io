---
title: Update Nodejs
date: 2016-05-31 09:55:08
tags: " nodejs  "
categories: " Nodejs  "
---

> ** 前言 **
　　今天在更新博客的时候发现总是出现一长串的提示,重视了一下，发现说某些功能运行不了是因为我的版本太低的原因,结果半天没有更新成功,最后在stackoverflow和一个大牛的博客中找到答案,也记录下来,以免自己以后忘记

#### 发现异常 ####
　每次在写完blog,生成静态文件与更新上传代码的时候总是有这样的提示:
![nodejs的异常提示](http://7xu3tw.com1.z0.glb.clouddn.com/error_nodejs.png)以我蹩脚的英语看来说的应该是某个工具并没有运行起来，虽然还是在工作但是会导致效率不够高,而导致这样的异常出现是因为我的nodejs的版本似乎够高
　Nodejs可以毫不犹豫地说一个版本狂魔，时不时就发布一个版本，而且还一直没有一个1.0版本，好囧呀
#### 解决异常 ####
　解决这样的异常的方法就是把我的nodejs版本升级到最新版
```
#更新你已经安装的NPM库,这个很简单,只需要运行这样的一行命令即可
npm update -g
#更新了npm库就需要更新nodejs自身,更新为最新版或者是稳定版
npm install -g n
n latest 或者 n stable
```
　n可以下载任意版本的nodejs安装到本机，非常方便.但是使用n来切换版本也有可能出现一定的问题,还没有遇见也就不好说了
　若是没有先更新npm库就更新nodejs本生的话会出现一个error,如图所示:![update_error](http://7xu3tw.com1.z0.glb.clouddn.com/error_nodejs_version.png)
　感谢[HuangJacky](http://www.cnblogs.com/huangjacky/p/4150365.html)的博客

