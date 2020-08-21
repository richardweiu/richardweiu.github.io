---
title: Nginx 调试与自定义 header
date: 2020-08-20 10:00:29
tags: " Nginx "
categories: " 运维 "
---

> **前言**
　记录之前提供一个简单代理的小工具时所用的调试方法与自定义 header 的使用注意

## 需求

- 请求参数中带访问地址，在 Nginx 中增加 header(header 中带秘钥不对外透露)，并访问参数中的地址
- 前端增加自定义的 header 用作判断访问的跳转条件

### Nginx 的 Debug

主要是需要对参数中的内容做正则获取、判断等操作，在我们不确定我们截取、判断等操作是否正确的时候可以通过以下两种方式做 Debug:

- 通过指定 txt 为返回类型 `default_type text/html;` 与 return 相关的字符串即可
- 通过 echo 模块与 `default_type text/plain` 方式也可以做到获取相关的字符串达到 Debug 的目的

```bash
# 通过 map 模块来获取 get 参数中 url 参数里面的文件名
map $arg_url $file_name {
    ~/(?<captured_request_basename>[^/?]*)(?:\?|$) $captured_request_basename;
}

server {
    listen       80;
    server_name  xxxx;
    access_log /var/log/nginx/test/access.log main;
    error_log /var/log/nginx/test/error.log;
    charset utf-8;
    # proxy_pass 的地址若是线上的域名或是公网域名必须带 resolver 否则 nginx 会报解析不了
    resolver 10.100.100.100;

    location /proxy {
        # 设置返回的类型为 text 的形式
        default_type text/html;

        # 通过正则获取参数中的 url 的 uri 部分
        if ($arg_url ~*  ^(.*)http://data.xxxx.xxx.com(.*)$){
            set $target $2;
            # 返回状态码与相关的 txt 内容，这样 response 内容就会是 $2 的 txt content
            return 200 $2;
            # 指定下载文件名否则文件名为初始 uri
            add_header Content-Disposition "attachment; filename=$file_name";
            # 带上鉴权的 header
            add_header x-CDN-Agent "xxx-xxx";
            proxy_pass http://xxx.xxx.domain$2;
        }
    }
}
```

需注意的事项:

- 以上功能没有使用的 rewrite 的方式实现是因为 rewrite 通过 301 或 302 转发是不会带请求中的 header 的
- 若是 rewrite 的方式可以通过 `rewrite_log on` 的方式打开 Debug 日志

### Nginx 自定义 Header

线上有时候会通过自定义的 Header 来作为判断依据从而反向代理到不同的地址，例如：

```bash
server{
    server_name xxx.domain;
    listen 80 default_server;

    # 若是 header 中存在下划线必须打开该配置
    underscores_in_headers on;
    access_log /var/log/nginx/msg.access.log;
    error_log /var/log/nginx/msg.error.log;

    location ~* .(agg|v1|admin) {
        #proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;

        # 判断名为 search_env 的 header 值
        if ($http_search_env = "mainland") {
            proxy_pass http://report.xxx.domain;
        }
        if ($http_search_env = "international") {
            proxy_pass http://report-inter.xxx.com;
        }
        # 若是以上都不匹配的默认转发地址
        proxy_pass http://report.xxx.domain;

    }
}
```

需注意的事项:

- 当增加自定义的 Header 时且 Header 中带下划线时，需要打开 `underscores_in_headers` 的配置项，否则 nginx 不识别且不转发
- 自定义的 Header 可以通过 `http_变量名` 的方式访问

## 总结

- rewrite 跳转是不能传递原来请求与自己增加的 Header
- 通过 `$arg_变量名` 可以获取 GET 中的参数值
- 自定义的 Header 且 Header 中带下划线时，需要打开 `underscores_in_headers` 配置项
- proxy_pass Debug 的方式有两种：
  - 通过 `default_type text/html;` 与 return 相关的字符串
  - 通过 echo 模块与 `default_type text/plain` 方式
- proxy_pass 的需要注意的细节
  - proxy_pass url 中根路径带 '/' 反向代理的时候会忽略 uri 
    - proxy_pass http://localhost/
    - 'http://localhost/proxy/index.html' --> 'http://localhost/index.html'
  - proxy_pass url 中根路径未带 '/' 反向代理的时候会加上原有的 uri
    - proxy_pass http://localhost;
    - 'http://localhost/proxy/index.html' --> 'http://localhost/proxy/index.html'
  - proxy_pass url 中非根路径带 '/' 反向代理的时候会
    - proxy_pass http://localhost/richard/;
    - 'http://localhost/proxy/index.html' --> 'http://localhost/richard/index.html'
  - proxy_pass url 中非根路径未带 '/' 反向代理的时候会
    - proxy_pass http://localhost/richard;
    - 'http://localhost/proxy/index.html' --> 'http://localhost/richardindex.html'

## 参考

- [map 匹配文件名](https://serverfault.com/questions/381545/how-to-extract-only-the-file-name-from-the-request-uri)
- [map 使用](http://www.ttlsa.com/nginx/using-nginx-map-method/)
