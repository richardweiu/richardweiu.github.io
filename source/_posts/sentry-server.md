---
title: Sentry Server的部署
date: 2016-06-10 14:36:24
tags: " sentry "
categories: " 运维 "
---

>** 前言 **
　sentry是用python基于Django构建的实时事件日志聚合平台，可将程序所有的exception记录下来并在一个web端中呈现出来。翻译成人类的语言来说就是主要用来实时监控你所添加需要监管的server端,出现error将第一时间在平台中收集并以邮件的形式通知你，并且将error的日志信息解读归类等等的简单处理

### 部数前注意 ###
　(1)Django是1.6的与sentry6.1.2的不兼容，这样可能会出现这样的错误信息在你部署之后:Internal Server Error
　所以官方的使用文档上会推荐或者说建议你使用virtualenv来做好环境隔离。避免影响你server端中的其他python环境，我认为若是我们专门那一台server（一般现在都用云，AWS中的instance等等）来做我们的sentry server也就不存在环境的影响也就可以不需要使用它.
>virtualenv的了解
　[VirtualEnv](https://virtualenv.pypa.io/en/stable/)用于在一台机器上创建多个独立的Python虚拟运行环境，多个Python环境相互独立，互不影响，它能够：
  1.在没有权限的情况下安装新套件
  2.不同应用可以使用不同的套件版本
  3.套件升级不影响其他应用
　虚拟环境是在Python解释器上的一个私有复制，你可以在一个隔绝的环境下安装packages，不会影响到你系统中全局的Python解释器。
　虚拟环境非常有用，因为它可以防止系统出现包管理混乱和版本冲突的问题。为每个应用程序创建一个虚拟环境可以确保应用程序只能访问它们自己使用的包，从而全局解释器只作为一个源且依然整洁干净去更多的虚拟环境。另一个好处是，虚拟环境不需要管理员权限。

　(2)sentry的依赖有很多,若是准备工作没有做好,没有都装上到后面也会报错让你补上的。
　(3)[pip](https://pip.pypa.io/en/stable/)是一个安装和管理 Python 包的工具,是 easy_install 的一个替换品。pip的安装需要setuptools 或者 distribute，如果你使用的是Python3.x那么就只能使用distribute因为Python3.x不支持setuptools。因为sentry是由python开发出来的,所以我们也将使用pip来安装以及管理它.
　这里将详细阐述分别使用virtuaenv与不使用virtuaenv两种方式的安装过程
### 使用virtuaenv ###
#### 依赖的安装 ####
　安装之前当然需要为sentry提供好其依赖的环境以及库咯
```
sudo apt-get update #更新源在本地的包的版本信息
sudo apt-get install python-setuptools #上文提到过
sudo apt-get install python-pip #安装pip管理工具
sudo apt-get install nginx postgresql redis-server 
#sentry是以nginx做其web server(准确来说是反向代理),postgresql主要用于存储用户信息,redis来做其缓存数据库
sudo apt-get install python-dev libxlt1-dev libxml2-dev zlib-dev libtfi-dev libssl-dev libpq-dev libyaml-dev lxml
sudo apt-get install psycopg2 postfix #psycopg2是python用于连接postgresql的第三方工具,postfix是邮件服务系统,可选
```
#### 环境的安装 ####
　虚拟环境的安装及其使用
```
pip install -U virtualenv
virtualenv /www/sentry #为sentry创造一个坑位
source /www/sentry/bin/activate #初始化启用环境
pip install -U sentry #sentry的安装
sentry init /www/sentry.conf.py #初始化sentry生成配置文件
```
#### 环境的配置 ####
　既然是自己安装sentry服务器当然高可自定义化咯,<a name="conf_en">自己修改配置文件咯</a>
##### 修改sentry的配置文件 #####
```
vim /www/sentry/sentry.conf.py

#第一个需要修改的是DB的使用,sentry可以使用postgresql,mysql,sqlite。默认使用的是sqlite，当然你可以使用默认的，也可以使用postgresql
#使用自己的postgresql的时候做如下的修改,在配置文件的注释中有提示使用postgresql记得安装psycppg2

default:{
		'ENGINE': 'sentry.db.postgres',
		'NAME':	'sentry',
		'USER': 'postgres'}

#接着需要修改配置的是HOST的访问,HOST板块中添加一下语句
#若是不添加词句那么你从其他的地方是无法访问的
#若是好心有条件也可以为其配置域名
ALLOWED_HOSTS = ['*']
SENTRY_URL_PREFIX = 'http://......'

#若是需要email的通知,记得修改末尾的SMTP SERVER板块的配置，若是本地则将host改为loaclhost
```
##### 修改Nginx的配置文件 #####
　使用Nginx做反向代理将访问80端口的请求悄悄的专向sentry默认的9000号端口的服务上
```
vim /etc/nginx/site-enable/default

location /{
		proxy_pass http://localhost:9000;
		proxy_redired off;
		proxy_set_header Host $host;
		proxy_set_header X-Forwarded-For $proxy_add_x;
		proxy_set_header X-Forwarded-Proto $scheme;
}

#既然修改了配置就需要重启nginx服务,让其生效
service nginx restart
```
##### postgresql的准备 #####
(1)修改pg的访问控制，不然sentry无法访问
```
vim /etc/postgresql/9.3/main/pg_hba.conf
local 		all 		postgres 		peer--->trust

#同样修改后需要重启数据库服务
service postgresql restart
```
(2)为sentry创建一个它操作的数据库（三种方法）
1.psql -U postgres create database -E utf-8 sentry
2.切换至用户postgres下然后create -E utf-8 sentry
3.直接进入psql中创建
#### sentry启动 ####
```
SENTRY_CONF=/www/sentry/sentry.conf.py sentry start/stop
```

### 不使用virtuaenv ###
#### 依赖的安装 ####
　首先当然是安装一些sentry需要依赖的环境以及库咯
```
sudo apt-get update #更新源在本地的包的版本信息
sudo apt-get install python-setuptools #上文提到过
sudo apt-get install python-pip #安装pip管理工具
sudo apt-get install nginx postgresql redis-server 
#sentry是以nginx做其web server(准确来说是反向代理),postgresql主要用于存储用户信息,redis来做其缓存数据库
sudo apt-get install python-dev libxlt1-dev libxml2-dev zlib-dev libtfi-dev libssl-dev libpq-dev libyaml-dev lxml
sudo apt-get install psycopg2 postfix #psycopg2是python用于连接postgresql的第三方工具,postfix是邮件服务系统,可选
```
#### sentry安装 ####
　既然不需要虚拟环境的隔离当然就直接安装sentry咯
```
pip install -U sentry
```
#### sentry的初始化 ####
```
mkdir ~/.sentry #总的给sentry一个家吧
sentry init ~/.sentry/sentry.conf.py #和上面相同初始化配置文件
```
#### 环境的配置 ####
　与[上文](#conf_en)相同这里就不在赘述了
#### sentry的启动 ####
```
sentry upgrate #设置初始化，创建root用户

输入root用户信息等等

sentry start #启动服务
```
>注意
　　若只是这样简单的启动没有为其启动worker进程，打开web端会有一个醒目的红色警告大标题栏提示你。

### sentry进程管理 ###
　当我们退出终端以后,并没有人来管理这些进程,就算哪天停止了服务我们也不能第一时间得知,所以官网建议我们使用supervisor来管理sentry的进程。supervisor能在sentry停止服务的第一时间重新将其启动起来。
#### 安装supervisor ####
```
sudo apt-get install supervisor
```
#### 生成supervisor配置 ####
```
echo_supervisord_conf > /etc/supervisord.conf
#若是放在默认位置以后便不用-c参数来指定配置文件的所在
```
#### 修改supervisor配置 ####
```
[program:sentry_web]
directory = /home/ubuntu/.sentry  #指出config文件的所在路径
environment = SENTRY_CONF = /path/sentry.conf.py #指出sentry的配置文件所在路径
command = sentry start #启动sentry的命令,也可以具体的来写
autostart = true #在停止服务的时候自动重启
redirect_stderr = true
stdout_logfile = syslog #将输出的日志信息放在那个文件中
stderr_logfile = syslog #将错误的日志信息输入至那个文件

[program:sentry_worker]
directory = /home/ubuntu/.sentry  #指出config文件的所在路径
environment = C_FORCE_Root = "true" #指出sentry的配置文件所在路径
command = sentry start celery worker -B #启动sentry的命令,也可以具体的来写
autostart = true #在停止服务的时候自动重启
redirect_stderr = true
stdout_logfile = syslog #将输出的日志信息放在那个文件中
stderr_logfile = syslog #将错误的日志信息输入至那个文件
```
#### 启动supervisor进程 ####
```
supervisord -c /path/config file #supervisord的配置文件路径
supervisorctl -c 同上 不加参数进入交互模式／加参数命令
```
>** 注意: **
1.出现unlink的错误说明你不是用的shutdown这样正规的方式来关闭进程需要
```
unlink /tmp/supervisor.sock
然后在启动
```
2.出现127的error,可能是config没有写正确,检查一下directory与command写正确没有

剩下的只需要在你需要监控的程序中加入sentry接口
