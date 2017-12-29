---
layout: post
title: Ubuntu 杂记
date: 2016-06-03 16:25:44
tags: " Linux "
categories: " 运维 "
---
>** 前言 **
　　有需求就有方向,之前在上班的时候学到了不少东西,很多东西只是加了书签没有写下来,也有很多写下来的但是没有分类,没有一个阶段性的总结,看起来特别的杂乱,找起来也特别的痛苦.这一篇就把一些非常基础的简单操作先记录下.Linux中很多操作也不好分类.我们可以使用该网站<http://man.linuxde.net/>来查寻命令

### Ubuntu 的时间修改 ###

#### 前言 ####

　之前在自己的电脑使用ubuntu的时候根本没有考虑这个问题,因为在装系统的时候便可以自己选择语言与时区,后来去公司,公司使用的AWS,新建instance选择ubuntu系统时区的选择可由不得我了,我记得默认的时区使用的是世界统一时间,全球标准时间(UTC) GMT Coordinated Universal Time(在英语中的英文缩写是CUT,因为各国的缩写不同,最后统一为UTC)
　使用该时间并没有任何的问题,但是在查看日志的时候,从日志中剔出有用信息来做统计的时候就很不方便了,为了统计的方便以及在所有的应用平台的统一(其他应用的忘记了有些不确定),也就都修改成中国时区的时间CST.
　我们将使用dpkg-reconfigure来修改我们的时间,dpkg-reconfigure命令是Debian Linux中重新配置已经安装过的软件包，可以将一个或者多个已安装的软件包传递给此指令，它将询问软件初次安装后的配置问题。

#### 操作 ####

