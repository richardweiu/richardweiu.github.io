---
title: Nginx遇Thinkphp的水土不服
date: 2016-06-09 12:20:35
tags: ["Nginx","Thinkphp"]
categories: "运维"
---

>** 前言 **
　曾经偶然的情况下接过一个很小的私活,是一个关于养老院的简单展示类型的网站,这是第一次自己做项目,从无到有,从网站实现到服务器部署都得自己搞定,虽然不难。但是也是花了不少功夫，本文得记录一下当时在Nginx中部署Thinkphp时所遇到的一个小问题.

### 问题的背景 ###
　　问题的产生肯定是与我的搭建环境选择有关.
　　我选用的是LNMP,也就是Linux(Ubuntu)+Nginx+Mysql+Thinkphp.
　　大家更多的知道LAMP(当然这是以我们在学校没有接触的角度来说),也就是说Linux+Apache+Mysql+php.而我当时之所以选用这样的环境有一下几个原因:
　　(1)Ubuntu的选用是因为首先之前在公司中的用的服务器就是Ubuntu这个发行版,对其相对熟悉很多;其次是因为Ubuntu的普及率很高很广泛,那么大家都在用遇到问题查资料也方便很多;还有就是Ubuntu中的一些小插件呀、小工具呀、源呀等等找起来也比较的方便。
　　(2)Nginx的选用是因为首先也是先入为主,对Nginx接触相对要对一些;其次对nginx与apache做了一个比较,之间的差异还是蛮大的
>Nginx的优点
　　1.轻量级:同样的web服务比起apache占用更少的内存,CPU资源。（因为Nginx是反向代理服务器,网络模型是事件驱动,更多的是在客户端与web服务器之间起一个数据中转的作用,更多的是I/O操作自身不涉及复杂运算）
　　2.抗并发:nginx处理请求是异步非阻塞的,而apache则是阻塞型的,在高并发下nginx能够保持低资源消耗而且还高性能(nginx是很不错的前端服务器，负载性能很好,Nginx支持7层负载均衡；Nginx可能会比apache支持更高的并发,在Apache+PHP（prefork）模式下，如果PHP处理慢或者前端压力很大的情况下，很容易出现Apache进程数飙升，从而拒绝服务的现象。)
　　3.高度模块化的设计:编写模块相对来说简单
　　4.社区活跃:各种高性能模块出来的很快
　Apache的优点
　　1.rewrite:比nginx的rewrite强大
　　2.模块非常的多:基本上你能想到的都可以找得到
　　3.少bug:nginx的bug相对要多一些
　　4.超稳定(pache对php等语言的支持很好，此外apache有強大的支持网路，发展时间相对nginx更久.nginx处理动态请求是鸡肋，一般动态请求要apache去做，nginx只适合静态和反向。)
apache有先天不支持多核心處理負載雞肋的缺點，建議使用nginx做前端，後端用apache。大型網站建議用nginx自代的集群功能
此处对nginx与apache的比较参考一下的一些文章
<http://weilei0528.blog.163.com/blog/static/206807046201321810834431/>
<http://www.chinaz.com/program/2015/0424/401376.shtml>
<http://www.php100.com/html/itnews/PHPxinwen/2012/0103/9606.html>
<http://www.jb51.net/article/58852.htm>

当时的最初的想法我们是一个展示类的网站,更多的是对静态文件的处理,加上经费也不多使用Nginx更加的轻量级。所以选用了Nginx

### 问题的原因 ###
　　ThinkPHP支持通过PATHINFO和URL rewrite的方式来提供友好的URL，只需要在配置文件中设置 'URL_MODEL' => 2 即可。在Apache下只需要开启mod_rewrite模块就可以正常访问了，但是Nginx中默认是不支持PATHINFO的，所以nginx默认情况下是不支持thinkphp的。不过我们可以通过修改nginx的配置文件来让其支持thinkphp。

### 问题的解决 ###
#### 服务器的基础部署 ####
　这里面肯定有一些不妥之处,只是还未来得及深入的去研究一下
　1.首先是更新下系统的源列表和系统咯,使用云中的服务器嘛
```
sudo apt-get update
sudo apt-get upgrade
```
　2.然后就是安装一些必要的工具了
```
sudo apt-get install nginx php5-cli php5-cgi mysql-server spawn-fcgi
```
#### 服务器的配置修改 ####
　3.接下来就是修改nginx的配置来支持我们的thinkphp了
>Nginx 中的 [Location](http://nginx.org/en/docs/http/ngx_http_core_module.html#location) 指令 是[NginxHttpCoreModule](https://www.nginx.com/resources/wiki/)中重要指令。Location 指令比较简单，但却是配置 Nginx 过程中不得不去了解的。Location 指令，是用来为匹配的 URI 进行配置，URI 即语法中的"/uri/"，可以是字符串或正则表达式。但如果要使用正则表达式，则必须指定前缀。

我对配置文件做出了如下的修改:
![nginx_conf1](http://7xu3tw.com1.z0.glb.clouddn.com/nginx_conf.png)
![nginx_conf2](http://7xu3tw.com1.z0.glb.clouddn.com/nginx_conf2.png)
```
vim /etc/nginx/sites-enabled/default

server {
		listen 80 default_server;
		listen [::]:80 default_server ipv6only=on;

		root /usr/share/nginx/www;
		index index.php index.html index.htm;

		# Make site accessible from http://localhost/
		server_name localhost;

		#location / {
				# First attempt to serve request as file, then
				# as directory, then fall back to displaying a 404.
				#	try_files $uri $uri/ =404;
				# Uncomment to enable naxsi on this location
				# include /etc/nginx/naxsi.rules
		#}
		location / {        
				 if (!-e $request_filename) {
				 		rewrite ^/Public/(.*)$ /Public/$1 last; 
				 		rewrite  ^/(.*)$  /index.php/$1  last;
#	  	        		rewrite  ^/Admin/(.*)$  /Admin/index.php/$1  last;
				 		break;
		         }
		}
		location /Admin {
		         if (!-e $request_filename) {
		           		rewrite ^/Public/(.*)$ /Public/$1 last;
#                       rewrite  ^/(.*)$  /index.php/$1  last;
						rewrite  ^/Admin/(.*)$  /Admin/index.php/$1  last;
						break;
				 }
		}
		location ~ \.php/?.*$ {
				 fastcgi_pass 127.0.0.1:9000;
				 fastcgi_index index.php;
#		         include fcgi.conf;
				 include /etc/nginx/fastcgi_params;
#		         include /etc/nginx/fcgi.conf;
				 set $real_script_name $fastcgi_script_name;
				 if ($fastcgi_script_name ~ "^(.+?\.php)(/.+)$") {
												set $real_script_name $1;
				 							    set $path_info $2;
				 }
				 fastcgi_param SCRIPT_FILENAME $document_root$real_script_name;
				 fastcgi_param SCRIPT_NAME $real_script_name;
				 fastcgi_param PATH_INFO $path_info;
		}

											
}
```
　4.重启nginx来使得刚刚的配置生效起作用
```
sudo service nginx restart
```
　5.启动fastcgi php
```
spown-fcgi -a 127.0.0.1 -p 9000 -C 10 -u www-data -f /usr/bin/php-cgi
#可以在/etc/rc.local中加入该命令使其开机自启动,以防忘记而出现错误
```
>注意:
1.若是出现502的错误提示可能是你的spown-fcgi没有启动
2.若是出现404的错误提示可能是你的网站文件访问权限的问题
3.若是出现no input file specified的错误提示可能是你没有加fastcgi_param的配置,或者说是没有生效
