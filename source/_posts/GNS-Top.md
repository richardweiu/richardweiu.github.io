---
title: Graduation Design Topology by GNS3
date: 2016-06-25 13:58:10
tags: ["GNS3","Cisco"]
categories: "Cisco"
---

>** 前言 **
　　上文说道，我们选用GNS3的IOU来做我们的毕业设计，不然后有很多好的功能无法实现。此文将说道基于医院的哪些考虑而设计出这样的拓扑结构，以及我们将使用哪些协议。如何实现出来的，将会在后续的文章中更新，深入至每个协议的原理部分
　　其实大部分都相当的简单，其中花费时间最长的就是Zone Based Firewall与IPSec VPN.他们单独的资料不好找，再加上他们一起使用会出现问题,几乎找不到资料.最应该理清原理与思路的便是这两项了。

### 需求分析 ###
#### 项目背景 ####
　　近二十年，我国医疗行业飞速发展，到目前为止全国各级别的医院大约22000家，其中三甲级别医院超过1000家占全部医院的大约5.5%，二级医院超过了5000家约占27%，一级医院超过3000家约占14%，其他类型的医院超过10000家约占到54.5%。但是由于我国人口基数过大，且医疗资源分部不均衡，国民普遍存在较大的就医问题；医院的超高效运作成立亟待解决的问题，医疗系统信息化成了解决此问题的最大武器之一。
　　医疗行业面临的问题有：
　　（1）医疗资源不足且分布不均，来自患者的直接反应主要是挂号难，等报告效率低等问题；
　　（2）纸质文档的生成和储存大大的增高了医院的运营成本，来自患者的直接反应主要是看病花费高； 
　　（3）有些病情相对复杂，涉及到多方的信息数据流动，患者担心出现误诊、用错药的情况发生；
　　（4）专家需要对病历、处方的文字做繁杂的处理，不能将时间高效的花在为患者问诊的核心部分导致的就诊效率低；
　　目前，总体来看，医疗卫生行业信息化建设已取得显著的进展，呈现明显的上升势头。2009年全国两会以后，信息化投资力度不断加大。大型医院的信息化已经进入初步整合阶段，未来会保持稳定的发展速度，数字化医院集成平台会逐步建立；医院的信息化建设进入快速发展期，各类软硬件产品保持快速增长，社区卫生和公共卫生管理信息化也呈现出加速发展的态势。医院信息化已进入实质阶段，是近几年信息化建设发展最为迅速的领域之一。
　　但是，从全国范围来看，医疗系统的信息化还远未实现，还需要大量的网络建设与应用集成工程建设工作。随着以电子病历为中心的CIS系统的推广，各种医疗单位之间联网的需求将更为迫切，因为只有高带宽的网络才能实现医疗数字影像等大容量信息的畅通传输，只有安全级别高的网络才能确保电子病历的高保密性，确保病人的隐私被保护和处方的安全性。
　　而且，病人的不断增多以及医疗管理技术的落后，信息化程度的不够深入和全面，致使许多医院的医务人员面临着越来越大的压力。一方面他们需要在更短的时间内处臵更多的病人，而另一方面医院的经济效益却在不断地下滑、因医疗事故引发的医疗纠纷日趋增多。为了缓解医务人员的这些压力，目前许多医院都在积极加大信息化系统建设，具体到医院信息化工程建设上，从五个方面逐步深入加强完善，即医院信息系统（HIS）、办公管理系统（OA）、检验系统（LIS）、医学影像系统（PACS）、远程医疗系统建设，借助网络科技手段提高医疗服务质量和医务管理水平，提高医院的经济效益。
　　医疗信息化发展要经历三个阶段：医院管理信息化（HIS）阶段、临床管理信息化（CIS）阶段、局域医疗卫生服务（GMIS）阶段。
#### 设计目标 ####
　　为适应医院信息化的发展，满足日益增长的通信需求和网络的稳定运行，今天的医院网络建设比传统医院网络建设有更高的要求，本文将从医院的应用系统进行需求分析来规划出一套最适用于目标网络的拓扑结构。
　　纵观医疗信息化的应用，大致可以分为三个方面：管理信息系统、临床信息系统、其他信息系统：
　　管理信息系统的应用特点是以文字、简单图形信息为主，虽然数据量不大，但对于基础架构需要比较高的可靠性和安全性高，对服务质量也有一定的要求。
　　相比之下，临床信息系统的应用则更为丰富。除一般文本信息外，医疗应用数据包含大量图形、视频、语音信息，要求安全、可靠、保证服务质量和高性能，需要同时支持语音、视频和数据等多种业务，又要方便以后的扩展。在某些关键应用上，对于服务质量、高性能、高可用性又提出了更高的要求。
　　其他信息系统涉及对外连接，用户对远程资源的访问控制以及广域网链路服务质量是关键，因而对系统可靠性和安全性要求较高，需要同时支持语音、视频和数据等多种业务。
　　由此可见网络系统建设主要实现以下几个目标：
　　(1)实现信息共享，医院信息管理系统、办公软件等应用程序的运行，
　　(2)提供安全、可靠、高速的平台；
　　(3)通过路由实现医保联网及Internet网的联接；
　　(4)建立内部网站，培养医院文化，降低办公成本，提高办公效率；
　　(5)实现网管和网络实时监控功能，减少网络故障发生率，缩短故障处理时间。
#### 性能需求 ####
　　此可见医院的网络系统建设主要应满足以下几个方面的需求：
　　（1）带宽性能的需求，医院网络已经发展成为一个多业务承载平台。不仅要继续承载办公自动化，Web浏览等简单的数据业务，还要承载涉及医院运营的各种业务应用系统数据，以及带宽和时延都要求很高的IP电话、视频会议等多媒体业务。因此，数据流量将大大增加，尤其是对核心网络的数据交换能力提出了前所未有的要求，核心层及骨干层必须具有万兆位级带宽和处理性能，才能构筑一个畅通无阻的"高品质"大型医院网，从而适应网络规模扩大，业务量日益增长的需要。所以要建立高速骨干网络，保证各楼宇、各网段之间线速无阻塞的数据交换；
　　（2）应用服务的需求，当前的网络已经发展成为"以应用为中心"的信息基础平台，网络管理能力的要求已经上升到了业务层次，医院日常业务的处理，如：门诊及住院管理、药品管理、电子病案管理等；良好的兼容性，保障医院各管理系统软件的正常运行
　　（3）稳定可靠的需求，设备的可靠性设计：医院每天都有庞大的数据量需要传输，难免在某些端口或者设备上出现故障，所以网络的可靠性不仅要考察网络设备是否实现了关键部件的冗余备份，还要从网络设备整体设计架构、处理引擎种类等多方面去考察。业务的可靠性设计：网络设备在故障倒换过程中，是否对业务的正常运行有影响。链路的可靠性设计：以太网的链路安全来自于多路径选择，所以在医院网络建设时，要考虑网络设备是否能够提供有效的链路自愈手段，以及快速重路由协议的支持；
　　（4）网络服务质量的需求，单纯的提高带宽并不能够有效地保障数据交换的畅通无阻，所以今天的大型医院网络建设必须要考虑到网络应能够智能识别应用事件的紧急和重要程度，如视频、音频、数据流；
　　（5）灵活性的需求，医院的服务会逐步的趋于完善，未来可能有更多的科室或者部门的加入进来，因此，需要能够灵活的扩充网络容量以及提升网络服务，同时还可以实现多种方式的网络接入，以便适应未来对网络规模的扩大，以及接入模式变化的需求；
　　（6）网络安全的需求，医院的内部信息主要是以病人的病例、处方和医嘱等信息为主，而医院是有义务保障病人的信息安全，保证病人的病例、处方和医嘱信息不被没有必要的部门或人员查看，同时也要保证信息不能外传。因此，如何保证整个系统的保密性、完整性、可用性、可审核性也是医院信息系统安全性的一部分，对网络病毒、DHCP攻击，ARP攻击甚至是对于黑客的DDoS攻击等的防御，同时能够完成一些基本的对于网络的管理和一些对于网络的实时监控功能。
#### 功能需求 ####
　(1)医院网内部功能：
　　1.医院内部方面，实现医院内部的行政、财务、HR、图书馆、药房、床位、医疗器械等等的电子化管理； 
　　2.医院与上下游供应商方面，可实现医院药品、医疗器械、日常耗用品的采购的电子化管理；
　　3.医院和用户即病人的方面，实现收费、社保医疗卡划账以及信息查询等的电子化管理；
　　4.通过医疗资讯系统、医学影像系统、医疗器械数据采集系统等先进技术实现电子病历、医疗信息无纸化传递，保证病历信息的快速流通，减少病历数据的出错率，最大限度的治病救人；
5.各个科室之间和各个部门之间通过高速骨干网络，保证各楼宇、各网段之间线速无阻塞的数据交换，以达到各个科室之间的协助治疗。
　(2)Internet功能
　　1.医院网接入internet，通过建立web站点，实现医院信息在internet上的发布；方便病患对医院的结构以及各科室的优秀医生有所了解，在挂号时更针对性的选择与自己病相关的，优秀的医生；
　　2.对医院内部提供ftp、视频点播、多媒体教学，为实习医生与资历浅的医生提供学习的平台，更快的提升个人能力；
　　3.建设医院bbs，促进医院文化的健康发展；
　　4.对外部资料实现管理，实现vod（视频点播）服务等。
### 详细设计 ###
#### 网络拓扑结构设计 ####
　　我们以省医院的结构为蓝本而做出以下的设计（本来还可以结构再大功能更全，但是GNS3太消耗资源了我的破电脑有些承载不了了）
　　层次化网络设计，能够在任何层面上都能将内外网隔离开来，通过可管理的防火墙、网闸、入侵检测等安全设备来实现基于iP的、端口的、应用的数据流管理，将复杂的网络设计分成3个层次，每个层次着重于某些特定的功能，这样就能够使一个复杂的大问题变成许多简单的小问题，易于定位，并且灵活易于扩展。在后期的管理维护上也会方便许多
　　整个个结构规划成三层架构，核心-汇聚-接入。核心和汇聚之间沟通全院网络的骨干，并采用双核心双链路设计。在这种架构下，整个骨干中的任何一台核心设备、任何一条链路出现故障，都可以确保整个网络的稳定。这样的结构才能消除稳定性隐患，确保医院所有应用系统稳定运行。
　　冗余是提高网络可靠性最重要的方法（当然很多医院为了节约成本可能连三层结构都不会采用，别说冗余结构了。当然这里的仿真不需要成本的考虑，再说考虑复杂些许，再简化也方便许多），可以减少网络上由于单点故障而导致整个网络故障的可用性。
　　院中共有三栋大楼，一栋是门诊部与住院部的结合大楼，一栋是行政部与后勤部的结合大楼，还有一栋为医技楼，以这样的物理结构再以冗余设计的基本思想，也就是通过重复设置网络链路和互连设备来满足网络的可用性需求。
　　这种结构下，规划了4个汇聚点：服务器区域汇聚点、门诊楼汇聚点、行政大楼汇聚点、医技楼汇聚点。
　　这样的规划，实现了核心-汇聚-接入、双核心双链路、模块化分区。全网的整体网络转发性能最高、全网的可扩展性最高。
　　从而设计医院的网络拓扑结构如下图所示：
![Top](http://7xu3tw.com1.z0.glb.clouddn.com/gns3_desgin_top.png)
#### 网络的功能实现 ####
##### vlan的划分 #####
　　vlan的划分主要的目的就是为了减小冲突域与广播域，可以降低产生网络风暴的几率，就算发生网络风暴也不会因此而波及其他的网段甚至整个网络。而且可以提高安全性（比如受到ARP攻击的时候不是所有网段都遭殃），提高性能（减少广播报文，从而达到避免无用的广播报文占据带宽，使得提高了性能）
　　规划了4个汇聚点：服务器区域汇聚点、门诊楼汇聚点、行政大楼汇聚点、医技楼汇聚点。为了尽可能的减少每个汇聚点内部广播域，并且合理利用ip地址，门诊部门基于所属的23科室来划分子网，行政部门基于下属的4个子部门划分，医技大楼基于4个科室划分子网每个子网内有30个有效地址，不会过多浪费ip，也不会造成终端人员添加而ip不够的情况，而且每个汇聚点所占据网段划分出来的第一个子网地址将保留作为与上联设备连接的地址，并且将再细化为广播域只有两个地址的子网。划分详情如表格中所示：
![vlan](http://7xu3tw.com1.z0.glb.clouddn.com/gns3_desgin_vlan.png)
##### 二层EtherChannel #####
　　二层的EtherChannel可以在GNS3中完美的实现，但是3层的有问题，一般只能强行启动而不能自动协商，不然会出现链路震荡。导致整个网络瘫痪出现问题。
　　DS1与DS2汇聚层开启了以太捆绑二层的etherchannel，除了增加带宽外，EtherChannel还可以在多条链路上均衡分配流量，起到负载分担的作用；当一条或多条链路故障时，只要还有链路正常，流量将转移到其它的链路上，整个过程在几毫秒内完成，从而起到冗余的作用，增强了网络的稳定性和安全性。
　　以如下方式实现： 
```
DS1中的设置方式：
Enable //进入特权模式 
configure terminal //进入全局模式 
int range e0/0-1 //进入ethernet 0/0 与0/1端口下 
shutdown //首先关闭该端口 
switchport trunk encapsulation dot1q //设置该端口为trunk模式，并以802.1q协议封装 
swit mode trunk //设置该端口为二层的trunk模式 
channel-group 1 mode on //强制开启第一组以太捆绑通道 
exit //退出该端口的配置 
int port-channel 1 //进入第一组以太通道设置
switchport trunk encapsulation dot1q //与端口设置相同 
swit mode trunk 
exit 
int range e0/0-1 no shutdown //启动激活该端口 
exit 
DS2中的设置方式： 
Enable //进入特权模式 
configure terminal //进入全局模式 
int range e0/0-1 //进入ethernet 0/0 与0/1端口下 
shutdown //首先关闭该端口 
switchport trunk encapsulation dot1q //设置该端口为trunk模式，并以802.1q协议封装 
swit mode trunk //设置该端口为二层的trunk模式 
channel-group 1 mode on //强制开启第一组以太捆绑通道 
exit //退出该端口的配置 
int port-channel 1 //进入第一组以太通道设置 
switchport trunk encapsulation dot1q //与端口设置相同 
swit mode trunk 
exit 
int range e0/0-1 
no shutdown //启动激活该端口 
exit
```
##### vtp实现 #####
　　在DS1中划分vlan并开启vtp功能，供DS2与接入层交换机AS学习，从而使得vlan的管理与配置更为的简化与方便： 
　　（1）其中DS1与DS2为服务器模式，对于vlan的修改便只能在DS1与DS2中，接入层AS1与AS2则是客户端模式，只能学习而不能修改vlan的信息。 
　　（2）各个汇聚点的vlan信息应该是相互独立，不能混杂所以为其配置了域名，只有域名与口令匹配的机器之间才能相互学习vlan的信息。 
　　（3）开启修剪功能的vtp能够减少一些不必要的数据包转发，进一步的减轻链路的压力。实现如下：
```
DS1与DS2的配置相同： 
vlan 2 //划分vlan2，并进入其设置模式
name guke //为其命名为骨科 
exit //退出该vlan的设置 
vlan 3 
name neike 
exit 
vlan 4 
name waike 
exit 
vlan 5 
name pifu 
exit 
vtp mode server //开启vtp功能，并设置其模式为服务器模式 
vtp pruning //开启vtp修剪功能 
vtp domain menzhen //开启vtp的域划分，并为该与设置名字为门诊 
int range e0/2-3 //为0/2，3端口开启trunk模式 
switchport trunk encapsulation dot1q 
switchport mode trunk 
no sh 
exit 
AS1与AS2中配置 
vtp mode client //设置为客户端模式 
vtp pruning //开启修剪功能 
vtp domain menzhen //开启域模式，并命名为门诊，只有名字与服务器模式匹配才能学习
```
##### mstp的实现 #####
　　DS1和DS2所在的汇聚层与AS1和AS2所在的接入层两两相互连接从而达到冗余式设计，但是却在区域中连成环，而环状会导致广播的网络风暴，多生成树协议将环路网络修剪成为一个无环的树型网络，避免报文在环路网络中的增生和无限循环，同时还提供了数据转发的多个冗余路径，在数据转发过程中实现VLAN 数据的负载均衡。
```
在DS1中的核心配置如下： 
spanning-tree mode mst //开启mstp模式功能 
spanning-tree mst conf //进入mstp的配置模式 
name menzhen //开启域名管理，并命名为门诊 
instance 1 vlan 2-3 //设置实例1 并与vlan2、3关联 
instance 2 vlan 4-5 //设置实例2 并与vlan4、5关联 
exit //退出mstp的设置模式 
spanning-tree mst 1 root primary //设置这台交换机为实例1的根交换机 
spanning-tree mst 2 root second //设置这台交换机为实例2第二根交换机 
在DS2中的核心配置如下： 
spanning-tree mode mst //开启mstp模式功能 
spanning-tree mst conf //进入mstp的配置模式 
name menzhen //开启域名管理，并命名为门诊 
instance 1 vlan 2-3 //设置实例1 并与vlan2、3关联 
instance 2 vlan 4-5 //设置实例2 并与vlan4、5关联 
exit //退出mstp的设置模式 
spanning-tree mst 2 root primary //设置这台交换机为实例2的根交换机 
spanning-tree mst 1 root second //设置这台交换机为实例1第二根交换机 
在AS1与AS2中的核心配置如下： 
spanning-tree mode mst //开启mstp模式功能 
spanning-tree mst conf //进入mstp的配置模式 
name menzhen //开启域名管理，并命名为门诊 
instance 1 vlan 2-3 //设置实例1 并与vlan2、3关联
instance 2 vlan 4-5 //设置实例2 并与vlan4、5关联 
exit //退出mstp的设置模式
```
##### DHCP的实现 #####
　　解决了环带来的问题，我们在汇聚层开启DHCP功能，可以从地址池中为终端自动分配ip地址
```
DS1中的核心配置如下 
service dhcp //开启DHCP服务 
ip dhcp pool guke //建立一个名为骨科的地址池 
network 172.16.0.32 255.255.255.224 //分配地址的网段范围 
default-router 172.16.0.33 255.255.255.224 //告诉网关 
exit 
ip dhcp pool neike //建立一个名为内科的地址池 
network 172.16.0.64 255.255.255.224 
default-router 172.16.0.65 255.255.255.224 
exit 
ip dhcp pool waike //建立一个名为外科的地址池 
network 172.16.0.96 255.255.255.224 
default-router 172.16.0.97 255.255.255.224 
exit 
ip dhcp pool pifu//建立一个名为皮肤科的地址池 
network 172.16.0.128 255.255.255.224 
default-router 172.16.0.129 255.255.255.224 
exit 
ip dhcp excluded-add 172.16.0.33 172.16.0.35 //排除地址池中用来做网关的地址
```
##### svi接口配置 #####
　　动态分配了ip地址并告知了其网关的地址，网关本质上是一个网络通向其他网络的关卡，在网络层便是靠网关来转发不同网段，不同vlan间的数据包，从而达到不同网段，vlan间的通信，此时将使用svi来作为vlan的网关接口，以实现三层包的转发，vlan间的通信
```
DS1中核心配置如下： 
int vlan 2 //进入vlan2的接口设置 
ip address 172.16.0.33 255.255.255.224 //指定其svi的ip 
no sh //并激活该接口 
exit 
int vlan 3 
ip address 172.16.0.65 255.255.255.224 
no sh 
exit 
int vlan 4 ip address 172.16.0.97 255.255.255.224 
no sh 
int vlan 5 ip address 172.16.0.129 255.255.255.224 
no sh 
exit
```
##### vrrp的实现 #####
　　不同网段与vlan是依靠网关来相互通信，而网关则配置在一台机器的接口中，门诊楼中每日都有大量的病患，医护的网络请求必然不少，其中数据转发量大，机器时常处于高负载的情况下，机器短暂性的停止工作或者直接宕机的可能性是非常大的，所以我们以vrrp做热备份，vrrp虚拟网关冗余协议会生成一个虚拟网关，机器的网关将指向它，而该网关的背后有两个实际存在的网关，两个网关会每隔1秒组播hello包在地址为224.0.0.18，优先级高者为主网关，反之为备网关，而备网关暂时不工作，虚拟网关将指向主网关，但是3秒内若是备网关收不到主网关的信息，便认为主网关不工作，把自己切换为主网关，从而实现无缝切换，让用户感觉不到网络中发生故障，实现网关的高可靠性。
>** 注意 **
　　GNS3中实现vrrp有一个很大的bug,就是会出现双master的情况,首先出现这样的情况有可能回事模拟器的一个bug，还有一种情况是因为IOU只有以太网口,每一秒发送vrrp通告组播的数据包，在加上其他协议的广播数据包压力太大，受不了而导致状态的更新有所延迟，所以出现部分双master的情况，而双master会导致双网关的情况发生，而双网关会让中继的二层交换机凌乱，不知道数据包该往哪里转发，而导致丢包，无法进行数据通信的情况发生。

```
在DS1中的核心配置如下：
int vlan 2 //进入vlan2的配置模式下
vrrp 1 ip 172.16.0.33 //vrrp组1的虚拟网关ip地址的设置
vrrp 1 priority 150 //vrrp组1的虚拟网关的优先级设置
vrrp 1 preempt //vrrp组1的虚拟网关的抢占功能开启
no sh
exit
int vlan 3
vrrp 2 ip 172.16.0.65
vrrp 2 priority 150
vrrp 2 preempt
no sh
exit
int vlan 4
vrrp 3 ip 172.16.0.97
vrrp 3 priority 150
vrrp 3 preempt
no sh
exit
int vlan 5
vrrp 4 ip 172.16.0.129
vrrp 4 priority 150
vrrp 4 preempt
no sh
exit
在DS2中核心配置
int vlan 2
vrrp 1 ip 172.16.0.33
vrrp 1 priority 100
vrrp 1 preempt
no sh
exit
int vlan 3
vrrp 2 ip 172.16.0.65
vrrp 2 priority 100
vrrp 2 preempt
no sh
exit
int vlan 4
vrrp 3 ip 172.16.0.97
vrrp 3 priority 100
vrrp 3 preempt
no sh
exit
int vlan 5
vrrp 4 ip 172.16.0.129
vrrp 4 priority 100
vrrp 4 preempt
no sh
exit
```
##### OSPF的实现 #####
　　当各个汇聚点完成之后将上联至核心交换机上，实现全网的通信，让整个网络从离散的状态整合成一个整体，而第一个版块与第二个版块还有第三个版块并不在相同的网段中，也不是同一个vtp的vlan管理域，而此时可以看出之前对vtp实行域管理的重要性，域管理让各版块自己的vlan还是独立并不会混杂在一起。而如此的大型的网络，若是使用静态路由是不合理的，不仅工作量大而且在网络扩建和缩减的时候将非常的麻烦而且不灵活，所以就得使用ospf动态路由协议，划分为各个域，其中区域0为骨干区域，为核心包转发的区域，其他为非骨干区域，每个非骨干域为一个物理版块，这样在逻辑上他们还是相互独立的，但是域与域之间可以相互学习，做到类似于软件设计中的高内聚，低耦合，生成更大更全的路由表，各自的域会生成一棵树，所有域组合生成一个树，使得路由到各节点都是最短路径，收敛速度快。
```
在CS1中的核心配置如下
conf t
int e0/2
no switch
ip add 172.16.0.1 255.255.255.252
no sh
ex //设置各个与汇聚层相连的ip地址
int e0/3
no switch
ip add 172.16.0.5 255.255.255.252
no sh
ex
int e1/0
no switch
ip add 172.16.3.1 255.255.255.252
no sh
ex
int e1/1
no switch
ip add 172.16.3.5 255.255.255.252
no sh
ex
int e1/2
no switch
ip add 172.16.4.1 255.255.255.252
no sh
ex
int e1/3
no switch
ip add 172.16.4.5 255.255.255.252
no sh
ex
int e2/0
no switch
ip add 172.16.6.1 255.255.255.252
no sh
ex
router ospf 1 //开启ospf进程1，并进入ospf进程的配置模式
network 172.16.0.1 0.0.0.0 area 1 //将住院楼模块所连接划分为区域1
network 172.16.0.5 0.0.0.0 area 1 //将住院楼模块所连接划分为区域1
network 172.16.3.1 0.0.0.0 area 2 //将行政楼模块所连接划分为区域2
network 172.16.3.5 0.0.0.0 area 2 //将行政楼模块所连接划分为区域2
network 172.16.4.1 0.0.0.0 area 3 //将医技楼模块所连接划分为区域3
network 172.16.4.5 0.0.0.0 area 3 //将医技楼模块所连接划分为区域3
network 172.16.6.1 0.0.0.0 area 4//将服务器模块所连接划分为区域4
network 172.16.10.1 0.0.0.0 area 0 //将核心捆绑区域划分为骨干区域0
```
##### NAT的实现 #####
　　公共的ip一直是很稀缺的，医院使用电信或者联通的网络也只能得到少数的公共ip，而医院的局域网内有着成千上百的终端需要上网，这时候我们将使用NAT来实现地址转换。
　　NAT的基本工作原理是，当私有网主机和公共网主机通信的IP包经过NAT网关时，将IP包中的源IP或目的IP在私有IP和NAT的公共IP之间进行转换，让大家共享公共ip出口上网，并使用ACL（访问控制列表）来使所有汇聚点所在的网段可以访问外网，其中门诊楼除外因为门诊楼并没有对外部网络的需求，减小该汇聚点中的设备负载量
　　NAT主要可以实现数据伪装:可以将内网数据包中的地址信息更改成统一的对外地址信息，在共享上网的同时，不让内网主机直接暴露在因特网上，保证内网主机的安全。端口转发:当内网主机对外提供服务时，由于使用的是内部私有IP地址，外网无法直接访问。因此，需要在网关上进行端口转发，将特定服务的数据包转发给内网主机。
```
在PBR中的核心配置如下
conf t
int s2/0 //进入与外网相连接的串口设置下
ip add 202.106.1.2 255.255.255.0 //为其配置ip地址
ip nat outside //并定义该端口为外网端口
no sh //激活该端口
ex
int e0/0 //进入与内网相连接的端口设置之下
ip add 172.16.10.5 255.255.255.252 //为该端口设置ip地址
ip nat inside //定义该端口为内网端口
no sh //并激活该端口
ex
int e0/1
ip add 172.16.5.1 255.255.255.252
ip nat inside
no sh
ex
ip nat inside source list 1 inter s2/0 //启用内部源地址转换的PAT, 设置应用NAT的内网和外网的接口为内网地址列表1中的地址，传输所经由的外网接口为串口
access-list 1 permit 172.16.3.0 0.0.0.255 //设置访问控制地址列表
access-list 1 permit 172.16.4.0 0.0.0.255
access-list 1 permit 172.16.5.0 0.0.0.255
access-list 1 permit 172.16.6.0 0.0.0.255
access-list 1 permit 172.16.10.0 0.0.0.255
ip route 0.0.0.0 0.0.0.0 202.106.1.1 //设置静态路由
router ospf 1
network 172.16.10.3 0.0.0.0 area 0
network 172.16.11.1 0.0.0.0 area 0
default-information originate //重发布静态路由进入ospf进程中，宣告给所有ospf的区域
```
##### telnet的实现 #####
　　医院中可能有专业的数据库的维护人员或者IT维护可以留在机房中维护，但是若是在外地或者分部中人需要远端访问数据库服务器中的数据亦或者是调试，那么telnet的存在便是非常必要的了。Telnet协议是TCP/IP协议族中的一员，是Internet远程登陆服务的标准协议和主要方式。用它连接到服务器，终端使用者可以在telnet程序中输入命令，这些命令会在服务器上运行，就像直接在服务器的控制台上输入一样。可以在本地就能控制服务器。
```
在Database Server中的配置如下
Enable //进入特权模式
configure terminal //进入全局模式
enable secret wei //进入特权模式的加密口令
line vty 0 4 //设置telnet连接最大连接数为0~4共五个端口
password richard //设置使用telnet连接登陆时的密码
login //登陆时要求口令验证
transport input telnet //为了安全设置vty接入的协议只能是telnet
```
>** 注意 **
　　在GNS3中必须添加transport input telnet这句命令否则你无论怎么远程登陆都会被拒绝，默认他会拒绝所有远程协议通过，只有加了这句命令，才能使得telnet这个协议的数据包通过，而不是丢弃
##### DHCP防护实现 #####
　　DHCP都非常熟悉了，DHCP服务器欺骗攻击还是存在的，对于DHCP客户端而言，初始过程中都是通过发送广播的DHCP discovery消息寻找DHCP服务器，然而这时候如果内网中存在私设的DHCP服务器，那么就会对网络造成影响，例如客户端通过私设的DHCP服务器拿到一个非法的ip地址，可能造成ip冲突，最终导致其他终端无法上网的情况发生。而DHCP snooping将设置信任trust端口，只有trust端口可以成功转发DHCP的数据包，其他的非信任端口受到的DHCP数据包将被丢弃，若有主机成功获取到ip地址将记录在其特有的数据库中。
```
在AS1中核心配置如下
ip dhcp snooping //开启dhcp snooping功能
ip dhcp snooping vlan 2 //在vlan 2中启用dhcp snooping
ip dhcp snooping vlan 3
ip dhcp snooping vlan 4
ip dhcp snooping vlan 5
no ip dhcp snooping information option //关闭dhcp中继代理信息选项
int rang e0/0-1 //进入两个与汇聚层直连的端口
ip dhcp snooping trust //将整两个端口设置为信任端口
```
##### DAI安全防护 #####
　　在防范假冒的DHCP服务器导致网络的混乱之时，我们同样需要防范ARP中间人的攻击，医院中的信息都很重要，关乎着病人的私密信息，若是局域网中有一台计算机感染ARP木马，则感染该ARP木马的系统将会试图通过“ARP欺骗”手段截获所在网络内其它计算机的通信信息，造成病患的私密信息的泄露，同时还会因此而造成网内其它计算机的通信故障。
　　思科的Dynamic ARP Inspection (DAI)依赖于DHCP Snooping建立的绑定表，并通过检查封包的源MAC地址和IP地址来判断是否是从正确的接口发出的，如果不是则进行丢弃处理。同时因为DHCP监听绑定表已经对其进行了绑定，可以有效地防止用户私自更改指定IP地址。
```
在AS1中的配置如下：
ip dhcp snooping
ip dhcp snooping vlan 2
ip dhcp snooping vlan 3
ip dhcp snooping vlan 4
ip dhcp snooping vlan 5
no ip dhcp snooping information option
ip arp inspection vlan 2-5
int rang e0/0-1
ip dhcp snooping trust
ip arp inspection trust
```
##### AAA认证的实现 #####
　　在实现内网或者外网使用telnet远程控制控制时，仅仅只是输入密码安全性上是不够的，同时在管理上也是混乱的，所以在这个时候AAA认证的存在时有必要的。AAA便是对于身份验证(Authentication)、授权 (Authorization)以及统计(Accounting)的一个缩写，是Cisco开发的一个提供网络安全的系统。常用的AAA协议是Radius。
```
在Database中的配置如下所示：
Enable //进入特权模式
configure terminal //进入全局模式
aaa new-model //启用AAA服务
aaa authentication login default local //使用本地默认的数据库来登录
username richard password richardwei //创建一个登录用户的用户名及其密码
```
##### Zone based Firewall的实现 #####
　　局域网内的安全我们以DHCP snooping来防范，而对于与外网相连接的出口路由我们将使用类防火墙技术来做进一步的安全措施。使用Zone base Firewall(路由防火墙)，我们将整体的网络的网络分为三个区域，分别为private内网区域、dmz非军事化也就是缓冲区以及Internet外网区域，默认三个之间是不能通信的，转发的包只会被丢弃。
　　我们将各个端口根据其作用划分到各个区域中，然后使用MQC模块化命令接口来设置类别映射，此处我们的功能是内网可以访问外网和DMZ区域的服务器，以及DMZ区域的服务器与外网的连接，外网与DMZ的访问，通过class-map对流量进行分类，一共三个类别pri-to- int，pri-to- dmz，int-to- dmz。其中pri-to- int能通过的所有类型的包，但是只有符合访问控制列表的主机可以，pri-to- dmz与int-to- dmz只能通过icmp包，接着以策略映射，用policy-map对关联的流量类型进行特殊的处理以及对应上一步的类别，设置好类别与策略之后得将其对应至端口之下，而此处的端口是我们划分出来的三个区域的虚拟端口zone-pair，在未来需求变化改动也极其方便。(其实防火墙的功能并不是轻量级路由防火墙所能轻易替代的，这里没有使用ASA防火墙的主要原因是太消耗系统资源的，而我这个网络已经规划的不小了，已经带不动了,当然这个东西的资料也不好找，这个已经涉及到了CCIE的东西了)
```
其中关键的PBR设置如下：
zone security internet //划分出外网区域
exit
zone security dmz //划分出dmz区域
exit
zone security private //划分出内网区域
exit
int range e0/0-1 //进入与核心交换机相连的两个端口
zone-member security private //将该端口分配给内网区域
exit
int s2/0 //进入与外网连接的端口
zone-member security internet //将该端口分配给外网区域
exit
int e0/2 //进入与dmz相连的端口
zone-member security dmz //将其分配给dmz区域
exit
class-map type inspect match-any pri-to- int //定义内网对外网的流量类别
match access-group 1 //与ACL访问控制列表匹配
exit
class-map type inspect match-any pri-to- dmz //定义内网对dmz的流量类别
match protocol icmp //该类别通过icmp包
match protocol ipsec-msft //该类别支持加密传输的包
exit
class-map type inspect match-any int-to- dmz //定义外网对dmz的流量类别
match protocol icmp
match protocol ipsec-msft //该类别支持加密传输的包
exit
policy-map type inspect private-to- dmz //设置内网对外网的策略映射
class type inspect pri-to- dmz //匹配内网对外网的类别设置
inspect //对通过的包类别进行检查
ex
class class-default //类别默认也就是没有归类的流量设置
drop //丢弃并不处理
ex
ex
policy-map type inspect private-to- internet
class type inspect pri-to- int
inspect
ex
class class-default
drop
ex
ex
policy-map type inspect internet-to- dmz
class type inspect int-to- dmz
inspect
ex
class class-default
drop
ex
ex
//将上一步的策略映射入区域端口中，并且该端口有方向的区分
zone-pair security private-internet source private destination internet
service-policy type inspect private-to- internet
ex
zone-pair security internet-dmz source internet destination dmz
service-policy type inspect internet-to- dmz
ex
zone-pair security private-dmz source private destination dmz
service-policy type inspect private-to- dmz
ex
```
##### IPsec VPN的实现 #####
　　使用了Zone based Firewall路由防火墙我们成功的实现了内外网的隔离，保证了内网的安全，也将开发使用的服务放置于DMZ中，但是医院以外的分部，亦或者不在本地的科研中心需要访问数据库中的私密信息时，将异常的麻烦，此刻则需要IPsec VPN的使用。
　　IPsec VPN就是就是利用公网去传输私网数据与路由。对于数据传输来说，公网应该是不安全的。但是VPN可以利用隧道，加密，认证等技术来确保数据的透传，外界是很难窥视加密后的数据内容的。
　　我们实现LAN to LAN的VPN，以隧道模式传输，使用svti来实现，IPSec的实现可以分为以下5个阶段（1）指定兴趣流量:定义需要保护的流量。（2）IKE的阶段1: 协商ISAKMP参数,生成双向的SA,并用于交换密钥（2）IKE的阶段2: 协商IPSec参数, 生成二个单向SA数据通道。（3）安全数据传送: 数据通过单向SA通道进行安全传送。（5）IPSec隧道终结
```
主要在PBR与Client中配置
在PBR中的配置如下
crypto isakmp enable //启用IKE策略集
crypto isakmp policy 10 //创建策略优先级(数字越小级别越高)，并使用策略10
authentication pre-share //创建共享密钥，路由器使用预先共享的密码
hash md5 //采用md5的hash算法
encryption 3des //采用3重的des加密算法
group 1 //表示密钥使用768位密钥
lifetime 3600 //指定SA(协商安全关联)的生存周期
exit
crypto isakmp key cisco address 0.0.0.0
crypto isakmp keepalive 10 2 periodic //指定发送DPD的时间间隔为10，失效重发时间间隔为2
crypto ipsec transform-set myset esp-3des esp-md5- hmac //配置传输方式的转换集以及验证的算法以及加密的算法
exit
crypto ipsec profile myset //指定先前所定义的传输方式的转换集
set transform-set myset
exit
interface Tunnel 0 //设置tunnel端口
ip address 172.16.7.1 255.255.255.0 //设置虚拟端口的ip地址
tunnel source s2/0 //设置源地址所映射的端口
tunnel destination 203.106.1.2 //设置包指向的目的地址
tunnel mode ipsec ipv4 //设置tunnel的类型
tunnel protection ipsec profile myset //传输方式的转换集在接口上的应用
exit
router ospf 1 //将该隧道发布于ospf中
network 172.16.7.1 0.0.0.0 area 0
ip access-list extended ICMP //防火墙对VPN的数据放行添加策略
no permit icmp any host 172.16.6.34 echo
no permit icmp any host 172.16.6.34 echo-reply
no permit icmp any host 172.16.6.34 traceroute
exit
ip access-list extended ISAKMP_IPSEC
permit udp any any eq isakmp
permit ahp any any
permit esp any any
permit udp any any eq non500-isakmp
exit
class-map type inspect match-all ICMP-cmap
match access-group name ICMP
exit
class-map type inspect match-all IPSEC-cmap
match access-group name ISAKMP_IPSEC
exit
policy-map type inspect out-vpn
class type inspect ICMP-cmap
inspect
exit
class type inspect IPSEC-cmap
pass
exit
class class-default
drop
ex
ex
zone-pair security out-vpn source internet destination self
service-policy type inspect out-vpn
ex
在Client中的配置如下
conf t
int s2/1
ip add 203.106.1.2 255.255.255.0
ip nat outside
no sh
ex
int e0/0
ip add 193.168.1.1 255.255.255.0
ip nat inside
no sh
ex
ip route 0.0.0.0 0.0.0.0 203.106.1.1
ip access-list extended GoInternet
permit ip 193.168.1.0 0.0.0.255 any
exit
ip nat inside source list GoInternet interface s2/1 overload
crypto isakmp enable
crypto isakmp policy 10
authentication pre-share
hash md5
encryption 3des
group 1
lifetime 3600
exit
crypto isakmp key cisco address 0.0.0.0
crypto isakmp keepalive 10 2 periodic
crypto ipsec transform-set myset esp-3des esp-md5- hmac
exit
crypto ipsec profile myset
set transform-set myset
exit
interface Tunnel 0
ip address 172.16.7.2 255.255.255.0
tunnel source s2/1
tunnel destination 202.106.1.2
tunnel mode ipsec ipv4
tunnel protection ipsec profile myset
exit
router ospf 1
net 193.168.1.0 0.0.0.255 area 0
net 172.16.7.0 0.0.0.255 area 0
default-information originate
exit
```
　　因为在Zone based Firewall与IPsec VPN的兼容上还有当初vrrp出现双master不知道是bug的时候话费了太多的时间，导致还有些许功能只是粗略的使用并没有太过完善。
