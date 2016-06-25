---
title: GNS3 Use IOU
date: 2016-06-25 12:03:01
tags: ["GNS","Cisco"]
categories: "Cisco"
---

>前言
>　　因为毕业毕业的事情耽误了不少的时间，当然还有自己很懒了。自律性还是不够。感觉再不把自己做毕业设计时自学的东西记录下来就要忘记了。
　　本文的主要记录下当时做该项目时的一个工具的选择,以及其的使用

### 项目背景 ###
　　当时做选择这个毕业设计自己还是有些懵，网络规划。既然做了就得尽自己最大的努力来做这个，所以给自己定了一个较大的网络题目——大型数字化医院的网络规划与设计（当然可以做的更好，毕竟时间有限加上自己的效率问题）
### 项目工具 ###
#### 品牌的选择 ####
　　首先网络规划无非就是对整个网络的大体设计与交换机、路由器的功能性配置，所以我们需要对品牌做出选择（在实际的工作中可能会因为资金的一些问题误会选择单一的品牌，但是我们现在做的是仿真环境，所以是单一的品牌）
>这样的网络设备的品牌有不少：
　　(1)行业老大哥：思科Cisco
　　(2)历史背景复杂的：华三
　　(3)坚强独立的：华为，还有锐捷等等。
　　思科属于这个行业的领头人了，有许多自己研发出来的协议和专利，设备相当的稳定，但是有个很大问题就是价格很高。
　　华三占据了大半的中国市场（不论是企业级还是政府级），这个曾经华为与3com工具联合的产物相当的成功后来被惠普这个第三者全盘收购而去，而在14年更是差点被国企背景的中国电子收购，只是最后失败了，在15年由于紫光集团搅上。真是背景复杂
　　华为，当年在华三出了大力气当然不会就此销声匿迹，还是蛮优秀的，也有不小得市场

