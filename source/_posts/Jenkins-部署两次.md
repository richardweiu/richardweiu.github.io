---
title: Jenkins 部署两次
date: 2017-11-29 09:20:08
tags: " Jenkins "
categories: " Devops "
---

> ** 前言 **
　折腾两次了，终于有了一个能够解决问题的方案了，再次证明自己对 Jenkins 的了解还是太少了，非常的片面。通过这两次折腾主要记录如何开启 Jenkins 的 debug 日志与 Jenkins 的权限问题。

## 问题

其实之前就有注意到 Jenkins 偶尔会触发两次的情况出现，但是没有引起重视，自己猜想的原因这样的：

pipeline 设计之初只是为了在 freestyle project 的完成之后被触发，而后来增加了 github 这样的插件之后设计可能没有很完善，导致 pipeline script from scm 的 trigger 并不是每次都触发，没有触发的时候使用 freestyle project 关联的 trigger 触发的，而触发的时候就发生了一个 webhook 触发了两次 build。

感觉其设计不是很完善的地方在于 pipeline 中 Build Triggers 里有 `GitHub hook trigger for GITScm polling` 的选项，但是并没有像 freestyle project 中有 Source Code Management 的配置项，若是我的配置是：

- pipeline 选择的是 pipeline script
- Build Triggers 选择的是 `GitHub hook trigger for GITScm polling`

那 pipeline 会被哪个项目的 github webhook 所触发呢？

而即使我的 pipeline 选择的是 pipeline script from scm 也没有被触发，似乎这里的 Source Code Management 与 Build Triggers 并没有关联性。

当然这一切都是我的猜想，只有看代码才知道其真正的原因是什么，在 Jenkins 的 JIRA 上也有看到类似的 issue 提出：

- <https://issues.jenkins-ci.org/browse/JENKINS-6819>
- <https://issues.jenkins-ci.org/browse/JENKINS-17346>
- <https://issues.jenkins-ci.org/browse/JENKINS-30350>

## 解决

遇到这个问题，去解决问题的想法有这样几个：

- 打开 debug 日志，看能不能从根本上解决问题
- google 一下网上时候有类似的情况发生，是否有现成的解决方案
- 自己想解决方案

### 1.我的第一反应是打开 debug 日志，看能不能从根本上看到一些端倪，即使看不出来应该也能获得一些日志信息，这样方便我去 Google

首先编写 log 的配置参数在 jenkins 的项目目录中，`vim log.properties`

```bash
handlers=java.util.logging.ConsoleHandler
jenkins.level=FINEST
java.util.logging.ConsoleHandler.level=FINEST
```

然后重新启动 jenkins，并让其加载参数配置文件：

```bash
docker run -d -p 8181:8080 -p 50000:50000 --env JAVA_OPTS="-Djava.util.logging.config.file=/var/jenkins_home/log.properties" -v /opt/jenkins/:
/var/jenkins_home --name jenkins jenkins
```

debug 的日志信息含量很大，可能自己过于粗心或者打开的方式不对，并未能获取到与之关联的日志信息，加上自己的对 Stapler 与 jelly 的不熟悉，准确来说应该是对 Jenkins 的框架结构、执行流程都没有理清楚就来看日志信息，头有点大。因为需要快速解决暂时放弃这条路。

### 2.找现有的解决方案

找到有用的信息也就上面给出的 issue 链接。

### 3.自己想解决方案

既然是短时间内被触发两次，那么能否在发现上一 build 还在运行中就把自己关闭结束掉呢？

那么需要解决的问题有：

- 能否获取上一次的 build 的状态：
    + 若是只能获取 result 似乎无用
    + 若是能够获取 Running 的状态，那么下一个问题：
        - 如何关闭当前 building

>**注意**：这一切的获取会涉及到权限的问题，将在最后详细说明。

#### build 状态获取

首先解决第一个问题如何获取上一个 build 的状态：

```groovy
currentBuild.getPreviousBuild().result
```

这样只能获取上一次 build 的结果，当然我测试了一下在上一个 build 还没有完成的时候该值为 null，其实也可以拿来做判断的依据。

还可以通过这样的方式：

```groovy
currentBuild.rawBuild.getPreviousBuildInProgress()
```

若是为 null 则上一个 build 并不在 build 中，反之则存在。

需要提醒的是若是为空的时候直接 `echo currentBuild.rawBuild.getPreviousBuildInProgress()` 没有问题，但是不为空的时候这样 echo 是会报错，然后直接退出该 build，虽然也达到目的了但是会给我们发邮件，而且这样处理并不好。

#### 退出 build

解决了 build 状态获取的问题，进一步便是如何退出 build。

可以通过这样的方式设置当前 build 的状态：

```groovy
currentBuild.result = 'ABORTED'
```

但是该命令也仅仅是设置当前的 build 的状态，并不会时期退出，他会继续执行，只是最后的结果是不是 success 与 fail 而是 aborted。

所以在该命令之后需要添加 `error('the build maybe repeat…')` 让其退出不再继续执行。

所以最终对 jenkinsfile 的调整只是：

- 删除 disableConcurrentBuilds 的配置选项
- 在 stage 流程的最前面增加了这样的一个判断：

```groovy
        stage('get pre-build status'){
            steps {
                script {
                    if (currentBuild.rawBuild.getPreviousBuildInProgress() != null) {
                        currentBuild.result = 'ABORTED'
                        error('the build maybe repeat…')
                    }
                }
            }
        }
```

由此便解决了触发两次的问题。

### 权限问题

