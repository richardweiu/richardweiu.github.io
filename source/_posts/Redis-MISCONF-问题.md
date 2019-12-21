---
title: Redis MISCONF 问题
date: 2019-12-18 23:13:22
tags: ["Python", "Celery", "Redis"]
categories: "Python"
---

> ** 前言 **
　redis 因为大量数据导致的保存数据库快照出错，使得 redis 的无法在执行所有的命令

## 问题

统一报警平台使用的是 django + celery 的方式，接口接收任务，任务发到 redis 中，celery worker 去任务执行，而在取消业务方的发送限制之后的一次高峰在发送任务的时候出现了这样的报错：

```python
MISCONF Redis is configured to save RDB snapshots, but is currently not able to persist on disk. Commands that may modify the data set are disabled. Please check Redis logs for details about the error
```

Redis 被配置为保存数据库快照，但它目前不能持久化到硬盘。所以用来修改集合数据的命令不能用。请查看Redis日志的详细错误信息。

遇到这种情况应急的解决方案就只有两种了：

- 老数据若是有办法恢复或者不重要，马上换一个 redis 确保服务的正常
- 使用 redis-cli 将stop-writes-on-bgsave-error设置为 no 即可（当然若是业务方也就狂轰乱炸最后只会把 redis 打爆）

```python
127.0.0.1:6379> config set stop-writes-on-bgsave-error no
```

简单来说就是那一波小高峰把 Redis 的内存撑爆，使得它没有办法将其中的数据持久化到磁盘中，而导致这样的原因有：

- 报警的方式支持三种，而每种报警在数据库中都是独立的集合存在，也就是说一条报警若是选择了三种方式，Redis 中会存三份这个报警的信息
- 报警的内容并没有截断，也就是说业务方写多大我的消息体就会有多大，而 Redis 需要存 3 倍这个消息体的大小，量一大 Redis 肯定吃不住
- 报警的接口取消掉了限流的机制，这样使得滥用的业务方很容易把整个服务给拖垮

## 参考

- https://www.jianshu.com/p/3aaf21dd34d6