　　最终我还是选择了Cisco，因为第一相对来说Cisco的设备更加的稳定效果不错，第二Cisco这样的老牌当年有不少的人选择它学习他所以不管是资料的查找，学习起来感觉方便不少。第三Cisco有自己思科认证（CCNA，CCNP，CCIE）不同的认证而且有许多模拟工具可以选择，如CCNA时其官网提供的Packet Tracer便非常的好用,不仅有PC端还有移动端。
#### 工具的选择 ####
　　既然定下了品牌，那么就需要工具的选择了。
　　Packet tracer是其官网出的一款非常强大的工具，可以网络拓扑设计，Cisco相关设备模拟使用，还可以做到实时装包，看协议完成的每个步骤，并且用在OSI模型的那一层中显示出来。而且不仅仅有PC端居然还有移动端确实很牛了。但是有很多高级的实验和不属于其自己的协议无法模拟如vrrp等
　　Web IOU很厉害，在国外学习CCNP，CCIE的人很多都使用这个工具，解决了很多实验无法模拟的情况，而且轻量，不会占用太多的系统资源。但是操作起来非常的麻烦了，在PT中网络拓扑结构只需要鼠标的简单拖动即可完成，而Web IOU就必须用Visio或者其他的工具先将拓扑图设计好然后上传，在配置等等操作，很繁复(当时更需要效率也就排除了)
　　GNS3是一款使用Python+QT写的一个开源的仿真工具，相对于PT来说更多的路由实验可以实现，相对于Web IOU来说操作更加的简便。当然他也不是完美的，它消耗的系统资源太吓人了。但是综合来说还是蛮不错的，所以最后还是选择了他。
　　当然还有其他的工具NS2，NS3，omnet++，opnet，mininet。这些相对来说更加的学术一些，当然上手的话也相对难一些，我还没来得及研究也不好过多的评价。
#### 工具的安装 ####
　　GNS3的[官网](https://gns3.com/)可能被墙，不用担心可以去github中下载<https://github.com/GNS3/gns3-gui/releases>
　　简单的安装这里不做过多的介绍了，GNS3在安装的时候要注意的是集成了wireshark，SolarWinds Response Time Viewer这样的装包工具和包分析工具，而这些工具在all-in-one安装包里面的下载地址是国外的，所以安装的时候可能会出错，解决方法就是安装的时候翻墙，或者提前将其下好，单独安装好，到时候再配置进去即可。
#### 工具的使用 ####
　　GNS3的简单操作不做过多的解释，内容不少，稍微查查英文的解释便没有什么问题了，只是说并没有自带的设备，需要自己查找你所需要的设备的IOS镜像导入即可，一般3640最受欢迎，因为其即可模拟交换又可以模拟路由的实验，所以一般是不错的选择。(还是给一个详细的文档给小白看看https://yunpan.cn/cRD3ItVtLULUV  访问密码 6fe4)
![GNS3](http://7xu3tw.com1.z0.glb.clouddn.com/gns3.jpg)
　　但是IOS只能做一些一般的实验，一些高级应用的实验还是做不了，比如Zone based Firewall等等，所以这个时候我们需要使用IOU(IOS on Unix)，因为Router与Switch其系统本就在Linux系统下运行，所以我们启动一台虚拟机(装着Linux系统)让镜像尽可能在真实的运行环境中运行，这样尽可能的将不依靠硬件只需要软件实现的功能都实现出来。
　　那么搭建好IOU的必要条件有GNS3、虚拟机(VMware推荐、virtualbox)
##### 导入镜像 #####
　　有了GNS3，有了VMware，我们就需要导入[官网](https://github.com/GNS3/gns3-vm/releases)为我们准备好的GNS3的IOU系统镜像<https://github.com/GNS3/gns3-vm/releases>
![导入GNS3镜像](http://7xu3tw.com1.z0.glb.clouddn.com/import_gns3_vm.png)
![GNS3启动](http://7xu3tw.com1.z0.glb.clouddn.com/gns3_iou_vmstart.png)
![GNS3信息](http://7xu3tw.com1.z0.glb.clouddn.com/gns3_iou_information.png)
　　从图中我们看到了它所获得的ip地址，一般不会变化，但是为了保险我们可以在系统中为其配置静态ip地址，该系统是以Ubuntu修改而来的，所以命令的使用以Ubuntu为主
##### 导入认证 #####
　　导入许可文件，CiscoIOUKeygen.py。在浏览器中输入刚刚虚拟机的ip:8000/upload。然后倒入上传即可
![导入许可文件](http://7xu3tw.com1.z0.glb.clouddn.com/import_lisence.png)
##### 运行认证 #####
　　导入许可文件之后，在虚拟机中运行该程序，为我们生成认证信息。
```
cd /opt/gns3/images/IOU
python3 CiscoIOUKeygen.py
#运行之后会有许可信息生成，复制下来
#在本地新建一个iourc.txt文件，并将信息粘贴至其中
```
![gns3_iou_running_license](http://7xu3tw.com1.z0.glb.clouddn.com/gns3_iou_running_license.png)
![gns3_iou_running_license1](http://7xu3tw.com1.z0.glb.clouddn.com/gns3_iou_running_license1.png)
##### 使用认证 #####
　　打开gns3的edit->preference->IOS on UNIX->any server选择许可文件
![添加配置](http://7xu3tw.com1.z0.glb.clouddn.com/modify_gns3_pre_iou.png)
##### 添加server #####
　　再到edit->preference->server->remote server中添加虚拟机的ip地址
![修改配置](http://7xu3tw.com1.z0.glb.clouddn.com/add_gns3_iou_pre.png)
![修改配置](http://7xu3tw.com1.z0.glb.clouddn.com/add_gns3_iou_pre1.png)
##### 上传镜像 #####
　　接下来我们主需要在刚刚的上传页面中倒入我们所需要的路由器或者交换机的镜像，如第二步中的图像所示
![add_gns3_iou](http://7xu3tw.com1.z0.glb.clouddn.com/add_gns3_iou.png)
##### 添加设备 #####
　　然后在gns3中添加设备一步一步操作即可edit->preference->IOS Devices->new
![gns3_add_iou_devices](http://7xu3tw.com1.z0.glb.clouddn.com/gns3_add_iou_devices1.png)
![gns3_add_iou_devices_choose](http://7xu3tw.com1.z0.glb.clouddn.com/gns3_add_iou_devices_choose1.png)
![gns3_add_iou_devices_sure](http://7xu3tw.com1.z0.glb.clouddn.com/gns3_add_iou_devices_sure.png)
![gns3_add_iou_devices_attention](http://7xu3tw.com1.z0.glb.clouddn.com/gns3_add_iou_devices_attention.png)
　　这样我们就成功的搭建好了我们的IOU环境，添加了IOU设备，可以运行更多的实验了，截图中的我导入的L3与L2的设备都是5月份最新的设备，相对来说bug修复了不少。
　　
　　
　　
　　
　　
　　　　
