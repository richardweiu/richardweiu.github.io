---
title: Jenkins 的坑
date: 2017-10-27 15:45:46
tags: " Jenkins "
categories: " Devops " 
---

> ** 前言 **
　应验了一句话，出来混总是要还的，曾将跳过的坑，现在重新来踩一遍，jenkins pipeline 未能通过 github webhook 触发。

## 问题

搞了一个新的产品，同事修个细小的 bug 也让我去部署一下，就想着加一个 Jenkins 自动化部署多方便呀，坑哧坑哧的写完了 jenkinsfile，然后添加了相关的 github webhook 与 pipeline，然后兴高采烈的去测试一番，发现不管怎样这边都没有触发 build，这就奇怪了。

1.首先去查看了 github 项目中 webhook 是否有 push 信息过来

这一点没有问题：

![check-webhook](http://7xu3tw.com1.z0.glb.clouddn.com/check-webhook.png)

不仅触发了 webhook，对端还成功返回 200，说明这一个环境没有问题

2.紧接着查看 jenkins 的日志

![check-jenkins-respons](http://7xu3tw.com1.z0.glb.clouddn.com/check-jenkins-respons.png)

发现确实使用收到 github webhook 发来的请求。

那既然 github 有发出请求，jenkins 有收到请求，说明中间的数据传输是没有问题的，那么问题就出现在 jenkins 接收到请求之后为什么没有触发我的 pipeline 去执行呢？

3.看之前成功的项目成功触发的日志

![check-success](http://7xu3tw.com1.z0.glb.clouddn.com/check-success.png)

果然是因为在接收到 webhook 之后 Jenkins 没有去通知我的两个 pipeline。

查了一下果然有同甘共苦的兄弟，在 github issue 与 stackoverflow 还有 JIRA 都有抱怨：

- [github-issue](https://github.com/jenkinsci/gitlab-hook-plugin/issues/31)
- [jira-issue](https://issues.jenkins-ci.org/browse/JENKINS-35132)

那之前我是如何悄无声息的躲过这个 bug 的呢？

当时并不是一来就创建 pipeline，当时想加 CI，所以流程是这样的：

- 创建了一个 freestyle project
- 然后配置了 github 与跑 testcase 的步骤
- 创建了一个 pipeline 用于 staging 的部署，其中的配置是 freestyle project 跑完 CI 之后自动触发
- 创建了一个 pipeline 用于 product 的部署，其中的配置是 freestyle project 跑完 CI 之后自动触发(测试的时候这么做，上线的时候关掉了)

后来 CI 因为用了 circle CI 我的 CI 根本用不着所以就关掉了，但是我的 pipeline 还是成功触发了。

## 解决

看来解决方法就是很以前一样，首先创建一个项目，即使里面什么都不干，然后在它跑完之后自动去触发我之前两个不争气的 pipeline，触发之后删除 project。

果然这样操作之后我的两个 pipeline 就可以成功触发了。

这个肯定就是 github plugin 的 bug 了，之前就觉得这里很奇怪。

pipeline 可能只是一些操作，只是一个简单的脚本，所以这里没有 Source Code Management，这个可以理解，但是 Build Triggers 里面却有 Github 的触发方式，你这是要触发哪个项目。

pipeline 可以使用脚本与 github 两种管理方式，但是通过 github 管理的方式的话，这一步应该是在 Build Triggers 之后吧。

所以我觉得这里应该改一下，若是勾选 github 触发的方式，下面出现一个框填写 github repo 的信息。