在 [jenkins 的官方文档](https://jenkins.io/doc/book/managing/script-approval/)中这样说明 sandbox 的：

```groovy
The sandbox only allows a subset of Groovy’s methods deemed sufficiently safe for "untrusted" access to be executed without prior approval. Scripts using the Groovy Sandbox are all subject to the same restrictions, therefore a Pipeline authored by an Administrator is subject to the restrictions as one authorized by a non-administrative user.
```

其实 sandbox 就是一个保护机制，让 script 在运行时不能使用一些较大权限的模块，若是在 sandbox 中使用较大权限的模块，会有这样的报错：

```groovy
org.jenkinsci.plugins.scriptsecurity.sandbox.RejectedAccessException: Scripts not permitted to use method org.jenkinsci.plugins.workflow.support.steps.build.RunWrapper getRawBuild
	at org.jenkinsci.plugins.scriptsecurity.sandbox.whitelists.StaticWhitelist.rejectMethod(StaticWhitelist.java:175)
	at org.jenkinsci.plugins.scriptsecurity.sandbox.groovy.SandboxInterceptor$6.reject(SandboxInterceptor.java:261)
	at org.jenkinsci.plugins.scriptsecurity.sandbox.groovy.SandboxInterceptor.onGetProperty(SandboxInterceptor.java:381)
	at org.kohsuke.groovy.sandbox.impl.Checker$6.call(Checker.java:282)
	at org.kohsuke.groovy.sandbox.impl.Checker.checkedGetProperty(Checker.java:286)
	at com.cloudbees.groovy.cps.sandbox.SandboxInvoker.getProperty(SandboxInvoker.java:29)
	at com.cloudbees.groovy.cps.impl.PropertyAccessBlock.rawGet(PropertyAccessBlock.java:20)
	at WorkflowScript.run(WorkflowScript:22)
	at ___cps.transform___(Native Method)
	at com.cloudbees.groovy.cps.impl.PropertyishBlock$ContinuationImpl.get(PropertyishBlock.java:74)
	at com.cloudbees.groovy.cps.LValueBlock$GetAdapter.receive(LValueBlock.java:30)
	at com.cloudbees.groovy.cps.impl.PropertyishBlock$ContinuationImpl.fixName(PropertyishBlock.java:66)
	at sun.reflect.GeneratedMethodAccessor195.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at com.cloudbees.groovy.cps.impl.ContinuationPtr$ContinuationImpl.receive(ContinuationPtr.java:72)
	at com.cloudbees.groovy.cps.impl.ConstantBlock.eval(ConstantBlock.java:21)
	at com.cloudbees.groovy.cps.Next.step(Next.java:83)
	at com.cloudbees.groovy.cps.Continuable$1.call(Continuable.java:174)
	at com.cloudbees.groovy.cps.Continuable$1.call(Continuable.java:163)
	at org.codehaus.groovy.runtime.GroovyCategorySupport$ThreadCategoryInfo.use(GroovyCategorySupport.java:122)
	at org.codehaus.groovy.runtime.GroovyCategorySupport.use(GroovyCategorySupport.java:261)
	at com.cloudbees.groovy.cps.Continuable.run0(Continuable.java:163)
	at org.jenkinsci.plugins.workflow.cps.SandboxContinuable.access$001(SandboxContinuable.java:19)
	at org.jenkinsci.plugins.workflow.cps.SandboxContinuable$1.call(SandboxContinuable.java:35)
	at org.jenkinsci.plugins.workflow.cps.SandboxContinuable$1.call(SandboxContinuable.java:32)
	at org.jenkinsci.plugins.scriptsecurity.sandbox.groovy.GroovySandbox.runInSandbox(GroovySandbox.java:108)
	at org.jenkinsci.plugins.workflow.cps.SandboxContinuable.run0(SandboxContinuable.java:32)
	at org.jenkinsci.plugins.workflow.cps.CpsThread.runNextChunk(CpsThread.java:174)
	at org.jenkinsci.plugins.workflow.cps.CpsThreadGroup.run(CpsThreadGroup.java:330)
	at org.jenkinsci.plugins.workflow.cps.CpsThreadGroup.access$100(CpsThreadGroup.java:82)
	at org.jenkinsci.plugins.workflow.cps.CpsThreadGroup$2.call(CpsThreadGroup.java:242)
	at org.jenkinsci.plugins.workflow.cps.CpsThreadGroup$2.call(CpsThreadGroup.java:230)
	at org.jenkinsci.plugins.workflow.cps.CpsVmExecutorService$2.call(CpsVmExecutorService.java:64)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at hudson.remoting.SingleLaneExecutorService$1.run(SingleLaneExecutorService.java:112)
	at jenkins.util.ContextResettingExecutorService$1.run(ContextResettingExecutorService.java:28)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
```

关闭的方法有两种，主要是对应 pipeline 的两种方法：

#### pipeline script 方式

这个方法较为简单就是在输入框的左下角有一个 `Use Groovy Sandbox` 的选择框：

![checkout-sandbox](http://7xu3tw.com1.z0.glb.clouddn.com/checkout-sandbox.png)

#### pipeline script from scm 方式

这个方法较为麻烦一点，因为这种方法默认就是在 groovy sandbox 中的，要想关闭 sandbox 需要分两个步骤：

- 安装 permissive script security plugin 的插件
- 添加参数 `-Dpermissive-script-security.enabled=true`

## 参考

- [判断状态的方法](https://stackoverflow.com/questions/40760716/jenkins-abort-running-build-if-new-one-is-started)
- [退出 pipeline 方法](https://stackoverflow.com/questions/42667600/abort-current-build-from-pipeline-in-jenkins)
- [关闭 sandbox](https://stackoverflow.com/questions/38276341/jenkins-ci-pipeline-scripts-not-permitted-to-use-method-groovy-lang-groovyobject)