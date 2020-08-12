---
title: Nginx 处理 HTTP 请求
date: 2017-10-22 23:25:13
tags:  " Nginx "
categories: " 运维 "
---

> **前言**
　虽然利用 openrestry 加载 Lua 模块实现了动态域名的转发，但是回过头来发现自己对 Nginx 与 Lua 一点都不了解，下载来的深入理解 nginx 的书也没有非常仔细的去查阅，这里先记录一下 nginx 处理 http 请求的 11 个阶段与 Lua 执行的 8 个阶段，回头开始啃书。

## Nginx 处理 HTTP 请求

Nginx 的模块化设计使得每一个 HTTP 请求可以专注的完成一个独立的、简单的功能，而一个请求的完整处理过程由无数个 HTTP 模块共同合作完成。

HTTP 框架依据常见的处理流程将处理阶段划分成 11 个阶段：

```
typedef enum {
    // 接收到完整的HTTP头部后处理阶段
    NGX_HTTP_POST_READ_PHASE = 0,

    // 将请求URI与location表达式匹配前，修改URI，即重定向阶段
    NGX_HTTP_SERVER_REWRITE_PHASE,

    // 只能由ngx_http_core_module模块实现，用于根据请求URI寻找location表达式
    NGX_HTTP_FIND_CONFIG_PHASE,

    // 上一过程结束后修改URI
    NGX_HTTP_REWRITE_PHASE,

    // 为了防止rewrite造成死循环（一个请求执行10次会被Nginx认定为死循环）
    NGX_HTTP_POST_REWRITE_PHASE,

    // 在“决定请求访问权限”阶段前
    NGX_HTTP_PREACCESS_PHASE,

    // 决定访问权阶段，判断该请求是否可以访问Nginx服务器
    NGX_HTTP_ACCESS_PHASE,

    // 当然请求不被允许访问Nginx服务器时，该阶段负责向用户返回错误响应
    NGX_HTTP_POST_ACCESS_PHASE,

    // 用try_files配置项。顺序访问多个静态文件资源阶段
    NGX_HTTP_TRY_FILES_PHASE,

    // 处理HTTP请求内容阶段，这是大部分HTTP模块介入的阶段
    NGX_HTTP_CONTENT_PHASE,

    // 记录日志阶段
    NGX_HTTP_LOG_PHASE
} ngx_http_phases;
```

## OpenResty 处理 HTTP 请求

![lua_module](http://7xu3tw.com1.z0.glb.clouddn.com/openresty_phases.png)

- init_by_lua*: 初始化 lua
- set_by_lua*: 流程分支处理判断变量初始化
- rewrite_by_lua*: 转发、重定向、缓存等功能(例如特定请求代理到外网)
- access_by_lua*: IP 准入、接口权限等情况集中处理(例如配合 iptable 完成简单防火墙)
- content_by_lua*: 内容生成
- header_filter_by_lua*: 响应头部过滤处理(例如添加头部信息)
- body_filter_by_lua*: 响应体过滤处理(例如完成应答内容统一成大写)
- log_by_lua*: 会话完成后本地异步完成日志记录(日志可以记录在本地，还可以同步到其他机器)
（此段与图来自于[openrestry 最佳实践](https://moonbingbing.gitbooks.io/openresty-best-practices/ngx_lua/phase.html)）

## 参考

- [openrestry 最佳实践](https://moonbingbing.gitbooks.io/openresty-best-practices/ngx_lua/phase.html)）