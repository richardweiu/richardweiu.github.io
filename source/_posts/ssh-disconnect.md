---
title: SSH 僵死
date: 2016-07-05 17:56:05
tags: [" SSH "," Linux "]
categories: " 运维 "
---

>** 前言 **
　　再次遇到同样的问题,幸好当年有留下书签，不然确实没有找到该怎么解决这个问题

## 问题 ##
　再次遇到这个问题：
 　　在本地ubuntu连接AWS的instance的时候，转身去查资料，回过头来发现用ssh连接AWS的instance的终端卡死在那里，无法动弹。你想ctrl+c没有用，敲键盘的所有按键都没有任何的作用，真是叫天天不应，叫地地不灵啊。

　这个问题只在ubuntu14.04上发生过，看到过。因为使用Mac的terminal的时候并没有这个情况发生，就算你出去玩一天回来它也不会僵死
 
## 解决 ##
　查了很多资料都是说的ssh超时，自动断开连接。应该去修改配置，
```
vim /etc/ssh/sshd_config
修改：
ClientAliveInterval 0-->60
#意思是每隔60s也就是一分钟告知server我还在连接
#这样也就不会自动断开了
```
　但是并没有什么用，而且并不能解决我当前的尴尬的局面，也就是僵死在那里都退出不了，更别说去修改配置了。
　真正的解决方法就是敲回车(Enter)+~+.
　这样就可以强制退出那个状态的了。
