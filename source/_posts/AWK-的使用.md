---
title: AWK 的使用（楼赛答案）
date: 2017-10-13 22:06:14
tags: ["awk"," Linux "]
categories: " 运维 "
---

> ** 前言 **
　之前举办了一次 Linux 相关的楼赛，已经过去三个月了，一直想写博客来总结与公布一下我的答案。幸好一直有 everyday note 的习惯，保留当时的记录。这次的楼赛真的是把我 awk 的老底都拿出来。

## 第一个挑战

### 题目

第一个挑战的题目是：提取文本中链接信息

这个题目的来源真的是工作中了，在打包课程时需要获取 markdown 文档中的所有的图片连接，然后将所有的图片都现在下来，并且替换所有的连接成相对路径。

而挑战中演变成了提取所有的课程连接连接，并且以 `课程名 link` 的格式展示出来。

### 思路

其实比较简单，在文档中课程连接这样的方式展现：

```
[课程](https://www.shiyanlou.com/courses/)
```

那我们需要做的就应该是：

- 通过 grep 把所有带 `[` 的给我找出来
- 通过 grep -v 把所有的图片连接都给我剔除，毕竟两种各种相差不打
- 通过 awk 把课程名与连接都剥离出来，然后以特定的方式展现

### 答案

从上面的思路我们很容易就得出答案，一行命令就搞定了：

```bash
cat shiyanlou_lab1.md | grep '\[' | grep -v '^!' | awk -F '[][)(]' '{print $2, $3 $4}'
```

这里需要注意的就是以 `[`、`]`、`(`、`)` 来分割文本的时候不要这样顺序配对的写，需要交错来写，否则会直接闭合，使用默认的分隔符来分割。

当然本来的题目要求灵活一点写成脚本，需要提取文本的文件名是参数，所以脚本就应该是这样了：

```bash
#!/bin/bash

filename=$1

cat $1 | grep '\[' | grep -v '^!' | awk -F '[][)(]' '{print $2, $3 $4}'
```

## 第二个挑战

### 题目

第二个挑战的题目是：统计网络数据包

这个应该是不是来自于生活中了，虽然平时我已有抓包但是没有统计数据包的需求，而且人家 tcpdump 也显示了抓了多少数据包。这个题目麻烦的地方在于定时，只抓去在规定时间内的数据包，当然着实是难到我了。

### 思路

之前的思路比较局限，就像通过 tcpdump 这个工具本身来完成，google 了一圈，查到的方式用的是用 `-G` 与 `-w` 参数，通过 `-w` 参数可以将抓到的数据写到文件中，而 `-G` 参数是每隔多少秒就轮询一次，但是这并不是我想要的结果。

我想要的是 3 秒钟中之后结束 tcpdump 这个命令。

然后我发现了一个新的命令叫做 `timeout`

这个工具真是神器，它的介绍就是 `run a command with a time limit`，有了它就完美解决了：

### 答案

```bash
#!/bin/bash

port=$1

sudo timeout 3 tcpdump -i eth0 port $port -l 2>&1 |tee test.txt >/dev/null 2>&1
cat test.txt |grep captured | awk '{print "Packages:",$1}'
```

当然这里我是用的 tee 来将输出的内容都写到一个文件中，然后在查看该文件获取 tcpdump 输出的统计，然后以要求的格式输出。因为自己去统计行数来计算包的数据来的非常不准确，所以还是用 tcpdump 给的结论比较靠谱。

在实际使用的时候若是以管道符与 tee 来将抓包的内容写到一个文件中并不好，我测试的感觉是会丢包较为的严重，还是用 `-w` 来写入文件比较靠谱，而且这样抓包的内容还可以使用 wireshark 来分析，比较方便一些。

## 第三个挑战

### 题目

第三个挑战的题目是：收集指定时间的文件

虽然在我的工作场景中并没有，但是感觉这样的场景在某些特定情况下还是很有用的。

要求是将实验楼实验环境中的 /etc 目录下的所有最后更新时间在2015年的文件拷贝到 /tmp 目录，需要保持目录结构。将实验楼实验环境中的 /etc 目录下的所有最后更新时间在2015年的文件拷贝到 /tmp 目录，需要保持目录结构。

### 思路

其实解决问题的思路是首先将所有符合要求的文件都找到，使用 find 来过滤出所有的时间范围的文件。

第二步我觉的就是一个难点，要保持文件的原目录结构复制到目标目录，复制只能将文件复制过去却不能保持原目录，唯一的想法就是获取到目录结构，然后创建相关目录，然后将相关文件复制过去。

关键的问题在于如何识别目录？

目录是用 `/` 符号一级一级划分，可以用 awk 来分割，分割了之后既要获取到目录保存在变量中，因为不仅需要创建目录还需要将文件相应的位置中去。

