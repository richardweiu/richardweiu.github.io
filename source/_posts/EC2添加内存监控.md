---
title: EC2 添加内存监控
date: 2016-06-12 00:11:31
tags: " AWS "
categories: " 运维 "
---
>** 前言 **
　上班那一阵，有一天早上开发与老板都不在，运营的人向我反应我们的后台网站打不开了，我去看了看，准确的来说是一会打开没有问题，一会儿打开有问题，具体是什么我已经忘了,好像是 404.

### 问题的原因 ###

　问题出现的原因很简单，就是因为被警告过很多次的运营人员在添加商品的解释图片时上传了像素过高，太大的图片，而导致 EC2 上的 instance 的内存不够溢出，致使后台一会儿出现问题，一会儿恢复正常（恢复正常是因为内存占用率过高之后后台程序自行重启，而恢复正常）

　但是并不知道问题的所在，首先当然是查看 Cloudwatch 的监控图像中有没有异常出现或者问题出现，但是并没有什么不对的地方呀，然后就是登陆 EC2 上去看看用 top 或者查看系统日志 syslog，但是当时就发现系统 ssh 上去都是时好时坏，最终还是趁着正常的时候看到了系统日志的信息，发现似乎确实是内存的问题(具体情况的截图并没有保存下来)。

### 添加监控的原因###

　为了以后避免类似的情况发生，为了以后更快的定位问题的所在还是决定添加一个内存的监控,Cloudwatch 默认并没有添加该服务。但是[官方文档](http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/mon-scripts.html)却提供了方法

### 添加监控 ###

#### 安装依赖 ####

```
sudo apt-get update

sudo apt-get install unzip #因为官网提供的脚本下载下来是用zip压缩的,所以需要该工具来解压

sudo apt-get install libwww-perl libdatetime-perl #依赖的库文件
```
#### 下载脚本 ####

```
curl http://aws-cloudwatch.s3.amazonaws.com/downloads/CloudWatchMonitoringScripts-1.2.1.zip -O #下载脚本使用curl或者wget都可以
unzip CloudWatchMonitoringScripts-1.2.1.zip
rm CloudWatchMonitoringScripts-1.2.1.zip
cd aws-scripts-mon
```

CloudWatchMonitoringScripts-1.2.1.zip 程序包包含以下文件：
　　CloudWatchClient.pm – 共享 Perl 模块，以简化从其他脚本调用 Amazon CloudWatch 的过程。
　　mon-put-instance-data.pl – 收集 Amazon EC2 实例中的系统指标（内存、交换、磁盘空间利用率）并将其发送到 Amazon CloudWatch。
　　mon-get-instance-stats.pl – 查询 Amazon CloudWatch 并显示在其上执行此脚本的 EC2 实例的最近利用率统计数据。
　　awscreds.template – AWS 凭据的文件模板，储存您的访问密钥 ID 和私有访问密钥。(只有没有关联角色的时候才需要)
　　LICENSE.txt – 包含 Apache 2.0 许可证的文本文件。
　　NOTICE.txt – 版权声明。

#### 添加策略 ####

　我们在创建instance的时候会为其添加IAM role，而要运行脚本的话必须的有执行一些操作的权限:
　　　　　cloudwatch:PutMetricData
　　　　　cloudwatch:GetMetricStatistics
　　　　　cloudwatch:ListMetrics
　　　　　ec2:DescribeTags

那么我们需要为我们的role添加策略，这是我当时写的仅供参考，当然本来也不难，以json的格式来即可

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeTags"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "cloudwatch:PutMetricData"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "cloudwatch:GetMetricStatistics"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "cloudwatch:ListMetrics"
            ],
            "Resource": [
                "*"
            ]
        },
```

　若是没有添加role的话就需要修改template文件写上aws的登入id与密码了

#### 定时收集 ####

　我们只需要使用crontable就可以定时收集我们需要的数据了,而具体需要什么数据执行时需要添加什么参数可以参照官方文档

```
*/5 * * * * ~/aws-scripts-mon/mon-put-instance-data.pl --mem-util --disk-space-util --disk-path=/ --from-cron
```

