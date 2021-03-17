---
title: Nginx-proxy-pass-坑
date: 2020-11-17 17:02:22
tags: " Nginx "
categories: " 运维 "
---

> **前言**
　在帮同事解决需求的时候无意间踩了一个小坑记录一下

## 需求

同事希望统一域名,让用户在请求的 url 中带上 AZ,然后由七层的负载均衡 Nginx 来做匹配与路由到各自所在 AZ 的域名上, 例如:

- 原请求: http://bj.ptc.xxx.com/images/xxxx
- 改造后: http://ptc.xxxx.com/hb-az-1/images/xxxx (由 nginx 路由 http://bj.ptc.xxx.com/images/xxxx)

## 解决

所以写出了这样的一个配置:

```bash
server {
    listen 7777 default_server;
    location ~* /hb-az-1/ {
        proxy_pass http://localhost:999/;
        # return 200 $uri;
    }

}

server {
    listen 999 default_server;
    location / {
        default_type text/html;
        return 200 $uri;
    }
}

```

- `proxy_pass` 在反向代理根路径的时候若是不带 '/' 会带上原有的 uri, 这样不是我们想要的,所以此处带上了 `/`
- 第一反应没有考虑开头精确匹配所以使用的忽略大小写的正则匹配

然后这样 `nginx -t` 的时候就报错了:

```
nginx: [emerg] "proxy_pass" cannot have URI part in location given by regular expression, or inside named location, or inside "if" statement, or inside "limit_except" block in /etc/nginx/conf.d/test.conf:126
nginx: configuration file /etc/nginx/nginx.conf test failed
```

所以修正的方式就是把 `~*` 改成 `^~` 就行了, 若是必须使用正则的方式,那就在 `proxy_pass` 之前增加一个 `rewrite`

## 总结

- Nginx 的 location 中使用 `~*` 与 `~` 这样的正则方式是内层的 `proxy_pass` 的 url 是不能带 `/` 的
- 分享一个工具: [Nginx location 解析树](https://detailyang.github.io/nginx-location-match-visible/)

## 参考

- [nginx location regex error](https://serverfault.com/questions/649151/nginx-location-regex-doesnt-work-with-proxy-pass/649196#649196)


