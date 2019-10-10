---
title: python 使用 es 出现 too long frame exception
date: 2019-10-10 09:18:28
tags: [" Python ", " elasticsearch " ]
categories: " Python "
---

> ** 前言 **
　在使用 elasticsearch 库 scan 数据的时候遇到 too long frame exception 记录一下

## 问题解决

在 python3.7 的环境使用 而 elastisearch scan 过滤数据的时候触发了这样的报错：

```
elasticsearch.exceptions.RequestError: RequestError(400, 'too_long_frame_exception', 'An HTTP line is larger than 4096 bytes.')
```

[官方 issue](https://github.com/elastic/elasticsearch/issues/2137) 中表示需要修改服务端的配置，但是这个是生产环境没有办法修改与重启（当然也是没有权限），所以使用参考一文章的方法得以解决将 elasticsearch 库的版本降低至 6.3.1：

```shell
pip install elasticsearch==6.3.1
```

>我使用报错时用的版本是 7.0.1

## 参考

- https://blog.csdn.net/weixin_34177064/article/details/94717669
