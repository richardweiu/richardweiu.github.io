---
title: Postgresql backup
date: 2016-06-13 09:00:03
tags: "Postgresql"
categories: "DB"
---
>** 前言 **
　　上班那会儿有一天老板让我做双机热备，我就从只听过postgresql的名字到了解它然后完成postgresql的双机热备用了半天的时间，所以并没有那么的熟悉，加上当时没有把思路理的太清楚，只是把部分记录在本子上面现在看起来还是晕晕的，还是得来梳理一遍。

### Postgresql最新消息 ###
　　2016年5月12日，PostgreSQL全球开发小组宣布了PostgreSQL 9.6第一个beta测试版提供下载。这个测试版本包含9.6最终版本中将要包含的新功能，当然，部分功能细节可能会发生改变。
　　9.6版本中包含一些重要的更新以及一些让人激动的增强功能，它们有：
　　并行顺序扫描、联合和聚合；
　　通过支持多个同步的备机和“远程申请”的同步事务提交，来实现支持强数据一致性、只读性可伸缩扩展的集群；
　　支持词组的全文检索；
　　postgres_fdw外部数据封装器现在对远程服务器可以执行排序、联合、更新和删除等指令；
　　通过避免“二次冻结”过期数据实现减少对大表的自动清理数据的影响；
　　特别地要提及的是，并行操作对所支持的查询将会带来非常大的性能提升。
　　具体可以查看[管网的wiki](https://wiki.postgresql.org/wiki/NewIn96)
### Postgresql备份方式 ###
　　和任何包含珍贵数据的东西一样，PostgreSQL数据库也应该经常备份。备份共有三种方法
　　1.SQL转储
　　2.文件系统级别备份（冷备）
　　3.PITR在线备份（Point in time Recovery）(归档，热备)
　　每种备份都有自己的优点和缺点
　　我们还有这样的一种分类方法：
　　　1.逻辑备份：SQL转储；(适用于跨版本和跨平台的恢复)
　　　2.物理备份：文件系统的备份与日志归档(适合用于跨小版本的恢复)
#### SQL转储 ####
##### 介绍 #####
　　[SQL的转储](http://www.postgres.cn/docs/9.4/backup-dump.html)很简单，我的理解的便是创建一个文件，将你现有的数据库结构以及内容都转为SQL语句存储其中，当你需要测试环境数据库时或者还原当时数据库时即可直接还原，由pg_dump创建的备份在内部是一致的，也就是说，在pg_dump运行的时候转储的是数据库的快照。pg_dump工作的时候并不阻塞其它的对数据库的操作 (但是会阻塞那些需要排它锁的操作，比如ALTER TABLE)。
>排它锁又称为写锁（(eXclusive lock,简记为X锁)）在更新操作（INSERT、UPDATE 或 DELETE）过程中始终应用排它锁。当视图修改数据时，事务会为所以来的数据资源请求排它锁，一旦授予，事务将一直持有排它锁，直至事务完成。这种锁模式之所以称为排它锁，是因为相对于相同的数据资源，如果有其他事务已经获得了该资源的任何类型的锁，就不能再获得该资源的排它锁；如果有其他事务已经获得该资源的排它锁，就不能再获得该资源的任何类型的锁。[参考](http://www.cnblogs.com/fast-michael/archive/2012/07/03/2574585.html)

##### 使用 #####
　　PostgreSQL为这个用途提供了[pg_dump](http://www.postgres.cn/docs/9.4/app-pgdump.html)工具。 这条命令的基本用法是：
```
pg_dump dbname > outfilename
#使用该命令时注意当前所在的用户
#输出可以是/路径/文件名
#linux中不以后缀识别文件,但是问了易读，让别人可看明白最好加上.sql or .dmg等等
```
　要声明pg_dump应该以哪个用户身份进行连接，使用命令参数-h host和-p port。缺省主机是本地主机或环境变量 PGPORT声明的值。类似的，缺省端口是环境变量PGPORT或 (如果它不存在的话)编译好了的缺省值。服务器通常都有相同的缺省， 所以还算方便。
　和任何其它PostgreSQL客户端应用一样，pg_dump缺省时用 与当前操作系统用户名同名的数据库用户名进行连接。要覆盖这个名字，要么声明-U选项，要么设置环境变量PGUSER。
　pg_dump超过后边描述的其它备份方法的一个重要优点 是pg_dump的输出通常可以 重新载入PostgreSQL新版本， 然而文件级别备份和连续归档都因 服务器版本而异。pg_dump是 将传输数据库到另一台机器体系结构工作时唯一的方法， 如从32位变到64位服务器。
　数据库的恢复的两个工具[psql](http://www.postgres.cn/docs/9.4/app-psql.html)与[pg_restore](http://www.postgres.cn/docs/9.4/app-pgrestore.html)基本语法
```
psql dbname < infile
#同样可以是/path/infile
#这个命令只会恢复该数据结构中的结构以及数据并不会创建数据库
#所以恢复之前记得先createdb然后再恢复

pg_restore [connection-option...] [option...] [filename]
```
>[pg_restore](http://www.postgres.cn/docs/9.4/app-pgrestore.html)用于恢复由pg_dump 转储的任何非纯文本格式中的PostgreSQL数据库。 它将发出必要的命令重建数据库，并把它恢复成转储时的样子。 归档(备份)文件还允许pg_restore有选择地进行恢复， 甚至在恢复前重新排列条目的顺序。归档的文件设计成可以在不同的硬件体系之间移植。
  pg_restore可以按照两种模式操作。如果声明了数据库名字， 那么pg_restore连接到那个数据库并直接恢复归档内容到数据库里。 否则，先创建一个包含重建数据库所必须的 SQL 命令的脚本，并且写入到一个文件或者标准输出。 这个脚本输出等效于pg_dump的纯文本输出格式。因此， 一些控制输出的选项就是模拟pg_dump的选项设置的。

##### 高级工具 #####
  pg_dump在一个时间只转储一个单独的数据库，它不转储有关角色或表空间信息（因为这些是集群范围，而不是每个数据库）。为了支持 方便转储整个数据库集群的全部内容。 因此提供了[pg_dumpall](http://www.postgres.cn/docs/9.4/app-pg-dumpall.html)程序。 pg_dumpall备份一个给出的集群中 的每个数据库，同时还确保保留像角色和表空间这样的全局数据状态。 这个命令的基本用法是：
 ```
#备份
pg_dumpall > outfile
#相应恢复
psql -f infile postgres
 ```
　pg_dumpall的工作原理是发射命令来重新创建角色， 表空间和空数据库，然后为每个数据库调用pg_dump。 这意味着，虽然每个数据库内部一致， 但不同的数据库快照是不同步的

#### 文件系统级备份 ####
##### 介绍 #####
　　直接拷贝PostgreSQL用于存放数据库数据的文件，一个数据库集群是一系列数据库的集合，用文件系统的术语来说，一个数据库集群是一个目录，所有数据都将存放在这个目录中。 我们把它称做数据目录或数据区。 在哪里存放数据完全取决于你的选择，pg并没有缺省值，一般/usr/local/pgsql/data 或/var/lib/pgsql/data这样的目录很常用。要初始化一个数据库集群， 可以使用initdb,命令
　　这个方法不常用，因为要受到两个限制，令这个方法不那么实用，或者至少比pg_dump的方法逊色一些：
　　1.为了进行有效的备份，数据库服务器必须被关闭。 像拒绝所有连接这样的折衷的方法是不行的， （部分因为tar和类似的工具在做备份的时候并不对文件系统的状态做原子快照。 但也因为服务器内部缓冲数据）. 不用说，你在恢复数据之前， 同样必须关闭服务器。
　　2.熟悉数据库在文件系统布局， 你可能试图从对应的文件或目录里备份几个表或者数据库。 这样做是没用的，因为包含在这些文件里的信息只是部分信息。 还有一半信息在提交日志文件pg_clog/*里面， 它包含所有事务的提交状态。 只有拥有这些信息，表文件的信息才是可用的。当然， 试图只恢复表和相关的pg_clog数据也是徒劳的， 因为这样会把数据库集群里的所有其它没有用的表的信息都拿出来。 所以文件系统的备份只适用于一个数据库集群的完整恢复。
##### 使用 #####
```
tar -cf backup.tar /usr/local/pgsql/data
```
#### PITR在线备份 ####
##### 介绍 #####
　　这才是这次的主角
　　在任何时候，PostgreSQL都在集群的数据目录的pg_xlog/ 子目录里维护着一套预写日志(WAL)。 这些日志记录着每一次对数据库的修改细节。 这些日志存在是为了防止崩溃： 如果系统崩溃，数据库可以通过"重放"上次检查点以来的日志记录以恢复数据库的完整性。 但是，日志的存在让它还可以用于第三种备份数据库的策略： 我们可以组合文件系统备份与WAL文件的备份。如果需要恢复，我们就恢复备份， 然后重放备份了的WAL文件，把备份恢复到当前的时间。这个方法对管理员来说， 明显比以前的方法更复杂，但是有非常明显的优势：
>1.在开始的时候我们不需要一个非常完美的一致的备份。 任何备份内部的不一致都会被日志重放动作修改正确 (这个和崩溃恢复时发生的事情没什么区别)。因此我们不需要文件系统快照的功能， 只需要tar或者类似的归档工具。
2.因为我们可以把无限长的WAL文件序列连接起来， 所以连续的备份简化为连续地对WAL文件归档来实现。 这个功能对大数据库特别有用，因为大数据库的全备份可能并不方便。
3.我们可没说重放WAL记录的时候我们必须重放到结尾。 我们可以在任意点停止重放，这样就有一个在任意时间的数据库一致的快照。 因此，这个技术支持即时恢复： 我们可以把数据库恢复到你开始备份以来的任意时刻的状态。
4.如果我们持续把WAL文件序列填充给其它装载了同样的基础备份文件的机器， 我们就有了一套热备份系统：在任何点我们都可以启动第二台机器， 而它拥有近乎当前的数据库拷贝。
　　抽象来看，一个运行着的PostgreSQL系统生成一个无限长的WAL日志序列。 系统物理上把这个序列分隔成WAL段文件，通常每段16M(在编译PostgreSQL的 时候可以改变其大小)。这些段文件的名字是数值命名的，这些数值反映他们在抽取出来的 WAL 序列中的位置。在不适用WAL归档的时候，系统通常只是创建几个段文件然 后"循环"使用它们，方法是把不再使用的段文件的名字重命名为更高的段编号。 系统假设那些内容比前一次检查点更老的段文件已经没用了，然后就可以循环利用。
　　[参考使用手册](http://www.postgres.cn/docs/9.4/continuous-archiving.html)

##### 使用 #####
```
1.首先创建主数据库服务器的归档的目录
mkdir -p /path/pg93/arch
chown -R postgres:postgres /path/pg93/arch

2.修改主数据库的配置，开启归档并设置归档命令
vim /etc/postgresql/9.3/main/postgresql.conf
listen_addresses = '*'     #默认是localhost改成*就是监听所有的连接
wal_level = hot_standby    #改成热备模式
archive_mode = on          # 归档，最好是打开
archive_command = 'DATE=`date+%Y%m%d`';DIR="/path/pg93/arch/$DATE";(test -d $DIR || mkdir -p $DIR)&& cp %p $DIR/%f    
#归档命令，就是复制wal日志的命令，这里我已每日的时间作为他们的复制目录。%p是$PGDATA的相对路径+xlog的文件名,%f则是xlog的文件名
max_wal_senders = 2        #传送日志的进程，一般几个备库几个进程就可以了
wal_keep_segments = 50     #wal日志保留数量。这个设置大一点。否则可能会出现，备库还没有接收到日志，主库已经删除，会报错。当然，9.4之后加入了slots后就解决了这个问题。


3.修改pg_hba.conf的配置文件，添加信任条目以使得备用数据库可以成功连接上来
host    replication     postgres        192.168.1.1/30        trust 
#备用数据库的ip与子网掩码，这里是乱写的
#用来备份，拉取wal日志的角色。这里我偷懒了直接用的超管。(当时我使用的是产品数据库的所属角色)

4.重新启动pg使得修改的配置文件生效
service postgresql restart

5.测试归档是否正常
sudo su postgres #s首先切换用户
psql #进入psql终端
checkpoint
select pg_switch_xlog();
\q
ll /path/pg93/arch/

6.做一次基础备份有两种方法：
方法一：使用pg_basebackup
在备用数据库中
pg_basebackup -h mydbserver_ip -D /usr/local/pgsql/data -p 5432 -U postgres
#在mydbserver创建一个服务器的基础备份，并存储到本地目录 /usr/local/pgsql/data中,或者说是本地的PGDATA的path
#使用登陆的用户我上面用的是超管所以这里也是，当然也可以创建一个专门rep用户咯

方法二：使用pg_start_backup
(1)将数据库切换至备份状态
sudo su postgres #s首先切换用户
psql #进入psql终端
select pg_start_backup('name');
#这里的name是任意你想使用的这次备份操作的唯一标识 (一个好习惯是使用备份转储文件的放置地全路径)。 
#pg_start_backup用备份信息在集群目录里 创建一个备份标签文件backup_label。 
#包含起始时间和标签字符串。该文件对于备份完整性是非常重要的，你需要从中恢复。
#至于你连接到集群中的那个数据库没什么关系。 你可以忽略函数返回的结果；但是如果它报告错误， 那么在继续之前先处理它。
#默认情况下，pg_start_backup可能需要很长的时间才能完成。 
#这是因为它会执行一个检查点，并且I/O所需的检查点会被分散在一个显著的时间段，
#默认情况下，使用一半你的相互检查点间隔。
#这是你想要的，因为它最大限度地减少对查询处理的影响。 如果你想尽快开始备份，使用：
SELECT pg_start_backup('label', true);

(2)scp或者其他工具将$PGDATA和表空间拷贝至备份服务器上
(3)结束数据库的备份状态
select pg_stop_backup();

7.修改备份数据库服务器的PG配置
vim /etc/postgresql/9.3/main/postgresql.conf
hot_standby=on

8.修改recovery.conf配置，默认并不存在，可以将模板拷过来
cp /usr/share/postgresql/9.3/recovery.conf.sample /etc/postgresql/9.3/main/recovery.conf
vim /etc/postgresql/9.3/main/recovery.conf

standby_mode=on
primary_conninfo='host=ip post=5432 user=postgres'
restore_command='cp $PG_ARCHIVE/%f %p'
trigger_file = '/path/trigger_file'      用来激发备库转换成主库的标识文件。
recovery_target_timeline = 'latest'

```
##### 注意 #####
　　在9.4的版本中还存在一些问题
　　在线备份技术还有几个局限。它们可能在将来的版本中修补：
　　1.在Hash索引上的操作目前没有使用WAL记录日志，所以重放就不会更新这些索引类型。 这将意味着任何新的插入被索引忽略，已更新行显然会消失，并且已删除行将仍然保留指针。 换句话说，如果你修改了带有hash索引的表，那么你将获得备用服务器上不正确的查询结果。 当完成恢复时，建议是在完成恢复操作之后手工REINDEX每个这样的索引。
　　2.如果在进行数据库备份的时候发出一个CREATE DATABASE命令， 然后在这个过程中CREATE DATABASE命令拷贝的模板数据库被修改了， 那么用这个备份进行恢复的数据库很有可能导致这些修改也传播到新创建的数据库中去。 这个行为当然是不愿意看到的。为了避免这个风险， 最好在进行数据库备份的时候不要修改任何模板数据库。
　　3.CREATE TABLESPACE命令是用文本的绝对路径记录WAL日志的， 因此会以相同的绝对路径重新创建。如果日志是在另外一台机器上重放， 那么这个行为可能不是我们想要的。即使在同一台机器， 但是在一个新的数据目录里重放日志，都很可能是危险的： 重放仍将会覆盖原来的表空间的内容。 为了避免这类的潜在问题， 最好的方法是在创建或者删除表空间之后进行一次新的基础备份。
　　4.还要注意，缺省的WAL格式体积相当大，因为它包含许多磁盘页快照。 这些磁盘页快照是设计来支持崩溃恢复的， 因为我们可能需要修补部分写入的磁盘页。根据你的系统硬件和软件的不同， 这种部分写入的危险可能是微乎其微的。这种情况下， 你可以通过使用full_page_writes关闭磁盘页面快照， 从而大大减少归档日志的总尺寸(在你这么做之前，阅读第 29 章里面的注意和警告)。 关闭页面快照并不阻止日志使用PITR操作。 一个将来需要开发的功能是在full_page_writes打开的时候， 通过删除不需要的磁盘页拷贝来压缩归档的 WAL 数据。同时， 管理员可以通过尽量合理地增加检查点的时间间隔来减少包含在WAL里的页面快照。


　　数据库的热备原理其实还是没有理解的很透彻，很多概念还没有搞清楚。。还会继续深入