　1.首先修改OS的时区，使用一下的命令
```
sudo dpkg-reconfigure tzdata
#该命令只能由有root权限的用户使用,输入之后会进入一个界面，自己选择自己需要的时区
#ntpdate cn.pool.ntp.org 与网络中的时间同步,这时中国这边最活跃的时间服务器:http://www.pool.ntp.org/zone/cn
#hwclock --systohc       set the hardware clock from the current system time
```
>注意:
　　弄清楚一个问题，ntpd与ntpdate在更新时间时有什么区别。ntpd不仅仅是时间同步服务器，他还可以做客户端与标准时间服务器进行同步时间，而且是平滑同步，并非ntpdate立即同步，在生产环境中慎用ntpdate，也正如此两者不可同时运行。
　　时钟的跃变，对于某些程序会导致很严重的问题。许多应用程序依赖连续的时钟——毕竟，这是一项常见的假定，即取得的时间是线性的，一些操作，例如数据库事务，通常会地依赖这样的事实：时间不会往回跳跃。不幸的是，ntpdate调整时间的方式就是我们所说的”跃变“：在获得一个时间之后，ntpdate使用settimeofday(2)设置系统时间，这有几个非常明显的问题：
　　第一，这样做不安全。ntpdate的设置依赖于ntp服务器的安全性，攻击者可以利用一些软件设计上的缺陷，拿下ntp服务器并令与其同步的服务器执行某些消耗性的任务。由于ntpdate采用的方式是跳变，跟随它的服务器无法知道是否发生了异常（时间不一样的时候，唯一的办法是以服务器为准）。
　　第二，这样做不精确。一旦ntp服务器宕机，跟随它的服务器也就会无法同步时间。与此不同，ntpd不仅能够校准计算机的时间，而且能够校准计算机的时钟。
　　第三，这样做不够优雅。由于是跳变，而不是使时间变快或变慢，依赖时序的程序会出错（例如，如果ntpdate发现你的时间快了，则可能会经历两个相同的时刻，对某些应用而言，这是致命的）。
　　因而，唯一一个可以令时间发生跳变的点，是计算机刚刚启动，但还没有启动很多服务的那个时候。其余的时候，理想的做法是使用ntpd来校准时钟，而不是调整计算机时钟上的时间。
　　NTPD 在和时间服务器的同步过程中，会把 BIOS 计时器的振荡频率偏差——或者说 Local Clock 的自然漂移(drift)——记录下来。这样即使网络有问题，本机仍然能维持一个相当精确的走时.此处从[suer0101兄弟的博客](http://blog.csdn.net/suer0101/article/details/7868813)看见的

![进入修改界面](http://7xu3tw.com1.z0.glb.clouddn.com/change_tz.png)
![选择使用的时区](http://7xu3tw.com1.z0.glb.clouddn.com/select_tz.png)
　2.我们可以使用一些命令来查看我们是否修改成功了
```
date
#这两个命令都可以查看我们是否成功修改了系统的时间
more /etc/timezone
```
　3.检验完毕，成功修改的话，需要重启系统日志服务否则里面的时间还是不正确的
```
service rsyslog restart
#rsyslog是负责syslog的程序，可以用来取代syslogd或者syslog-ng.支持输出日志数据库
#这样之后基本所有的程序的时区都应该正确了,若是postgresql不对的话，修改/etc/postgresql/postgresql.conf该项中的值为
log_timezone = "PRC"
```
### Ubuntu 的版本查看 ###

#### 前言 ####

　我们很有可能接触一台新的设备,并想为其安装某些软件以方便使用,这个时候需要选择是32位版本还是64位版本了.
　同样,为了某些更好的服务,或者对该设备进一步的了解,我们很可能还需要当前系统的版本信息号.

#### 操作 ####

　1. 如何判断Ubuntu是32 or 64位的版本
```
uname --m #若显示为i386则为32位,若是x86_64则为64位
#当然我们也可以使用uname -a显示所有该系统的信息
```
![系统信息](http://7xu3tw.com1.z0.glb.clouddn.com/system_info.png)
　2. 如何查看系统的版本以及发布信息(该方法与上面查看的信息有所冗余)
```
cat /etc/issue

lsb_release -a 
#两种方法查看所得结果差不多,只是第二种稍微详细一些
```

### Ubuntu 的 IP 地址 ###

#### 前言 ####

　有时候我们是以DHCP的方式来获得我们的ip地址;
　还有的时候我们是为自己配置一个静态的固定ip地址,配置固定ip的好处是:不可避免的有时候会突然停电,而路由中有arp映射表与路由表,arp映射表有着MAC地址与ip地址的一一对应,而突然的停电然后又突然恢复,所有的机器又发出DHCP Discover申请地址,这时之前的arp映射表并没有刷新，有重新动态分配地址,这样会有很大的几率会导致地址冲突,部分机器可以上网,而部分机器无法上网（这事是原来在公司的时候发生的事情,整个部门中一小半人可以上网，剩下的无法上网）

#### 操作 ####

　1. 进入并修改接口文件,并重启使之生效
```
　#修改接口配置文件
　vim /etc/network/interface
　#关闭然后开启该网络接口
　ifdown 网卡名 && ifup 网卡名
　#或者是这个方法
  service networking restart
```
![修改实例](http://7xu3tw.com1.z0.glb.clouddn.com/interface_modify.png)
>ifconfig,ifup,ifdown.这3个命令的用途都是启动网络接口，不过，ifup与ifdown仅就 /etc/sysconfig/network- scripts内的ifcfg-ethx（x为数字）进行启动或关闭的操作，并不能直接修改网络参数，除非手动调整ifcfg-ethx文件才行。至于 ifconfig则可以直接手动给予某个接口IP或调整其网络参数.参考[gumingpo](http://blog.sina.com.cn/s/blog_53a7e8a30100oywa.html)

### Ubuntu 的网络信息解释 ###

#### 前言 ####

　我们经常使用一个命令来查看我们的网卡信息,也就是上面所提到的ifconfig.可是上面显示的信息可不仅仅是ip地址哦，还有你的网管，子网掩码，还有你网卡的流量进出信息情况什么的等等..一些我们常用的cacti,ntopng,nagios等等一些网络监控程序的流量统计就需要这个网卡流量的详细信息哦.

#### 解释 ####

![ifconfig_information](http://7xu3tw.com1.z0.glb.clouddn.com/ifconfig_info.png)
```
eth0                  表示第一块以太网卡 
Link encap            表示该网卡位于 OSI 物理层(Physical Layer）的名称
HWaddr                表示网卡的MAC 地址（Hardware Address） 
inet addr             表示该网卡在 TCP/IP 网络中的IP(ipv4) 地址 
Bcast                 表示广播地址（Broad Address）  
UP LOOPBACK RUNNING   UP（代表网卡开启状态）RUNNING（代表网卡的网线被接上）MULTICAST（支持组播）
MTU                   表示最大传送单元，不同局域网类型的 MTU值不一定相同，对以太网来说，MTU 的默认设置是 1500 个字节(MTU这个概念指数据帧中有效载荷的最大长度， 不包括帧首部的长度.ARP和RARP数据包的长度不够46字节,要在后面补填充位,数据包长度大于拨号链路的MTU了，则需要对数据包进行分片（fragmentation）)
Metric                表示度量值，通常用于计算路由成本,跃点数(跃点数是为用于特殊网络接口的 IP 路由分配的值，用来标识与使用该路由有关的成本。例如，可以根据链接速度、跃点计数或时间延迟来计算跃点数.)
RX                    表示接收的数据包(在这种情况下，如果你看到接收和传送的包的计数(packets)增加，那有可能是系统的IP 地址出现了混乱；如果你看到大量的错误(errors)和冲突(Collisions)，那么这很有可能是网络的传输介质出了问题)  
TX                    表示发送的数据包  
collisions            表示数据包冲突的次数  
txqueuelen            表示传送列队（Transfer Queue）长度  
interrupt             表示该网卡的IRQ 中断号  
Base address          表示I/O 地址
```
(参考[abelian](http://blog.sina.com.cn/s/blog_45497dfa0100jrdl.html))

### Ubuntu 的 top 信息查看 ###

#### 前言 ####

　top命令经常用来监控linux的系统状况，比如cpu、内存的使用，程序员基本都知道这个命令.当服务器出问题的时候我们可以通过top来查看系统的信息,从而分析并定位出问题的地方,是内存泄露呀还是进程过多呀等等,还可以得知当server在高负载的时候的瓶颈所在之处,从而对程序做出针对性的优化.
　所以看懂top中的信息,用好top还是很重要的

#### 操作 ####

　在本机中top显示的数据信息如图中所示:
![top_info](http://7xu3tw.com1.z0.glb.clouddn.com/top_info.png)
```
第一行：
15:38:37 当前系统时间
5:40     系统已经运行了5小时40分钟（在这期间没有重启过）
3 users  当前有3个用户登录系统
load average: 0.50, 0.47, 0.42 load average后面的三个数分别是1分钟、5分钟、15分钟的负载情况。(load average数据是每隔5秒钟检查一次活跃的进程数，然后按特定算法计算出的数值。如果这个数除以逻辑CPU的数量，结果高于5的时候就表明系统在超负荷运转了.)
 
第二行：
Tasks 任务（进程），系统现在共有222个进程，其中处于运行中的有2个，220个在休眠（sleep），stoped状态的有0个，zombie状态（僵尸）的有0个。
 
第三行：cpu状态
16.5% us 用户空间占用CPU的百分比。
1.3%  sy 内核空间占用CPU的百分比。
0.0%  ni 改变过优先级的进程占用CPU的百分比
82.3% id 空闲CPU百分比
0.0%  wa IO等待占用CPU的百分比
0.0%  hi 硬中断（Hardware IRQ）占用CPU的百分比
0.0%  si 软中断（Software Interrupts）占用CPU的百分比
在这里CPU的使用比率和windows概念不同，如果你不理解用户空间和内核空间，需要充充电了。
 
第四行：内存状态
5995396k total 物理内存总量（5.7GB）
3692528k used 使用中的内存总量（3.5GB）
2302868k free 空闲内存总量（2.2GB）
163364k buffers 缓存的内存量 （160M）
 
第五行：swap交换分区
7811068k total 交换区总量（7.4GB）
0k used 使用的交换区总量（0）
7811068k free 空闲交换区总量（7.4GB）
1674776k cached 缓冲的交换区总量（1.6GB）
  
第六行是空行
第七行以下：各进程（任务）的状态监控
PID 进程id
USER 进程所有者
PR 进程优先级
NI nice值。负值表示高优先级，正值表示低优先级
VIRT 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
RES 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
SHR 共享内存大小，单位kb
S 进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
%CPU 上次更新到现在的CPU时间占用百分比
%MEM 进程使用的物理内存百分比
TIME+ 进程使用的CPU时间总计，单位1/100秒
COMMAND 进程名称（命令名/命令行）
多U多核CPU监控
在top基本视图中，按键盘数字1，可监控每个逻辑CPU的状况：
```
>　第四行中使用中的内存总量（used）指的是现在系统内核控制的内存数，空闲内存总量（free）是内核还未纳入其管控范围的数量。纳入内核管理的内存不见得都在使用中，还包括过去使用过的现在可以被重复利用的内存，内核并不把这些可被重新使用的内存交还到free中去，因此在linux上free内存会越来越少，但不用为此担心。
　如果出于习惯去计算可用内存数，这里有个近似的计算公式：第四行的free + 第四行的buffers + 第五行的cached，按这个公式此台服务器的可用内存
　对于内存监控，在top里我们要时刻监控第五行swap交换分区的used，如果这个数值在不断的变化，说明内核在不断进行内存和swap的数据交换，这是真正的内存不够用了。

### Ubuntu 的包管理 ###

#### 前言 ####

　起初GNU/Linux系统中只有.tar.gz。用户 必须自己编译他们想使用的每一个程序。在Debian出现之後，人们认为有必要在系统 中添加一种机 制用来管理 安装在计算机上的软件包。人们将这套系统称为dpkg。至此着名的‘package’首次在GNU/Linux上出现。不久之後红帽子也开始着 手建立自己的包管理系统 ‘rpm’。
　GNU/Linux的创造者们很快又陷入了新的窘境。他们希望通过一种快捷、实用而且高效的方式来安装软件包。这些软件包可以自动处理相互之间 的依赖关系，并且在升级过程中维护他们的配置文件 。Debian又一次充当了开路先锋的角色。她首创了APT（Advanced Packaging Tool）。这一工具後来被Conectiva 移植到红帽子系统中用于对rpm包的管理。在其他一些发行版中我们也能看到她的身影.
　“同时，apt是一个很完整和先进的软件包管理程序，使用它可以让你，又简单，又准确的找到你要的的软件包， 并且安装或卸载都很简洁。 它还可以让你的所有软件都更新到最新状态，而且也可以用来对ubuntu 进行升级。”
　aptitude与apt-get一样，是 Debian 及其衍生系统中功能极其强大的包管理工具。与 apt-get 不同的是，aptitude 在处理依赖问题上更佳一些。举例来说，aptitude 在删除一个包时，会同时删除本身所依赖的包。这样，系统中不会残留无用的包，整个系统更为干净。

#### 操作 ####

　dpkg绕过apt包管理数据库对软件包进行操作，所以你用dpkg安装过的软件包用apt可以再安装一遍，系统不知道之前安装过了，将会覆盖之前dpkg的安装。
　dpkg是用来安装.deb文件,但不会解决模块的依赖关系,且不会关心ubuntu的软件仓库内的软件,可以用于安装本地的deb文件
　apt会解决和安装模块的依赖问题,并会咨询软件仓库, 但不会安装本地的deb文件, apt是建立在dpkg之上的软件管理工具
　当时在使用ansible在aws上实现自动化部署时接触到,当时做了一定了了解，更多深入的了解并没有记录下来,以后再补充.此处参考[该博文的部分内容](http://blog.csdn.net/xiaoyanghuaban/article/details/22946987)
　常用命令:
```
安装软件包
dpkg -i   package_name.deb   #安装本地软件包，不解决依赖关系
apt-get   install    package #在线安装软件包
aptitude  install    pattern #同上
apt-get   install    package --reinstall   #重新安装软件包
apitude   reinstall  package #同上

移除软件包
dpkg     -r      package #删除软件包
apt-get  remove  package #同上
aptitude remove  package #同上
dpkg     -P              #删除软件包及配置文件
apt-get  remove  package --purge      #删除软件包及配置文件
apitude  purge   pattern #同上

自动移除软件包
apt-get autoremove #删除不再需要的软件包
注：aptitude 没有，它会自动解决这件事

清除下载的软件包
apt-get   clean   #清除 /var/cache/apt/archives 目录
aptitude  clean   #同上
apt-get   autoclean   #清除 /var/cache/apt/archives 目录，不过只清理过时的包
aptitude  autoclean         #同上编译相关   
apt-get   source    package #获取源码
apt-get   build-dep package #解决编译源码 package 的依赖关系
aptitude  build-dep pattern #解决编译源码 pattern 的依赖关系

更新源
apt-get   update #更新源
aptitude  update #同上

更新系统
apt-get   upgrade #更新已经安装的软件包
aptitude  safe-upgrade #同上
apt-get   dist-upgrade #升级系统
aptitude  full-upgrade #同上
```

### Ubuntu 的 crontab ###

#### 前言 ####

　cron = chronograph,【unix】(时钟)守护程序，(精密)计时程序.简单的说，cron在预定的时间执行预订的命令或者脚本。cron由crond守护进程和一组表（crontab文件）组成.
　crond守护进程是在系统启动时由init进程启动的，受init进程的监视，如果它不存在了，会被init进程重新启动。这个守护进程每分钟唤醒一次，并通过检查crontab文件判断需要做什么。
　每个用户有一个以用户名命名的crontab文件，存放在/var/spool/cron/crontabs目录里。若管理员允许或者禁止其他用户拥有crontab文件，则应编辑/etc/下面的cron.deny和cron.allow这两个文件来禁止或允许用户拥有自己的crontab文件。每一个用户都可以有自己的crontab文件，但在一个较大的系统中，系统管理员一般会禁止这些文件，而只在整个系统保留一个这样的文件。
　当时对crontab了解是为了做一些数据的统计,以及需要每几个星期对数据库做一次冷备,虽然为数据库做了热备但是为了以防热备在不知情的情况下断掉,更保险因此冷备也同时上,当然还有一种情况就是为开发人员提供一个更仿真的测试环境时也需要为他们做好冷备,而我不可能每隔几个星期就去做一次冷备,输一次命令而且万一我忘记了呢或者我当时不方便呢,所以这个时候crontab就显得极为的重要了.而且这样不仅效率更高还可以防止有时候的操作失误,要知道在生产服务器上输错命令,做错事是多么致命的事情吗.

#### 操作 ####

　cron在3個地方查找配置文件：
  1，/var/spool/cron/ 這個目錄下存放的是每個用戶包括root的crontab任務，每個任務以創建者的名字命名，比如tom建的crontab任務對應的文件就是/var/spool/cron/tom。
　2./etc/crontab 這個文件負責安排由系統管理員制定的維護系統以及其他任務的crontab。它的文件格式與用戶自己創建的crontab不太一样
　3./etc/cron.d/ 這個目錄用來存放任何要執行的crontab文件或腳本。一旦cron進程启動，它就會讀取配置文件，並將其保存在內存中，接着自己轉入到休眠狀態。以後每分钟會醒來一次檢查配置文件，讀取修改過的，並執行为這一刻安排的任務，然後再轉入休眠。
　这里的部分内容参考<http://www.crazelinux.com/archives/105>与<http://fanli7.net/a/bianchengyuyan/C__/20130726/400454.html>
　在这里还有人探究crontab的工作原理<http://www.yunweipai.com/archives/4479.html>
```
crontab file [-u user]-用指定的文件替代目前的crontab。
crontab-[-u user]-用标准输入替代目前的crontab.
crontab-1[user]-列出用户目前的crontab.
crontab-e[user]-编辑用户目前的crontab.
crontab-d[user]-删除用户目前的crontab.
crontab-c dir- 指定crontab的目录。
crontab文件的格式：M H D m d cmd.
crontab文件格式：
基本格式 :     *　　*　　*　　*　　*　　command
              分　 时　 日　 月　 周　 命令
第1列表示分钟1～59 每分钟用*或者 */1表示第2列表示小时1～23（0表示0点）第3列表示日期1～31第4列表示月份1～12第5列标识号星期0～6（0表示星期天）第6列要运行的命令
crontab文件的一些例子：
30 21 * * * /usr/local/etc/rc.d/lighttpd restart上面的例子表示每晚的21:30重启apache。
```

　　因为当时没有趁热打铁很多东西查了没有及时的巩固与记录,东西基本都是懂了更大概,这次更多的是回忆当时所查找的资料以及当时所应用的场景,很多东西没有太多的深入理解,原理没有搞明白所以也就没有更多自己的理解.以后会继续补充.


