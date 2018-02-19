---
title: ELK 实现标签排名
date: 2017-10-22 19:59:03
tags: " ELK "
categories: " 运维 "
---

> ** 前言 **
　前段时间实现了线上课程与课程标签的排名，由此记录一下 ELK 的使用，免得又因为长时间不用而忘记。先简单记录后面在继续深入了解，学习。
- 2017/10/29 更新：内容填充

## Logstash 简介

Logstash 是世界著名的运维工程师乔丹西塞 (JordanSissel) 用 JRuby 语言编写的一个数据收集、分析的工具（这是我暂时的认识）。

## Logstash 配置

Logstash 配置了自己 DSL(domain-specific language，针对特定应用领域而设计使用的计算机语言，比如 awk、puppet 等，与之相对的 GPL(general-purpose language) 指的是针对跨应用领域而设计使用的计算机语言，比如 C、Java)，通过 DSL 配置主要指明数据的从哪里来，最后向哪里去，有时候还会说明一下输出前会做怎样的一个处理，这样的一个过程称之为 logstash 的 pipeline，而实现这一切的主要依靠这样的三个插件：

- input：数据输入
- filter：数据过滤
- output：数据输出

![logstash-pipeline](http://7xu3tw.com1.z0.glb.clouddn.com/logstash-pipeline.png)
(此图来自于 [IBM elk 栈博客](https://www.ibm.com/developerworks/cn/opensource/os-cn-elk/index.html))


所以 Logstash 的配置框架是这样的：

```
input {
    stdin/redis/collectd/file/syslog 等等
}
filter {
    date/grok/geoip 等等
}
output {
    stdout/file/elasticsearch/redis 等等
}
```

### Logstash 数据源配置

数据的来源可以有很多种，比如：

- stdin：以标准输入作为数据源
- file： 以文件的内容作为数据源（FileWatch 的 Ruby Gem 库来监听文件变化，会记录文件读取的位置）
- tcp： 以读取网络数据作为数据源
- syslog： 以 syslog 作为数据源
- redis： 以 redis 作为数据源

还有很多很多，具体可以查看[官方文档](https://www.elastic.co/guide/en/logstash/current/input-plugins.html)。

举个例子，比如 stdin 标准输入

logstash 通过标准输入来获取数据，也就是 logstash 会等待你输入，你输入的内容作为其分析的来源，例如：

```bash
logstash -e 'input{stdin{}}output{stdout{codec=>rubydebug}
}'
```

通过这样的一个命令，logstash 会等他你输入，输入的内容会以特定的格式输出。

在平时的使用中，主要通过 redis 作为 Broker。所以这里介绍一些使用 redis 作为数据来源该如何配置：

> 这里补充一下，logstash 的生态系统中主要的组成部分:
> - Shipper：数据的收集、发送，一般使用 logstash（太重） 或者 Filebeat（轻量级）;
> - Broker：数据的收集，数据的缓冲区，一般使用 redis 或者 kafka; 
> - Indexer：索引器，数据的索引化，主要是数据的分析、处理，一般使用 logstash;
> - Search and Storage：数据的搜索与存储，一般使用 elasticsearch;
> - Web Interface:数据的页面展示，一般使用 kibana;

通过一个例子来做了解：

```ruby
input {
  redis {
    host => "localhost"
    type => "nginx"
    data_type => "list"
    key => "logstash:nginx"
    db => "1"
  }
}
```

- host：配置 redis 所在的机器，默认值为 `127.0.0.1`；
- type：配置事件类型，就像是给数据归类、打标签一样，方便于 filter 中使用；
- data_type：配置 redis 消息传递的方式，必须配置项，可以设置的值有：list，channel，pattern_channel 这些值分别对应 redis 这样一些方式：BLPOP，SUBSCRIBE，PSUBSCRIBE
- key：对数据的命名
- db：设置 redis 的哪一个数据库，默认值是 0。

更多的配置项与详细的解说可以查看其[官方文档](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-redis.html#plugins-inputs-redis-data_type)。

### Logstash 过滤器

Filter 可以说是 Logstash 这么强大的原因之一了，可以对数据进行复杂的逻辑处理，对数据增加、拆分或者转换出需要的事件。

在 filter 中使用的插件非常的多，不仅可以使用别人写好的插件还可以使用自己编写的插件，插件非常的多这里无法一一列举，同样我们通过一个例子来做相关的介绍。

在此之前我们列举几个常用的：

- grok：正则捕获，可以在其中预定义好命名正则表达式，在后期引用。
- date：时间处理，对时间戳的格式转化
- mutate：数据修改，对数据的类型转化，字符串的切割、字段的处理等
- geoip：IP 地址的查询归类
- json：json 数据的处理

举个例子：

```ruby
filter{
    if [type] == "nginx" {
        grok {
            match => [ "message" , "%{COMBINEDAPACHELOG} %{GREEDYDATA:extra_fields}"]
            overwrite => [ "message" ]
        }

        mutate {
            convert => ["response", "integer"]
        }

        geoip {
          source => "clientip"
          target => "geoip"
          add_tag => [ "nginx-ip" ]
        }

        date {
          match => [ "timestamp" , "dd/MMM/YYYY:HH:mm:ss Z" ]
          remove_field => [ "timestamp" ]
        }       
    }
    if [type] == "statis" {
        json {
            source => "message"
            target => "jsoncontent"
        }

        mutate {
            split => ["[jsoncontent][things_Tag]", ","]
        }
    }
}
```

在这个例子中我们将数据分为两种类型的处理方式，通过数据的 type 来区分：

- nginx
- statis

这个 type 是我们在 input 插件获取数据是对数据加的一个标签，用于对数据的一个归类，方便我们在后期对不同的数据做做不同的处理。

1.grok

首先使用到的插件就是 grok 正则捕获，grok 配置项有很多，可以查看[官方文档](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html#plugins-filters-grok-match)，这里我们用到的有 match、overwrite。

通过 hash 值来匹配我们需要处理的字段 message，后面用双引号扩主我们的正则表达式，当然官方也给我们提供了许多的正则表达式模版，这样我们就不用再幸幸苦苦的去写正则表达式，[官方的正则模版](https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns)，从系统日志到 DNS 日志到 Java 日志各种个样都有。

在 httpd 中我们可以看到 apache 的日志正则模版 COMMONAPACHELOG，但是标准的 nginx 是与 apache 不同的，所以一般 nginx 标准日志的正则表达式应该是这样的：

```bash
%{COMBINEDAPACHELOG} %{QS:x_forwarded_for}
```

在 grok-patterns 中我们可以看到 `GREEDYDATA .*`，这个模版就是传说中的懒人模式，用于匹配一行的全部内容。

grok 表达式的打印赋值格式的完整语法是下面这样的:

```bash
 %{PATTERN_NAME:capture_name:data_type}
 #模版名：捕获的字段名：数据类型（只支持 int 与 float）
```

当然还可以自定义模版：

```bash
(?<field_name>the pattern here)
```

若是需要多行匹配需要在前面加上 `(?m)`。

overwrite 的作用是重写 message 字段，若是没有这个配置项的话，grok 之后把所有的内容切分在不同的字段中，而 message 中也有相应的内容，这样会导致数据存有两份。

这就是上面 grok 的使用相关的内容。

grok 在性能上面还是有所欠缺，所以在 5.0 之后支持新的字段 dissect。

2.mutate

mutate 是一个专门负责数据的切分、转换、拼接等的一些功能的插件。常用的配置项有：

- convert：对数据类型的转换，如 integer, float, string, and boolean；
- join：对数组数据按特定的分离器进行拼接；
- lowercase/uppercase：对 string 数据进行大小写的切换；
- split: 对数据安特定的分离器进行切割，切割成数组；
- rename：对某个字段进行重命名；

更多的配置项查看[官方文档](https://www.elastic.co/guide/en/logstash/current/plugins-filters-mutate.html#plugins-filters-mutate-merge)

所以在上面的例子中，我们使用了 convert 对数据类型的一个转换，使用了 split 对列表的一个切割。

因为希望做对事物标签的一个排名，后台传过来的数据是一个列表，不对这个列表进行处理的话，Kibana 在画排名图的时候就会做一个整体去匹配，所以需要将其切割，这样才能做标签个体的一个单独排名。

3.geoip

geoip 是一个免费的 IP 查询库，通过它可以获取一个 IP 的地域信息，包括国别，省市，经纬度等，这个在希望查看用户的分布的时候非常有用了。

4.date

date 是对一个时间戳格式进行处理的插件。

5.json

json 是对 json 数据的格式化处理。

### Logstash 数据输出

对数据的格式、内容处理完毕之后便需要将数据存放到一个地方，一般是输出至 elasticsearch，当然在输出至 elasticsearch 之前我们希望看到我们处理后的数据是否是正确的，做一下 debug，我们可能会选择输出到屏幕上亦或者是文件中。

举一例子：

```ruby
output {
  elasticsearch {
    hosts => ["localhost:9200"] 
    index => "logstash-%{+YYYY.MM.dd}"
  }
  stdout {
    codec => "rubydebug"
  }
  file {
    codec => "rubydebug"
    path => "/var/log/logstash/debug"
  }
}
```

stdout 为标准输出，所以该插件就是将所有的数据输出至屏幕上。其中 codec 的 rubydebug 会把数据格式化输出。

## Kibana 配置

Kibana 上都是一些界面的操作，搜索的方式可以使用直接搜索也可使用 api 的方式搜索。可以话的图的类型也很多，方式也很多，逐一讲解的话内容太多了。

在标签排名中主要是选择了 Vertical bar chart 图，Y 轴使用 Count，X 轴使用 Term。

![show-tag-count](http://7xu3tw.com1.z0.glb.clouddn.com/show-tag-count.png)

当然还有一些其他的统计数据，集合在 dashboard：

![show-dashboard](http://7xu3tw.com1.z0.glb.clouddn.com/show-dashboard.png)

感觉自己现在的需求几乎都是没有东西都摸一下但是都没有深入的去了解学习，什么都知道一点，但是什么都不精。这样并不好啊。就像这 ELK，搞定了之后就再也没有碰过了，几乎都忘的差不多了，若是深入学习的话还有什么搜索的优化、插件的开发等等

## 参考

- [ELK Stack权威指南](https://github.com/chenryn/logstash-best-practice-cn)