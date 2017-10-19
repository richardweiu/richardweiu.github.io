---
title: Jenkins 的折腾
date: 2017-10-19 19:09:21
tags: " Jenkins "
categories: " Devops " 
---

> ** 前言 **
　公司的 Jenkins 跑起来也有 5 个月吧，当时去折腾 Jenkins 也是废了我老半天劲啊，从搭建起来跑 CI 到梳理升级流程跑 CD，到发布 staging 同时发送邮件给相关人员更新成功与更新内容。groovy 语法还好，就是 api 是在不好用。

## Jenkins 介绍与 CI／CD

一个产品的过程必然是 Design --> Develop --> Test --> Release 这样一个不断迭代的过程:

![工作流](http://7xu3tw.com1.z0.glb.clouddn.com/though-work.png)
(图片的来源[阮一峰](http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html))

所以会不断的涉及到新的代码加入，需要整体测试一遍，然后发布。

### CI 的简介

所谓的 CI（Continuous integration）也就是持续集成。

持续集成指的是，频繁地将代码集成到主干，开发人员提交了新代码之后，立刻进行构建、（单元）测试。根据测试结果，我们可以确定新代码和原有代码能否正确地集成在一起。

它的好处主要有两个：

- 快速发现错误。每完成一点更新，就集成到主干，可以快速发现错误，定位错误也比较容易。
- 防止分支大幅偏离主干。如果不是经常集成，主干又在不断更新，会导致以后集成的难度变大，甚至难以集成。

![ci-info](http://7xu3tw.com1.z0.glb.clouddn.com/ci-info.png)
(图片来自于[mindproduct](https://www.mindtheproduct.com/2016/02/what-the-hell-are-ci-cd-and-devops-a-cheatsheet-for-the-rest-of-us/))

### CD 的简介

所谓的 CD 也就是持续交付（Continuous delivery）指的是频繁地将集成后的程序部署至 staging，一个预发布的环境中供用户、质量评估的团队审核：

![cd-info](http://7xu3tw.com1.z0.glb.clouddn.com/cd-info.png)
(图片来自于[mindproduct](https://www.mindtheproduct.com/2016/02/what-the-hell-are-ci-cd-and-devops-a-cheatsheet-for-the-rest-of-us/))

还有一个 CD 也就是持续部署（Continuous Deployment），指的是在持续集成与持续交付之后确认无误部署至生产环境中，也就是我们所说的 product 环境中。

![cd-info2](http://7xu3tw.com1.z0.glb.clouddn.com/cd-info2.png)
(图片来自于[mindproduct](https://www.mindtheproduct.com/2016/02/what-the-hell-are-ci-cd-and-devops-a-cheatsheet-for-the-rest-of-us/))

整个这样的快速交付迭代，自动化实现的方式可以说 Devops 文化体现的一种方式。

Devops 这几年来炒的非常火热的一个概念，好像不说 Devops 就 out 了一样，好像不说 Devops 就不是一个好运维一样，Devops是两个词语的结合 development 与 operations，它可以是一种方法论、可以是一种工具连、可以是一种文化，这些东西说起来过于广泛。我仅仅只是将其看作一种高效的工作方式，不限于任何工具。

这是 AWS 的观点：

```
DevOps is the combination of cultural philosophies, practices, and tools that increases an organization’s ability to deliver applications and services at high velocity: evolving and improving products at a faster pace than organizations using traditional software development and infrastructure management processes. This speed enables organizations to better serve their customers and compete more effectively in the market.
```

这是一篇我比较[认同的文章](http://www.infoq.com/cn/articles/detail-analysis-of-devops)

### Jenkins 介绍

在 CI 的工具中有很多 Jenkins、Travis CI、Circle 等等，其中 Jenkins 非常的全面，也是大多数人的选择，google 一搜可以说有 90% 的人都会选择 Jenkins。

Jenkins 大部分的功能都由插件来完成，更容易扩展，因为使用的人多，社区的活跃，所以功能更加的全面，若是踩到坑也肯定会被快速的修复掉。

Jenkins 使用 java 开发的，原来的名字叫 Hudson，是 Sun在 2004 年启动的一个项目，后来 Oracle 将 Sun 收购了，在 Hudson 的归属上与开源社区谈崩了，创始人 Kohsuke Kawaguchi ，一怒之下，和社区的人其他人重起炉灶，建立了 Jenkins 社区。

Jenkins 依托于 Stapler 处理路由，依靠 Jelly 来做脚本编制与处理引擎（Jelly 是一种基于Java 技术和 XML 的脚本编制和处理引擎。Jelly 的特点是有许多基于JSTL (JSP 标准标记库，JSP Standard Tag Library）、Ant、Velocity 及其它众多工具的可执行标记。），Jelly 与 Stapler 都是由Kohsuke Kawaguchi 开发，而这个大哥就是Jenkins 的 maintainer。换言之，除了 Jenkins 基本没有什么其他的项目在使用 Jelly 与 Stapler。还有一点让人惊讶的就是 Jenkins 的数据持久化居然是采用 xml 文件的方式，所以在 long run 的 Jenkins 项目中，需要持久化的数据是惊人的，而这也导致了单个 Master 可以创建的 Job 和长连接的 Slave 的数目受到了巨大的限制,单个 Master 的 Job 数量最好不要超过 1000，单个 Master 的 Slave 数目最好不要超过 250 个。（来自于[莫源](https://yq.aliyun.com/articles/70751?spm=5176.100239.blogcont70441.23.jbC58R)）


## Jenkins 的部署

Jenkins 的部署倒是很方便，自从有了 docker，这一切都很方便，直接一个 docker pull 然后在 docker run 一下就搞定，只需要注意一下要挂载卷，使得数据持久化，别造成数据丢失一切从头开始，还有端口的映射。

Jenkins 搭建起来之后就是这样的一个界面：

![jenkins-ui](http://7xu3tw.com1.z0.glb.clouddn.com/jenkins-ui.png)

## Jenkins 的使用

从上图可以看到 Jenkins 的功能也是非常全面的从人员、项目机器的管理都是有的，还有很多通过插件的实现的功能如与 github 的连接、gitlab 的连接、ansible 的接入等等这些都需要在 Manage Plugins 中安装相关的插件，总而言之用的人多了，工具肯定就是非常全面的。

其中有一个 Blue Ocean 主要是对 pipeline 的管理，新界面比较漂亮，就是功能少点，毕竟还没有完全开发完：

![blue-ocean](http://7xu3tw.com1.z0.glb.clouddn.com/blue-ocean-ci.png)

这里就不做全面的介绍了，这样的话篇幅太长了。只记录一下主要的使用功能（其实我主要想写的是 jenkinsfile 与发邮件的部分）

### 创建一个项目

这里的使用，创建项目主要的用途是用来跑测试用例，也就是我们之前所说的 CI 中测试的部分。

首先点击左上角的 New Item:

![new-item](http://7xu3tw.com1.z0.glb.clouddn.com/new-item.png)

然后填写项目名-->选择 freestyle project：

![create-project](http://7xu3tw.com1.z0.glb.clouddn.com/create-project.png)

2.配置项目，这里就不逐步的讲解了

写了项目名，描述，选在来源，哪个分支。这里值得一提的就是我在 git 中增加了一个 Advanced checkout behaviours，这是因为默认情况下去检查代码变更的时候，默认是网络连接 10 分钟不成功就会 timeout，或者如果项目资源比较大 10 分钟 clone 不下来就会 timeout 中断调，我通过这个修改成 30 分钟：

![checkout-timeout](http://7xu3tw.com1.z0.glb.clouddn.com/checkout-timeout.png)

后来是在忍受不了了就在服务器上加了一个 proxy，配置了一下 git 的参数：

```bash
git config --global http.proxy 'socks5://127.0.0.1:1080' && git config --global https.proxy 'socks5://127.0.0.1:1080'
```

然后 Build Triggers 的出发条件我选择的是 `Github hook trigger for GITScm polling`，意味着每次 push，pr 的合并都会触发 build(当然选择这个的前提是你在 Manage Jenkins 中 configure system 里配置了 Github server，还有配置了项目的 webhook，具体的方法可以参见[github 集成](http://www.jianshu.com/p/b2ed4d23a3a9))

![github-api](http://7xu3tw.com1.z0.glb.clouddn.com/github-api.png)

紧接着在 build 中我选择了 virtualenv builder，因为我这里打算的是跑 python 的测试用例。当然若是打算与 github 深度的集成可以在 build 与 Post-build Actions 中选择 Github PR 状态的控制。

### 创建一个 Pipeline

这里创建 Pipeline 主要是用在 Delivery 与 Deployment 上。

与创建项目类似，在创建的时候选择 Pipeline，然后其中大部分的配置也类似，只有最后一部分不再是 Build，而是 Pipeline 了。为了让 Pipeline 更加灵活可控，我没有选择 Pipeline script，而是选择了 Pipeline script from SCM，因为有些项目并不是每次都需要执行，所以我都用环境变量来控制，这样所有的开发人员只需要改改相关的环境变量参数即可。

> 注意：这里的 Pipeline script 是在 web 端写死脚本，而 Pipeline script from SCM 会根据项目中 jenkinsfile 的变化而变化，我一般都是放在项目里随着 git 在更新项目时一同更新。

这样可以更加灵活的控制，让部署不用每次都浪费一些不必要的时间。

这里值得一提的是之前我的 pipeline 的触发条件是 build after other projects are built，也就是说在测试用例跑成功了之后就可以触发这里 staging 的部署。这样就是 CI --> CD 的流程。

### Jenkinsfile 的书写

终于来到本章节的重点了，本来也是给自己看的，写上面的部分时发现自己好多都忘记了。

jenkinsfile 在书写的时候有两种方式：

- Scripted Pipeline：脚本式的 node 在外，stage 在里
- Declarative Pipeline：声明式的 stage 在外， node 在里面

![jenkinsfile-mode](http://7xu3tw.com1.z0.glb.clouddn.com/jenkinsfile-mode.png)

这两种方式的选择主要在于你的 stage 需要在一台还是多台机器执行(观点来自于 [stackoverflow](https://stackoverflow.com/questions/41660427/jenkins-pipeline-node-inside-stage-vs-stage-inside-node))

选择了不同的方式当然写法就不同了，而且区别就非常的大了，之前看文档的时候就是没有太在意这点，都被坑傻了。来来回回怎么写都错，自己都绕晕掉了。像 staging 是单一的节点会更倾向于使用脚本式的写法，但是 product 环境被我改成高可用之后又是多节点，则更倾向于声明式，最后为了统一与判断成功与否来发邮件，我还是都选择了声明式。

这里根据已经写过的 jenkinsfile 来讲解，在声明式中所有的内容都包含在 pipeline 中：

```groovy
pipeline {
    stuff
}
```

agent 配置的 any，这是最外围的 agent，后续的 stages 还会单独选择单独的节点。

```groovy
agent any
```

接着就是 environment 的配置：

```groovy
    environment {
        CREATE_DB = 'no'
        UPDATE_DB = 'no'
    }
```

这个就是我上文所说的用来控制部分有时候执行有时候可以不用执行的 stages。

紧接着的是一个选项，这是用于配置防止这个 pipeline 同时执行多个：

```groovy
    options {
        disableConcurrentBuilds()
    }
```

因为代码更新的会触发一次，而我的 jenkinsfile 是放在项目里面，所以若更新代码的同时也更新了 jenkinsfile 的话会再触发一次，也就是说一共出发两次，之前代码写的不严谨这里会出现问题，虽然后面代码优化了，但是我这里还是添加上，强迫症不喜欢一个 pipeline 同时跑两遍，当然之前若是选择的 `Pipeline script` 便不会有这个问题的存在。

紧接着的就是重头戏了，`stages` 的配置，`stages` 代表的是各个需要执行的任务，其中的 `stage` item 就是各个任务中的一个：

```groovy
stages {
    stage('this stage info') {
        stuff
    }
    stage('this stage info') {
        stuff
    }
    ......
}
```

在 `stage` 中首先配置的就是 `agent` 决定此任务在那个节点上执行，然后就是 `steps` 配置该任务执行的步骤，还可以用 `when` 来判断前面设置的 environment 的值，从而来控制部分有时执行有时不执行的步骤：

```groovy
        stage('install dependence for node1') {
            agent {
                node {
                    label 'node1'
                }
            }
            when {
                environment name: 'UPDATE_ENV',
                value: 'yes'
            }
            steps {
                dir ('/home/richardwei/project'){
                    sh 'pip install -r requirements.txt'
                }
            }
        }
```

`dir()` 可以在你执行命令之前切换至某个目录非常的方便，这样就不用老是写 `cd xxx` 了，此处需要使用绝对路径，否则的话就会在 workspace 中运行，剩下的东西就很简单了，sh 模块里面输入你想执行的 shell 命令即可。

还有一个 git 模块用来更新代码：

```groovy
           steps {
                dir ('/home/richardwei/project'){
                    git url: 'git@github.com:richardweiu/project.git', branch: 'master', credentialsId: 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'
                }
            }
```

注意这里，若是你的项目资源很大，加上网络又不好，clone 超过 10 分钟没有完成就会 timeout，请机智的配置 proxy（但是现在梯子垮了）。

除了邮件模块，我用的模块居然就这么点，居然感觉自己好 low 啊。

我居然没有 script 模块像 groovy 脚本一样用 if/else，没有用 try/catch 模块，当然我也用不着。

还有最后一块，就是在 stages 所有步骤完成之后或者若是中途报错需要做的操作放在:

```groovy
post {
    success {
        stuff
    }
    failure {
        stuff
    }
}
```

我这里主要用来发送邮件，邮件的内容由模版提供：

```groovy
    post {
        success {
            emailext body:
                '''
                ${SCRIPT, template="build.template"}''',
                subject: "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) is success :)",
                recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
                to: "weizj@weizj.cn, test@weizj.cn"
        }
        failure {
            emailext body:
                '''
                ${SCRIPT, template="build.template"}''',
                subject: "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) is failure :(",
                recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
                to: "weizj@weizj.cn, test@weizj.cn"
        }
    }
```

需要注意的是这里多个收件人需要用逗号隔开，若是用分号隔开会有这样的报错：

```groovy
javax.mail.internet.AddressException: Illegal semicolon, not in group in string ``316597028@qq.com;'' at position 16
```

这样的报错是因为 java 提示没有办法解析该字符串后面的分号，他只能解析逗号（我踩坑，我骄傲）

所以完整的 jenkinsfile 就是这样的：

```groovy
pipeline {
    agent any
    environment {
        CREATE_DB = 'no'
        UPDATE_DB = 'no'
    }

    options {
        disableConcurrentBuilds()
    }

    stages {

        stage('modify the nginx config'){
            agent {
                node {
                    label 'node1'
                }
            }
            steps{
                sh 'sed -i xxxxxxxxx'
            }
        }
    }

    post {
        success {
            emailext body:
                '''
                ${SCRIPT, template="build.template"}''',
                subject: "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) is success :)",
                recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
                to: "weizj@weizj.cn, test@weizj.cn"
        }
        failure {
            emailext body:
                '''
                ${SCRIPT, template="build.template"}''',
                subject: "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) is failure :(",
                recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
                to: "weizj@weizj.cn, test@weizj.cn"
        }
    }

}
```

### Jenkinsfile 的邮件模版

这个邮件模版真的是呕心沥血啊。每次邮件的内容都是更新的内容，Jenkins 的 API 真的看懂并在这里的模版用对真的不容易啊。

模版文件放在 jenkins 的 home 目录下的 `email-templates` 目录中，若是没有则创建，你在 run jenkins 之前记得创建 jenkins 用户 `useradd -m jenkins -d xxx`，启动容器的时候挂载该目录来使得数据持久化 `docker run -d -p 8080:8080 -p 50000:50000 -v /xxxx/:/var/jenkins_home --name jenkins jenkins`

邮件模版就像 jsp 一样在 html 文件中加入 jelly。

中间的内容如何实现的我都忘记了，先贴出来以后再来补充了：

```groovy
<STYLE>
.richardwei-container {
    color: #fff;
    font-family: "Microsoft Yahei";
}

.richardwei-header {
    padding: 10px 20px;
    background: #08bf91;
}

.richardwei-content {
    width: 80%;
    max-width: 900px;
    margin: 20px auto;
    box-shadow: 0 0 1px 2px #eee;
}

div.richardwei-content-banner {
    height: 100px;
    padding: 20px;
    background: #5e5e5e;
    text-align: center;
    font-size: 22px;
}
.richardwei-content-banner-text {
    display: inline-block;
    margin: 35px 50px 0 0;
    vertical-align: top;
}
.richardwei-content-banner img {
    width: 100px;
}

.richardwei-content-body {
    padding: 50px;
    color: #333;
}
.richardwei-content-body a {
    color: #08bf91;
    text-decoration: none;
}
.richardwei-content-body hr {
    border-color: #fff;
}

.richardwei-footer {
    color: #999;
    font-size: 12px;
    text-align: center;
}
.richardwei-footer a {
    color: #08bf91;
}
</STYLE>

<BODY>
<div class="richardwei-container">
    <div class="richardwei-header">
        <div class="richardwei-logo">
            <img src="" />
        </div>
    </div>

    <div class="richardwei-content">
        <div class="richardwei-content-body">
            <p>亲爱的检测官：</p>

<%
import  hudson.model.Result;
if (build.result == Result.SUCCESS) {
    result_img = "static/e59dfe28/images/32x32/blue.gif"
} else if (build.result == Result.FAILURE) {
    result_img = "static/e59dfe28/images/32x32/red.gif"
} else if (build.result == null) {
    result_img = "static/e59dfe28/images/32x32/blue.gif"
    build.result = Result.SUCCESS
} else {
    result_img = "static/e59dfe28/images/32x32/yellow.gif"
}

if (project.name == "richardwei-staging" && build.result == Result.SUCCESS ) {
%>
            <p> staging 环境已经成功更新 </p>
<%
}
else if (project.name == "richardwei-deploy" && build.result == Result.SUCCESS ) {
%>
            <p> 线上环境已经成功更新 </p>
<%
}
else if (project.name == "richardwei" && build.result == Result.FAILURE ) {
%>
            <p> staging 环境更新失败 </p>
<%
}
else if (project.name == "richardwei-deploy" && build.result == Result.FAILURE ) {
%>
            <p> 线上环境更新失败 </p>
<%
}
else {
%>
            <p> 在 build 中 </p>
<%
}
%>
            <p valign="center">
                部署结果：
                <img src="${rooturl}${result_img}" />
                <b>${project.name} BUILD ${build.result}</b>
            </p>
            <p>部署日志：
                <a href="${rooturl}${build.url}">${rooturl}${build.url}</a>
            </p>
            <p> 部署时间：${it.timestampString} </p>
            <p> 花费时间：${build.durationString} </p>
            <p>开发人员：
<%
        for (hudson.model.Cause cause : build.causes) {
%>
            ${cause.shortDescription}
<%
        }
%>
        </p>
<%
def changeSets = build.changeSets
if(changeSets != null) {
    def hadChanges = false
%>
                <p><B>更新概要:</B></p>

<%
    changeSets.each() { p4ChangeSet ->
        p4ChangeSet.each() { cs ->
            hadChanges = true
%>
            <p><B>${cs.msgAnnotated}</B></p>
<%
        }
    }
%>
                <p><B>更新详情:</B></p>

<%
    changeSets.each() { p4ChangeSet ->
        p4ChangeSet.each() { cs ->
            hadChanges = true
%>
    <TR>
        <TD colspan="2" class="bg2">&nbsp;&nbsp;Revision <B><%= cs.metaClass.hasProperty('commitId') ? cs.commitId : cs.metaClass.hasProperty('revision') ? cs.revision :
            cs.metaClass.hasProperty('changeNumber') ? cs.changeNumber : "" %></B> by
            <B><%= cs.author %>: </B>
            <p><B>(${cs.msgAnnotated})</B></p>
        </TD>
    </TR>
<%
    cs.affectedFiles.each() { p -> %>
        <p>${p.editType.name}:${p.path}</p>
<%
            }
%>
        <hr />
<%
        }
    }
    if (hadChanges == false) {
%>
        <p>没有更新内容</p>
<%
    }
}
%>


<div class="richardwei-footer">
<p>本邮件由系统自动发出，可在<a href="${rooturl}${build.url}" target="_blank">Jenkins </a>中查看具体日志输出</p>,在<a href="" target="_blank">staging 中查看</a>
</div>
</div>
</BODY>
```

这是最后的效果：

![email-template](http://7xu3tw.com1.z0.glb.clouddn.com/email-template.png)

### 注意事项

1.在 pipeline 中用 sudo 需要密码：

为了能够在 pipeline 中使用 sudo，只有两种方式

- 在 sudo 的文件中对相关用于的相关密码使用 NOPASS
- 还有就是 echo pwd | sudo -S command

2.jenkins 下设置邮件一直不成功

报错信息：

```
 Failed to send out e-mail  com.sun.mail.smtp.SMTPSendFailedException: 501 mail from address must be same as authorization user;  nested exception is: com.sun.mail.smtp.SMTPSenderFailedException: 501 mail from address must be same as authorization user  at hudson.util.PluginServletFilter$1.doFilter(PluginServletFilter.java:95)
```

解决方法：

在设置 Jenkins URL 底下有一个文本框 System Admin e-mail address，这里要设置发送者的邮箱地址，我晕。看来以后还是要细心啊。(来自于[anxuyong](https://my.oschina.net/anxuyong/blog/353897))

3.创建管理员账户的脚本

```groovy
#!groovy
import hudson.security.*
import jenkins.model.*

def instance = Jenkins.getInstance()
def hudsonRealm = new HudsonPrivateSecurityRealm(false)
def users = hudsonRealm.getAllUsers()
users_s = users.collect { it.toString() }
if ("{{ jenkins_admin_username }}" in users_s) {
    println "Admin user already exists"
} else {
    println "--> creating local admin user"

    hudsonRealm.createAccount('{{ jenkins_admin_username }}', '{{ jenkins_admin_password }}')
    instance.setSecurityRealm(hudsonRealm)

    def strategy = new FullControlOnceLoggedInAuthorizationStrategy()
    instance.setAuthorizationStrategy(strategy)
    instance.save()
}
```

4.在模版中发送更新内容的时候

这个报错：

```groovy
No such property: changeSet for class: org.jenkinsci.plugins.workflow.job.WorkflowRun
Possible solutions: changeSets
```
解决来自于这个页面（https://issues.jenkins-ci.org/browse/JENKINS-38968）

5.安全配置

若是开源项目可以共搭建都查看使用，但是若是私人的项目一定要要求登陆才能查看相关的内容：

![jenkins-security](http://7xu3tw.com1.z0.glb.clouddn.com/jenkins-security.png)

## 参考

- [阮一峰](http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html)
- [mindtheproduct](https://www.mindtheproduct.com/2016/02/what-the-hell-are-ci-cd-and-devops-a-cheatsheet-for-the-rest-of-us/)
- [Jenkins 架构](https://www.ibm.com/developerworks/cn/java/j-lo-jenkins-plugin/index.html)
- [云栖分享，jenkins 源码解读](https://yq.aliyun.com/articles/70441)
- [github 集成](http://www.jianshu.com/p/b2ed4d23a3a9)
- [jenkins-email-ext-plugin-templates](https://github.com/jenkinsci/email-ext-plugin/tree/master/src/main/resources/hudson/plugins/emailext/templates)