awk 的划分之后若是只有 3 个变量（在 `/` 之前有一个空格，然后是 `etc`,然后就是文件）说明没有子目录，直接就是文件，这样的话就可以直接 cp 过去，若是变量数目大于 3 的话就说明有子目录，获取最后一个变量之前的所有内容即为我们所需要的目录，这样我们就可以创建相关的子目录，然后在 cp 文件。这里必须在 awk 中调用 cp 命令，因为相关的变量在 awk 之外无法使用。

### 答案

通过上述的分析，就得出了这样的脚本：

```bash
#!/bin/bash

sudo find /etc/* -type f \( -newermt '2015-01-01 00:00' -a -not -newermt '2015-12-31 23:59' \)
if [! -d "/tmp/ect"];then

    sudo mkdir -p /tmp/ect;
fi

sudo find /etc/* -type f \( -newermt '2015-01-01 00:00' -a -not -newermt '2015-12-31 23:59' \) | while read line
do

    echo $line | awk -F '/' \
    '{
        path="/tmp/etc";
        if (NF==3){
        system("sudo cp " $0 " " "/tmp"$0)
        } else {
            for(i=3; i < NF; i++ ) {
                path=path"/"$i;
            }
        system("sudo mkdir -p " path)
        system("sudo cp " $0 " " path"/"$NF)
        }
    }'

done
```

通过 find 获取到相关时间段的文件，然后读取每一行数据，紧接着就通过 awk 来划分了，在 awk 中有变量、有判断、又循环、有系统调用完美的胜任了所有的需求，我也是第一次这样使用 awk，十分的强大。之前没有这样的需求，就算有我可能会选择 python 来实现。

## AWK 的回顾([摘录哈宝狗博客](http://www.habadog.com/2011/05/22/awk-freshman-handbook/))

### 语法

```bash
awk 'pattern + {action}'
```

说明：

- 单引号”是为了和shell命令区分开；
- 大括号 `{}` 表示一个命令分组；
- pattern 是一个过滤器，表示命中 pattern 的行才进行 action 处理；
- action是处理动作；
- pattern和action可以只有其一，但不能两者都没有,默认的 action 是 print；

其中的内置变量有[tutorialspoint](https://www.tutorialspoint.com/awk/awk_built_in_variables.htm)：

变量 | 说明 
----|-----
$0	| 当前记录，也就是所有内容（作为单个变量）
\$1~$n |	当前记录的第n个字段，字段间由 FS 分隔
FS	| 字段分隔符 默认是空格
NF	| 当前记录中的字段个数，就是有多少列
NR	| 已经读出的记录数，就是行号，从1开始
RS	| It represents (input) record separator and its default value is newline.
OFS	| It represents the output field separator and its default value is space.
ORS	| It represents the output record separator and its default value is newline.
ARGC |	命令行参数个数
ARGV |	命令行参数数组形式展示
FILENAME |	当前输入文件的名字
IGNORECASE |	如果为真，则进行忽略大小写的匹配
ENVIRON	| 环境变量
ERRNO	| 系统错误消息
FIELDWIDTHS	| 输入字段宽度的空白分隔字符串
FNR	| It is similar to NR, but relative to the current file. It is useful when AWK is operating on multiple files. Value of FNR resets with new file.

内置函数有：

函数 | 说明 
----|-----
gsub(r,s) | 在 $0 中用 s 代替 r
index(s,t) | 返回 s 中 t 的第一个位置
length(s) | s 的长度
match(s,r) | s是否匹配r

几点注意用法：

- awk 与 shell 直接传递变量

awk 中使用 shell 中定义的变量：使用单引号即可；

```bash
awk '{print "'${PATH}'"}'
```

shell 中使用 awk 的变量是没办法了

- awk 中使用 shell 命令

1. 使用系统调用 `system()`
2. 使用双引号扩起来 `awk '{print $0 | "cat"}'`

- 系统调用需要多个参数的时候

在 awk 脚本中，如果需要调用 shell 脚本/命令，则需要使用 `system()` 函数，如果需要将变量传递给被调用的 shell，则写为 `system("sh my.sh " $var)` (注意第二个引号前有一个空格)

若是需要多个参数的时候，参数直接的空格需要用引号扩起来(来自于 [stackoverflow](https://stackoverflow.com/questions/15823216/awk-system-call
)，例如：

```bash
awk '{system("mv -R " $1 " " $2)}' file.cfg
```

## 参考

- [tutorialspoint](https://www.tutorialspoint.com/awk/awk_built_in_variables.htm)
- [哈巴狗](http://www.habadog.com/2011/05/22/awk-freshman-handbook/)
- [stackoverflow](https://stackoverflow.com/questions/15823216/awk-system-call
)
