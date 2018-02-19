---
title: Modify configuration for hexo
date: 2016-05-29 12:11:55
tags: " blog "
categories: " Blog  "

---


> **前言**
　　搭建好了我们的博客平台并不代表我们就只能按部就班的有总结就更新，没总结就放着，作为一个爱折腾的人根本停不下来，每个都都有自己的审美，有时候做出的选择只是相对更符合自己口味的，自定义的才是最符合自己审美的，在有能力自己写一个主题前就修改修改一些主题，配置信息等等吧!

　　都是一些自己片面的理解，若是有遗漏，错误的观点会不断改正

### 主题的使用 ###
>　　第一步该被折腾的当然就是主题啦，每个主题对插件的支持，使用的字体，字大小等等排版都不一样，没能力写一个就先尽可能的找一个符合自己胃口的主题来调整，毕竟万事开头难，在有大体框架下做一些细节性的修改工作难度就降了很多啦！

#### 主题的选择 ####
　　好看的主题真的不少，大牛很多，有自己原创的，有在别人的主题上修改而来的，也有从其他平台移植过来的，我也是偶然间看到知乎上有个很全面的回答了解到，[这是链接](http://www.zhihu.com/question/24422335)
　　现在选择使用的是next主题，虽然感觉有些烂大街了，但是确实简洁，功能也蛮全面的，很符合胃口。这是其[GitHub地址](https://github.com/iissnan/hexo-theme- next),若有兴趣可以预览看看。
#### 主题的应用 ####
　　前期的准备工作终于做完了，可以开始捣腾了。其实修改一个主题并不困难，只要选定一个满意的主题将其应用只需要两个步骤
　　1. 使用git工具将主题的源码下载至本地的blog初始化好的theme文件夹中
```
#因为我就在博客的文件夹中输入该命令,所以直接使用theme的相对距离theme/next 
git clone https://github.com/iissnan/hexo-theme- next theme/next
```
　　2. 将主题下载至本地之后就只需要修改hexo的配置文件将其应用即可
```
vim /blog路径/_config.yml
#找到theme选项,并修改其值
theme: next 
```
#### 主题的使用 ####
　next的[使用文档](http://theme-next.iissnan.com)写的非常的详细,这记录下我的使用历程
　　1.next以同一种风格为我们写了三种使用模式,分别是Muse,Mist, Pisces
　　　+ Muse - 默认 Scheme，这是 NexT 最初的版本，黑白主调，大量留白
　　　+ Mist - Muse 的紧凑版本，整洁有序的单栏外观
　　　+ Pisces - 双栏 Scheme，小家碧玉似的清新
　　　我呢就喜欢小家碧玉的清新，很简单，清楚明了，很不错,我们只需要进入主题文件夹中同样修改_config.yml文件，将带有Scheme的关键字值修改为你想要的那种就可以了,当然官方已经为你写好了,你只需要将你需要模式前面的注释去掉即可,不需要的加上＃号注释掉，不能同时拥有两个值存在
　　2.我在手机上设置的字是尽可能的最小,win7的图标调到最小,同样我的博客文字我也将调整到合理范围内的最小,而next中的字体,基本的页面布局是在/blog路径/theme/next/source/css/_variables/base.styl这样的csss文件中,其中font size就是用来调整字的大小等参数,变量名很规范,很容易就看懂,可以做一些适当的调整
### Next的使用技巧 ###
#### 添加分类 ####
　1.首先创建页面,执行这一步之后会在source中创建出一个新的文件夹,里面放着分类页面的首页,这样才不会出现无法连接的错误提示
```
#创建一个新的页面
hexo new page categories
```
　2.进入source中刚刚创建的文件夹,编辑刚新建的页面，将页面的类型设置为categories ，主题将自动为这个页面显示所有分类。
```
title: 分类
date: 2014-12-22 12:39:04
type: "categories"
---
```
　3.在菜单中添加链接。编辑主题的 _config.yml ，将 menu 中的 categories: /categories 注释去掉，如下:
```
menu:
  home: /
    categories: /categories
	archives: /archives
	tags: /tags
```
　4.默认的新文章模板中并无该选项，所以可以修改模板为其加上
```
vim scaffolds/post.md 

---
title: {{ title }}
date: {{ date }}
tags:
categories:
---
```
#### 添加标签 ####
　添加标签的方法与上面的方法相同，只需要将所有的categories都改成tag即可，这里将不在赘述

以上的所有信息，在[官方的使用说明文档](http://theme-next.iissnan.com/)中都有详细的介绍可以查看
