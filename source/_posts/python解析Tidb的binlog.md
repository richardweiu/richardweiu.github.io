---
title: Python 解析 Tidb 的 binlog
date: 2019-10-10 09:37:31
tags: [" Python ", " Tidb ", " Protobuf "]
categories: " Python "
---

> **前言**
　数据原来是存在 mysql 中，并通过 binlog 同步至 elasticsearch 中，最近将存储切换到 Tidb 中，所以需要通过 Tidb 的 binlog 来同步数据

## python 解析 tidb 的 binlog

- 首先 tidb 的 binlog 可以直接发送到 kafka 中,所以我们只需要直接从 kafka 中获取数据
- 其次 tidb 的 binlog 数据是以 protobuf 的协议去封装的,所以需要通过 protobuf 来解析数据
- 最后 tidb 的 binlog 数据格式 table info 是个列表,change row 里面也是一个列表,只能通过顺序去做对应

### 什么是 Google Protocol Buffer

Protocol buffers 是一种语言中立，平台无关，可扩展的序列化数据的格式，可用于通信协议，数据存储等。

Protocol buffers 在序列化数据方面，它是灵活的，高效的。相比于 XML 来说，Protocol buffers 更加小巧，更加快速，更加简单。一旦定义了要处理的数据的数据结构之后，就可以利用 Protocol buffers 的代码生成工具生成相关的代码。甚至可以在无需重新部署程序的情况下更新数据结构。只需使用 Protobuf 对数据结构进行一次描述，即可利用各种不同语言或从各种不同数据流中对你的结构化数据轻松读写。

Protocol buffers 很适合做数据存储或 RPC 数据交换格式。可用于通讯协议、数据存储等领域的语言无关、平台无关、可扩展的序列化结构数据格式。

- https://halfrost.com/protobuf_encode/
- https://www.ibm.com/developerworks/cn/linux/l-cn-gpb/index.html

### 安装 protoc 

反序列话需要知道数据格式,而数据格式是通过 .proto 文件定义好的,然后通过 protoc 工具编译成想要的语言 model

Protocol Compiler 是用 C++ 写的,所以安装有两种方式:

- 下载源码编译安装
- 用编译好的 release,放在响应位置即可

我们采用第二种方式

```shell
wget https://github.com/protocolbuffers/protobuf/releases/download/v3.9.1/protoc-3.9.1-linux-x86_64.zip
unzip protoc-3.9.1-linux-x86_64.zip

# 其中 readme 中有交代如何操作

sudo cp -r include/google /usr/local/include
sudo cp bin/protoc /usr/bin/

# 验证
protoc --version

# 结果: libprotoc 3.9.1
```

### 写 tidb 的数据格式说明文件 .proto

这个数据格式可以直接从官网公开的 binlog tools 里面找到,当然在了解之后我们可以自己修改,可以参照:

- https://github.com/pingcap/tidb-binlog/blob/master/proto/binlog.proto
- https://github.com/pingcap/tidb-tools/blob/master/tidb-binlog/slave_binlog_proto/proto/binlog.proto

####  依赖处理

在编译的时候我们可能需要外部的一些依赖比如(go 的依赖前提是安装好 go,并配置有 gopath 等环境变量):

```go
import "gogo.proto";
option (gogoproto.marshaler_all) = true;
option (gogoproto.sizer_all) = true;
option (gogoproto.unmarshaler_all) = true;
```

- 首先安装依赖:

```shell
go get github.com/gogo/protobuf/gogoproto
```

- 安装依赖之后我们可以在相关的目录找到其要引入的文件:

```
$ ll ~/go/src/github.com/gogo/protobuf/gogoproto
total 80K
-rw-r--r-- 1 richardwei richardwei 8.8K 8月  19 09:45 doc.go
-rw-r--r-- 1 richardwei richardwei  33K 8月  19 09:45 gogo.pb.go
-rw-r--r-- 1 richardwei richardwei 1.3K 8月  19 09:45 gogo.pb.golden
-rw-r--r-- 1 richardwei richardwei 4.8K 8月  19 09:45 gogo.proto
-rw-r--r-- 1 richardwei richardwei  16K 8月  19 09:45 helper.go
-rw-r--r-- 1 richardwei richardwei 1.7K 8月  19 09:45 Makefile
```

注意一般在 go 的代码中我们会这样写引入的路劲,但是这样会导致生产的 python 代码依赖有问题,例如:

```
import "github.com/gogo/protobuf/gogoproto/gogo.proto";
```

会导致生产的 python 文件有这样的依赖引入,但是 python 中并没有 github 这样的依赖库:

```python
from github.com.gogo.protobuf.gogoproto import gogo_pb2 as github_dot_com_dot_gogo_dot_protobuf_dot_gogoproto_dot_gogo__pb2
```

导致这样的原因是 protoc 编译生产文件的时候导致的,只有通过 go 中的相对路径来解决这种问题

- https://github.com/protocolbuffers/protobuf/issues/4295
- https://stackoverflow.com/questions/48708496/python-protobuf-grpc-generates-a-dependency-that-doesnt-exist

所以只能这么写, 这么写之后在编译时引入库是需要注意路径问题:


```go
import "gogo.proto";
```

#### 编译依赖库

通过一个命令了解常用的参数:

```
protoc -I=. -I=$GOPATH/src/ $GOPATH/src/github.com/gogo/protobuf/gogoproto/gogo.proto -I=/usr/local/include/ /usr/local/include/google/protobuf/descriptor.proto --python_out=/home/richardwei/Code/data-transfer/protoc test.proto
```

- 通过 -I 参数告知 protoc 所需要用到的文件路径
    - `-I=.` 让 protoc 引入当前目录,这样才能找到我写的 proto
    - `-I=$GOPATH/src/github.com/gogo/protobuf/gogoproto` 让 protoc 引入依赖库地址,能够找到 gogo.proto 这个依赖文件
    - `-I=/usr/local/include` 让 protoc 引入依赖库地址,能够找到 gogo.proto 所需要的依赖文件 descriptor.proto
- 通过 --python_out 告知编译输出的文件路径
- test.proto 需要编译的 proto 文件

编译错误的话会有报错日志,一般报错不是你写错了,就是没有提供正确的依赖地址,若是却文件会有明确的提示没有找到那个文件,成功编译没有任务输出信息,但是会在你指定的目录下生产 `package_pb{version}` 的文件,例如 `test_pb2.py`

#### 使用依赖文件

编译成功之后需要在 python 中 import 即可,因为 `test_pb2.py` 可能会依赖这样的文件 `import gogo_pb2 as gogo__pb2` 不用到处找,他会生成在当前目录,提前引入即可

如何解析 binlog 数据格式:

- 从 kafka 中获取数据
- 创建 binlog 对象
- 使用 ParseFromString() 方法解析即可

#### 其他

名词说明：
- DML(data manipulation language):它们是SELECT、UPDATE、INSERT、DELETE，就象它的名字一样，这4条命令是用来对数据库里的数据进行操作的语言
- DDL(data definition language):DDL比DML要多，主要的命令有CREATE、ALTER、DROP等，DDL主要是用在定义或改变表（TABLE）的结构，数据类型，表之间的链接和约束等初始化工作上，他们大多在建立表时使用
- DCL(Data Control Language):是数据库控制功能。是用来设置或更改数据库用户或角色权限的语句，包括（grant,deny,revoke等）语句。在默认状态下，只有sysadmin,dbcreator,db_owner或db_securityadmin等人员才有权力执行DCL

## 参考

- 忘记记录了
