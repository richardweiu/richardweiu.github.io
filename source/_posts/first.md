---
layout: post
title: Building Github Page by hexo
date: 2016-05-23 09:59:44 +08:00
tags: " blog "
categories: " Blog  "
toc: true
---

>**前言**
>　 其实一直都想写博客来的，就是自己一直偷懒，一直拖延，拖到现在。原来上班的时候每天都会面对新的问题，然后去解决它，当时是懂了，明白了但是过一段时间遇到相同的问题又忘了，而因为找资料的时候又添加了特别多的书签，没有直接的总结性语言，找起来的特别的麻烦
>　 所以感觉很有写博客的必要，抽出时间来整理，整理自己的思路，并记录下来，忘记的时候找起来的特别的方面，自己写在本子上的记录也是乱七八糟的

　　今天就以如何搭建起Github Page作为写博客的开始好了

## Blog的了解 

### <small>&#160;&#160; 静态博客与动态博客 </small>

　　静态博客与动态博客的区别所在
　　静态博客:它是在写好文章后，在本地生成静态页面后再上传到服务器，可以避免不必要的系统维护，从而个人博主们能够更加专注于内容本身的建设，静态博客程序更加轻便快捷，减轻了服务器的负担，基本上就是和html和js打交道，没有数据库、没有后台、完全静态页面，从而大大提高了网站的打开速度，避免被数据库注入啊，跨站攻击等网站安全问题
>常见的静态博客程序有：
1.Jekyll  
　　作者:Tom Preston-Werner，Nick 等
　　主页: https://github.com/jekyll/jekyll
　　简介:Jekyll 是一种基于Ruby开发的、适用于博客的静态网站生成引擎。使用一个模板目录作为网站布局的基础框架，提供了模板、变量、插件等功能，最终生成一个完整的静态Web站点。即只要安装Jekyll的规范和结构，不需写html，便可生成网站。
2.Octopress
　　作者：Brandon Mathis
　　主页：https://github.com/imathis/octopress
　　简介：Octopress 是一款基于Ruby开发的静态化、本地化的博客系统。其最大的优势就是静态化，不依赖脚本程序，没有MysqL等数据库，因此它可在一些性能差的服务器或者虚拟空间上运行，同等条件下打开页面的速度会比较快。
3.Pelican
　　作者：getpelican团队
　　主页：https://github.com/getpelican/pelican
　　简介：Pelican是一个用Python语言编写的静态网站生成器，目前Pelican已发布3.2.2版本，有许多优秀的主题和插件可供使用，支持使用restructuredText和Markdown写文章，配置灵活，扩展性强。
4.Hexo
　　作者：tommy351
　　主页：https://github.com/hexojs/hexo/
　　简介： Hexo是一款基于node.js开发的博客程序，拥有简单的服务器，可用作简单的动态博客使用。也有生成器，生成的静态文件可以一键部署到Github Pages上，也可以部署到任意静态文件服务器上。它相当简约，并且可使用Markdown来编写文章！
5.Simple
　　作者：Rui Wang
　　主页：https://github.com/isnowfy/simple
　　简介： simple是简单的静态博客生成器，基于GithubPages，完全在线操作，不需要服务器，只需一个 Github 账号即可。简而言之，就是可以在线写blog，然后程序会自动在用户的github pages下的项目生成静态文件。

　　动态博客:基于动态语言、数据库博客那样在访问的时候从数据库读取数据动态生成页面,功能很强大,使用很方便.
>常见的动态博客程序有:
1.wordpress
　　作者:免费开源软件
　　主页:https://wordpress.org(国外官网)https://cn.wordpress.org/(国内官网)
　　简介:WordPress是一种使用PHP语言开发的博客平台，用户可以在支持PHP和MySQL数据库的服务器上架设属于自己的网站。也可以把 WordPress当作一个内容管理系统（CMS）来使用
2.z-blog
　　作者:RainbowSoft Studio团队
　　主页:http://www.zblogcn.com/
　　简介:Zblog是ASP开发的，当然也推出了php版本.支持access+mysql双数据库.国内访问量最大的独立博客，月光博客 和 卢松松博客都是ZB的.
3.bo-blog
　　作者:bo-blog团队,github上开源
　　主页:http://bw.bo-blog.com/
　　简介:Bo-blog前身基于php+txt的免费Blog程序,现基于PHP+MYSQL平台
因为了解的并不多所以没有更多的介绍了

### <small>&#160;&#160; Github与静态博客</small>

