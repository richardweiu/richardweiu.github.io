---
title: Add hotspot on Ubuntu
date: 2016-06-08 22:56:08
tags: " Ubuntu "
categories: " Ubuntu "
---

>** 前言 **
　总有机会我们会遇到这样的窘境,你的电脑可以上网,但是你的手机却没有办法,而其实你需要的是你的手机能够上网.这个时候我们将会让我们的电脑提供个热点,让我们能以此为跳板而成功的上网.

### 在windows中 ###
　这个根本就没有必要说了,方法太多了.如腾讯管家中的wifi工具,如360中的wifi工具等等.可以轻轻松松的用你的电脑提供一个热点让你的移动端或者说是其他的设备成功的上网.

### 在ubuntu中 ###
　这才是我们的重点.其实让我们的ubuntu提供一个热点其实并不难,最主要的问题在于ubuntu提供的热点其他的windows设备可以搜到并且成功的连接、其他的IOS设备可以搜到并连接上。但是我们的安卓机,却要装一下了,搜不到添加隐藏网络也无法成功。最主要的问题我们的安卓机并不支持在添加热点设置中的mode模式。所以它无法识别，无法搜索到。
　而要让安卓机能够成功的识别,能够成功的搜索到并连接上有两种方法.
#### 第一种方法 ####
　1.首先设置我们网络的连接信息,点击屏幕右上角的wifi信号标志(连接的是wifi的话)或者是上下箭头的数据传输标志(连接的是网线)，然后进入设置窗口,如图所示
![push_wifi_sigin](http://7xu3tw.com1.z0.glb.clouddn.com/pushwifi.png)
　然后会出现如下的menu,编辑网络连接
![modify_menu](http://7xu3tw.com1.z0.glb.clouddn.com/modify_menu.png)
　之后会弹出这样的一个编辑窗口我们点击添加也就是add
![add](http://7xu3tw.com1.z0.glb.clouddn.com/add_button.png)
　2.接下来我们将选择添加网络连接的用途，我们选择wifi
![choose](http://7xu3tw.com1.z0.glb.clouddn.com/choose.png)
　3.然后我们为我们的网络连接设置一个名字以及ssid还有模式的选择
![create_1](http://7xu3tw.com1.z0.glb.clouddn.com/create_1.png)
　4.为我们的wifi设置一个密码以及加密的方式
![add_pwd](http://7xu3tw.com1.z0.glb.clouddn.com/add_pwd.png)
　5.选择共享方式
![share_mode](http://7xu3tw.com1.z0.glb.clouddn.com/share_mode.png)
　6.完成之后我们点击save,这样我们就初步完成了热点的创建,windows与iphone连接一点问题的没有.我们开始为安卓机服务的准备。我们需要kde下的一款网络包，kde-nm-connection-editor，打开ubuntu-software-center,在搜索一栏搜索 network，找到 kde-nm-connection-editor，安装
![install_kde](http://7xu3tw.com1.z0.glb.clouddn.com/install_kde.png)
　7.安装完成之后我们使用终端开开启该应用,在终端中输入该软件名称.
　8.在弹出的窗口中点击之前我们所创建的热点,然后编辑它。
　9.在接下来的窗口中我们修改wifi的模式从ad-hoc修改为access point,然后点击OK就可以了(这里我盗了一个图，呵呵)
![modify_mode](http://7xu3tw.com1.z0.glb.clouddn.com/modify_mode_acc.jpg)
　10.最后一步了。马上就成功啦！接下来只需要电脑在有线联网的情况下激活刚才创建的wifi热点即可，同前，右上角打开　网络设置，选择　创建新的wifi网络(Create New Wi-Fi Network)，弹出窗口，连接(Connection)一栏中选择刚才创建的wifi热点名称,其他选项系统自动设置完成，单机OK,等待片刻后，你的android设备就可顺利搜索到你的wifi网络并连接了～～!!

#### 第二种方法 ####
　这种方法使用的是ap-hotspot,这里参考<http://www.cnblogs.com/csulennon/p/4418734.html>
  1.安装ap-hotspot
```
$ sudo add-apt-repository ppa:nilarimogard/webupd8

$ sudo apt-get update

$ sudo apt-get install hostapd

$ sudo apt-get install ap-hotspot
```
　2.配置ap-hotspot
```
$ sudo ap-hotspot configure

Detecting configuration...
Detected eth0 as the network interface connected to the Internet. Press ENTER if this is correct or enter the desired interface below (e.g.- eth0, ppp0 etc.):
	// 回车确认

Detected wlan0 as your WiFi interface. Press ENTER if this is correct or enter the desired interface (e.g.- wlan1):
	// 回车确认

Enter the desired Access Point name or press ENTER to use the default one (myhotspot):
	// 输入wifi的名字

Enter the desired WPA Passphrase below or press ENTER to use the default one (qwerty0987):
	// 输入wifi的密码

```
　3.启动wifi
```
$ sudo ap-hotspot start
```
这样就搞定了
