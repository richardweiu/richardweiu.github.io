---
title: 监控工具与ntopng部署
date: 2016-06-10 19:58:35
tags: "ntopng"
categories: "运维"
---
>** 前言 **
　最开始接触流量监控的原因居然是集团公司人说我们电商部门的人搬过来之后网速变慢了,怀疑我们电商部门有人上班之余偷看电视剧，下电影什么的，最开始的想法是看是否真的有人在下载东西所以最开始就想找一款简单的查看是否有流量异常的工具

### 流行的监控 ###
　对于局域网络的监控的工具有很多很多,做一个大致的分类:(1)字符界面(2)web网页呈现。其中常用的工具有:
　(1)字符界面的:
　　1.nload(最开始用的便是这款,但是只能看到每张网卡的实时网速和总流量。不够)
　　2.nethogs(这个能够看到网卡的总流量以及流量是通过哪些程序发送的和接收的)
　　3.ethstatus(这个类似于nload，差别在于实时流量图，和一些显示的信息详尽)
　(2)web界面的:
　　1.[ntopng](http://www.ntop.org/products/traffic-analysis/ntop/):web界面呈现的话这个是我最先接触的工具了,在github上也有订阅，Ntopng是ntop的下一代，他不仅仅可以看到总的流量走势图还可以所有关联主机的流量使用以及流量用于何处,甚至流向远程主机的ip地址、还有做该主机使用流量的分类统计(比如说该主机今天一共用了10G流量,其中3G用于windows up,4G用于迅雷,3G用于浏览器等等)等等。功能很强大配置也很简单,方便。记得当时还在github建议它加入LDAP模块的管理用户登陆,在5月的时候看到是加了该功能的。还可以做发邮件通知流量异常情况，让你第一时间得到消息，只是我还没有用过。
　　它是由libcap-dev来实时捕获数据(数据包捕获函数库,由伯克利的两位教授编写的,或者直接使用PF_ring是Luca Deri发明的提高内核处理数据包效率,libcap也是用的它)、由rrdtool来将数据存储以及生成图表，由sqlite做用户登陆数据库以及rra数据与主机的登陆、用redis做缓存队列这样的基本框架来实现的,并且用bootstrap做前端的呈现。可以说ntopng就是专注于简单的流量监控而且走的很深
　　2.mrtg(接触的也不多，但是名气很大，自定义化程度高，可以生成图。但是网页的生成管理等等都需要自己管理，好像很复杂的样子，cacti就好很多了)
　　3.[cacti](http://www.cacti.net/):这个是我接触的第二个web界面呈现的网络监控程序了,因为似乎它的名气很大呀。虽然它的界面不好看(应该是很丑)，不像ntopng那样使用bootstrap做前端的呈现，很流量的分类统计和去向的追踪，但是它的优势是它的自定义化程度很高,而且他可以的不仅仅只是监控你的网卡还可以监控你的cpu使用率，内存等等。。它主要功能可以说不是流量监控，他可以做的是服务器的全方位监控，可以通过自己写模板来呈现你所需要的数据的走势图。可以说它与ntopng的着重点不同吧。而且可以做发邮件通知流量异常情况
　　而cacti是由net.snmp来实现数据的采集工作,由rrdtool来存储所采集的数据以及数据的绘图，由php/mysql实现web的呈现以及用户登陆和rra与主机对应的数据存储.cacti还有一个基于centos的Linux发行版叫做cactiEZ,就是专门做一台流量监控的server端
　　4.[nagios](https://www.nagios.org/):这个工具与cacti在企业中简直就是神一般的存在，nagios用的好不好可以直接看出这个运维的水平高不高，与cacti功能类似，可以写模板监控服务器的全方位数据，画出你想要的数据走向图，还有高警示度警告功能。
　　单纯nagios可能你以为就这样，但是他可以加各种各样的插件，厉害起来吓死你，可以我还为对他有更深入的研究也就没有权力做更多的评论
　　5.[zabbix](http://www.zabbix.org/wiki/Main_Page):与nagios相同，神器啊简直就是，当然还未来得机更多的深入研究，与nagios之间有很多争论谁才是老大哥。
　　6.[Cloud Insight](http://www.oneapm.com/ci/feature.html):Cloud Insigh 是一个通过 StatsD收集数据，使用 OpenTSDB 对性能指标进行聚合、分组、过滤，利用 highcharts 做前端展示的数据管理平台。它的呈现风格更好看（至少我很喜欢）而且功能也相当的强大。
　　而且该公司有类似于sentry这样的产品但是功能强大很多，数据归纳的更加的清晰明了，很容易检查出问题，就是有些贵。
### ntopng的部署 ###
　说了那么多牛的监控监控工具，但是我们的需求并没有这么高，只是简单的查看是否有人偷使用网络看电视剧下电影这么简单。所以当时我还是先使用的ntopng(当然还有原因就是我当时没有接触这么多,不知道这些腻害的监控工具,只是想尽快的弄出个东西找出元凶给老板一个交代，看起来好像很高效率做出他想要的结果的样子，所以当时选用了这个，其他牛都有些复杂不好弄)
#### ntopng的依赖 ####
　首当其冲的当然是依赖包的安装拉
```
sudo apt-get update #当然先更新一下源咯
sudo apt-get install libcap rrdtool librrd-dev redis-server libsqlite3-dev libmysqlclient-dev #这些是主要功能实现的库文件或者程序
sudo apt-get install subversion libglib2.0 libxml2-dev libtool autoconf automake autogen wget libhirdis-dev libgeoip-dev 
libcurl4-openssl-dev libpango1.0-dev libcairo2-dev libpng12-dev git #这些主要是一些辅助的功能库和工具等
#还需要安装nDPI,这个是通过下载源码编译安装
git clone https://github.com/ntop/nDPI.git
cd nDPI/
./autogen.sh
make
```
#### ntopng的安装 ####
　同样的ntopng本生，我们也是通过编译安装，因为这样才能安装的最新的版本，apt-get源库中的版本太老了,而且我记得似乎只有ntop的版本
```
cd ..
git clone https://github.com/ntop/ntopng.git
cd ntopng
./autogen.sh
./configure
make
make install
```
#### ntopng的启动 ####
　(1)首先确保redis是否已经启动了
　(2)然后启动它。./ntopng或者是ntopng
之前在部署sentry的时候使用过一个工具就是supervior来管理进程，你要是愿意的话也可以使用他来管理我们的ntopng.当时部署使用的时候没有写博客的想法也就忘记截图了