　　静态博客的流行跟著名的开源社区Github的支持是分不开的。因此，绝大部分静态博客平台设计得还是比较极客，更适合有一定编程技能的人来使用，并不适合完全零起点的完全没有编程知识的普通用户。而Github除提供在线Markdown编辑器之外，还提供了Github Page服务，可以将用户托管在Github上的Markdown博客发布为静态网站。博主完全可以将自己的博客站免费寄放在Github上。此外，通过Github发布博客，还有一个额外的好处，那就是你的博客文章可以用Git来管理，这样你就可以在Github上获得博客文章的完整发布历史（像程序员管理源代码一样），并且可以随时获取历史版本或回退和回退到特定的版本上，相当于有了强大的备份系统。
　　hexo出自台湾大学生tommy351之手，是一个基于Node.js的静态博客程序，其编译上百篇文字只需要几秒。hexo生成的静态网页可以直接放到GitHub Pages，BAE，SAE等平台上。

## Hexo的搭建

### Hexo的环境搭建

>在hexo的[官网](https://hexo.io/docs/)上有详细的安装与使用的说明文档,这里只叙述我在本机的搭建过程,我的本地环境是Ubuntu

#### NodeJs与git安装

在ubuntu上有许多中安装的方法:
(1)通过Nodejs的[官网](https://nodejs.org/en/)下载源码编译安装
(2)通过hexo的[官网](https://hexo.io/docs/)所提供的方法使用nvm安装
(3)通过添加ppa源，使用apt-get install来安装(前两种的好处在于可以安装到最新版)

这里使用ppa源安装nodejs,命令如下所示:

```
sudo add-apt-repository ppa:chris-lea/node.js
sudo apt-get update
sudo apt-get install nodejs
sudo apt-get install git
sudo npm install hexo-cli -g
#这里需要注意的是使用sudo安装的注意权限的问题，你的nodejs是属于root的，那么在使用npm的时候也得使用sudo，以后修改文件都需要sudo
#安装之后可以通过一下命令来查看nodejs的版本
nodejs -v
```

#### Github pages仓库创建

　　安装好hexo只是在本地上的环境搭建，要让你的blog能online,不仅是为了别人能够看见,更重要的是自己无论在哪里都可以看到、翻阅那么服务器端将必不可少,使用github pages免费不仅节约成本,而且与github捆绑,在备份与版本控制方面非常的方便.关于github pages其[官网](https://pages.github.com/)上有很详细的介绍.
(1)首先你必须要有一个Github的账号，没有点击这个就滚去[Github官网](https://github.com/)吧
(2)然后新创建一个仓库,并且该仓库的名字必须是:你的github用户名．github．io。在后期使用的时候这就是通往你博客的链接,当然你也可以用自己的域名,做dns解析便是.如下图所示:
![creat_new_repo](http://7xu3tw.com1.z0.glb.clouddn.com/creat_new_repo.png)
(3)图方便可以在本地做ssh,可以免密码登陆,省去每次都必须输入密码

### blog开始建立

　　(1)首先我们得在本地找一个坑位作为blog相关文件的存放之地,并初始化

```
#在自己的家目录或者找个一个顺眼的地方
mkdir Blogs
cd Blogs
#用hexo的命令初始化环境
hexo init
npm install (#在自己的家目录或者找个一个顺眼的地方

＃若是之前你使用的sudo安装的，那么这里也必须得用sudo,否则会报错你没有权限，当然你用sudo来安装的话,注意你的所有的权限以及归属组属于root的

```
看这就是初始化之后多的东西了
![hexo-init](http://7xu3tw.com1.z0.glb.clouddn.com/run_hexo_init.png)


(2)创建博文并预览
>系统默认为你创建了一篇名为helloward的文章，里面是叫你如何使用hexo来创建新的博文和一些常用的命令

```
#hexo提供了本地服务器预览,这些将是以后我们常用的命令
#将我们以markdown语法写的文章转换成html的静态网页
hexo generate
#开启本地服务，可以通过localhost:4000来访问，预览生成的网页样子
hexo server
#这个命令很少用,也会用到,清除之前生成的缓存
hexo clean
＃创建新的博文,是一个md文件,存放在source/_post中
hexo new blogname
```
### 部署至Github中

　　在博客配置文件_config.yml中,有个deployment,在这边我们进行github pages 的配置,让hexo知道以后更新了代码只收往哪里上传

```
#在配置给项目时得先装一个插件否则无法上传,会报错的
npm install hexo-deployer-git --save
#进入博客的配置文件下面,在初始化的目录中
vim _config.yml
#修改deployment,这是我的配置信息

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/richardweiu/richardweiu.github.io
  branch: master

#修改，保存好之后就可以上传了
hexo clean
hexo generate
hexo deploy
```

若是没有装 hexo-deployer-git 插件的下场就是这样
![erro-deployer](http://7xu3tw.com1.z0.glb.clouddn.com/erro_deploy.png)

上传成功之后我们变可以通过 github名字.github.io 来访问，看到我们的成果咯.hexo 的主题修改放在下一篇文章来将了.第一次写太多不熟悉了．写了很久
