# HBase 2.2.5



![hbase-logo](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/hbase-logo.png)

![image-20221122101813121](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/image-20221122101813121.png)

![image-20221122101935145](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/image-20221122101935145.png)





## HBase介绍

BigTable：[入口链接](https://blog.csdn.net/accesine960/article/details/595628)



### HBase简介

Hadoop Database，是一个高可靠性、高性能、面向列、可伸缩、实时读写的分布式数据库。

利用Hadoop HDFS作为其文件存储系统，利用Hadoop MapReduce来处理HBase中的海量数据，利用Zookeeper作为其分布式协同服务主要用来存储非结构化和半结构化的松散数据（列存 NoSQL 数据库)

![image-20221122104029507](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/image-20221122104029507.png)



### HBase优点

- 容量大：
  Hbase单表可以有百亿行、百万列，数据矩阵横向和纵向两个维度所支持的数据量级都非常具有弹性
- 面向列：
  面向列的存储和权限控制，并支持独立检索，可以动态增加列，即，可单独对列进行各方面的操作
  列式存储，其数据在表中是按照某列存储的，这样在查询只需要少数几个字段的时候，能大大减少读取的数量
- 多版本：
  Hbase的每一个列的数据存储有多个Version，比如住址列，可能有多个变更，所以该列可以有多个version
- 稀疏性：
  为空的列并不占用存储空间，表可以设计的非常稀疏。
  不必像关系型数据库那样需要预先知道所有列名然后再进行null填充
- 拓展性：
  底层依赖HDFS，当磁盘空间不足的时候，只需要动态增加datanode节点服务(机器)就可以了
- 高可靠性：
  WAL机制，保证数据写入的时候不会因为集群异常而导致写入数据丢失
  Replication机制，保证了在集群出现严重的问题时候，数据不会发生丢失或者损坏Hbase底层使用HDFS，本身也有备份。
- 高性能：
  底层的LSM数据结构和RowKey有序排列等架构上的独特设计，使得Hbase写入性能非常高。
  Region切分、主键索引、缓存机制使得Hbase在海量数据下具备一定的随机读取性能，该性能针对Rowkey的查询能够到达毫秒级别，LSM树，树形结构，最末端的子节点是以内存的方式进行存储的，内存中的小树会flush到磁盘中（当子节点达到一定阈值以后，会放到磁盘中，且存入的过程会进行实时merge成一个主节点，然后磁盘中的树定期会做merge操作，合并成一棵大树，以优化读性能）
  LSM树的介绍：[入口链接]( https://www.cnblogs.com/yanghuahui/p/3483754.html)

下图显示了列族在面向列的数据库：

![image-20221122105050340](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/image-20221122105050340.png)



### HBase应用

Hbase是一种NoSQL数据库，这意味着它不像传统的RDBMS数据库那样支持SQL作为查询语言。Hbase是一种分布式存储的数据库，技术上来讲，它更像是分布式存储而不是分布式数据库，它缺少很多RDBMS系统的特性，比如列类型，辅助索引，触发器，和高级查询语言等待。那Hbase有什么特性呢？如下：

什么时候用Hbase？Hbase不适合解决所有的问题：

- 首先数据库量要足够多，如果有十亿及百亿行数据，那么Hbase是一个很好的选项，如果只有几百万行甚至不到的数据量，RDBMS是一个很好的选择。因为数据量小的话，真正能工作的机器量少，剩余的机器都处于空闲的状态
- 其次，如果你不需要辅助索引，静态类型的列，事务等特性，一个已经用RDBMS的系统想要切换到Hbase，则需要重新设计系统。
- 最后，保证硬件资源足够，每个HDFS集群在少于5个节点的时候，都不能表现的很好。因为HDFS默认的复制数量是3，再加上一个NameNode。
- Hbase在单机环境也能运行，但是请在开发环境的时候使用。

内部应用

- 存储业务数据：车辆GPs信息，司机点位信息，用户操作信息，设备访问信息。。。
- 存储日志数据：架构监控数据（登录日志，中间件访问日志，推送日志，短信邮件发送记录。。。)，业务操作日志信息
- 存储业务附件：UDFS系统存储图像，视频，文档等附件信息

不过在公司使用的时候，一般不使用原生的Hbase API，使用原生的API会导致访问不可监控，影响系统稳定性，以至于版本升级的不可控。



### 数据库分类

简单来说，关系模型指的就是二维表格模型，而一个关系型数据库就是由二维表及其之间的联系所组成的一个数据组织。

**关系性数据库**

- 关系型数据库的3大优点：

  - 容易理解、使用方便、易于维护

- 关系型数据库的3大瓶颈：

  - 高并发读写需求：网站的用户并发性非常高，往往达到每秒上万次读写请求，对于传统关系型数据库来说，**硬盘I/O是一个很大的瓶颈，并且很难能做到数据的强一致性。**
  - 海量数据的读写性能低：网站每天产生的数据量是巨大的，对于关系型数据库来说，在一张包含海量数据的表中查询，效率是非常低的。
  - 扩展性和可用性差：在基于web的结构当中，数据库是最难（但是可以）进行横向扩展的，当一个应用系统的用户量和访问量与日俱增的时候，数据库却没有办法像 web server 和 app server 那样简单的通过添加更多的硬件和服务节点来扩展性能和负载能力。对于很多需要提供24小时不间断服务的网站来说，对数据库系统进行升级和扩展是非常痛苦的事情，往往需要停机维护和数据迁移。

  PS：当然，对网站来说，关系型数据库的很多特性不再需要了，比如：事务一致性、读写实时性。在关系型数据库中，导致性能欠佳的最主要原因是多表的关联查询，以及复杂的数据分析类型的复杂sQL报表查询。为了保证数据库的ACID特性，我们必须尽量按照其要求的范式进行设计，关系型数据库中的表都是存储一个格式化的数据结构。

非关系型数据库提出另一种理念，**例如**：以keyvalue键值对存储，且结构不固定，每一个元组可以有不一样的字段，每个元组可以根据需要增加一些自己的键值对，这样就不会局限于固定的结构，可以减少一些时间和空间的开销。使用这种方式，用户可以根据需要去添加自己需要的字段，这样，为了获取用户的不同信息，不需要像关系型数据库中，要对多表进行关联查询。仅需要根据id取出相应的value就可以完成查询。

**非关系型数据库特点：**

- 一般不支持ACID特性，无需经过SQL解析，读写性能高
- 存储格式：key value，文档，图片等等
- 数据没有耦合性，容易扩展

​			**但**非关系型数据库由于很少的约束，他也不能够提供像SQL所提供的where这种对于字段属性值情况的查询。并且难以体现设计的完整性。他只适合存储一些较为简单的数据，对于需要进行较复杂查询的数据，SQL数据库显的更为合适

总结一下：

![image-20221122113215056](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/image-20221122113215056.png)

由于非关系型数据库本身天然的多样性，以及出现的时间较短，因此，不像关系型数据库，有几种数据库能够一统江山，非关系型数据库非常多，并且大部分都是开源的。这些数据库中，其实实现大部分都比较简单，除了一些共性外，很大一部分都是针对某些特定的应用需求出现的，因此，对于该类应用，具有极高的性能。依据结构化方法以及应用场合的不同，主要分为以下几类：

- 面向高性能并发读写的key-value数据库：Redis，Tokyo Cabinet，Flare
- 面向海量数据访问的面向文档数据库：MongoDB以及CouchDB，可以在海量的数据中快速的查询数据
- 面向可扩展性的分布式数据库：这类数据库想解决的问题就是传统数据库存在可扩展性上的缺陷，这类数据库可以适应数据量的增加以及数据结构的变化

**简单的理解：**

- 关系型数据库
  - 通过表与表的字段管理描述对象与对象的关系
  - 这种表都是基于行存储的
    - (插入数据的时候，即使没有数据也要插入null)，获取数据的时候，即使只获取一个列，也要先查出一行，我们获取数据只需要两个维度即可（id,column）
- 非关系型数据库
  - 半结构化--更加灵活
    - user1={uname:''zs'',age:18}
  - 非结构化
    - 图片，视频

**Hbase和RDBMS**

| 属性     | Hbase                | RDBMS                  |
| -------- | -------------------- | ---------------------- |
| 数据类型 | 只有字符串           | 丰富的数据类型         |
| 数据操作 | 增删改查，不支持join | 各种各样的函数与表连接 |
| 存储模式 | 基于列式存储         | 基于表结构和行式存储   |
| 数据保护 | 更新后仍然保留旧版本 | 替换                   |
| 可伸缩性 | 轻易增加节点         | 需要中间层，牺牲性能   |





## HBase数据模型

数据类型：int，char。。。。。。；HBase不存在数据类型，里面存储的是字节

数据模型：行，列							；HBase不仅有行列，HBase总共有：RK，CF，TS，Q，cell

![image-20221122115530897](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/image-20221122115530897.png)

在Hbase中，有一些术语需要提前了解。如下：

### NameSpace：

- 命名空间是类似于关系数据库系统中的数据库的概念，他其实是表的逻辑分组。这种抽象为多租户相关功能奠定了基础。
- 命名空间是可以管理维护的，可以创建，删除或更改命名空间。
- HBase有两个特殊预定义的命名空间：
  - default - 没有明确指定名称空间的表将自动落入此名称空间
  - hbase - 系统命名空间，用于包含HBase内部表



### Table：

- Hbase的table由多个行组成

- | Row Key | Time Stamp | Column Family1 | Column Family2 | Column Family3 |
  | ------- | ---------- | -------------- | -------------- | -------------- |
  | 111     | t6         |                | CF2:q1=val1    | CF3:q3=val3    |
  | 112     | t3         | CF1:q2=val3    |                |                |
  |         | t2         |                | CF2:q8=val2    |                |



### RowKey：

- RowKey 是用来检索记录的主键，是一行数据的唯一标识
- RowKey行键（RowKey）可以是任意字符串(最大长度是64KB，实际应用中长度一般为10 - 100 bytes)，RowKey以字节数组保存。
- 存储时，数据按照RowKey的字典序(byte order)排序存储。设计RowKey时，要充分排序存储这个特性，将经常—起读取的行存储放到一起。



### Column Family：

- 列簇在物理上包含了许多的列与列的值，每个列簇都有一些存储的属性可配置。
  - 例如是否使用缓存，压缩类型，存储版本数等。在表中，每一行都有相同的列簇，尽管有些列簇什么东西也没有存。
- 将功能属性相近的列放在同一个列族，而且同一个列族中的列会存放在同一个store中
- 列族一般需要在创建表的时候就进行声明，而且一般一个表中的列族数不要超过3个
  - 这个和后期的优化相关
- 列隶属于列族，列族隶属于表



### Column Qualifier：

- 列簇的限定词，理解为列的唯一标识。但是列标识是可以改变的，因此每一行可能有不同的列标识
- 使用的时候必须  列族：列
- 列可以根据需求动态添加或者删除，同一个表中不同行的数据列都可以不同



### Cell：

- Cell是由row，column family，column qualifier，version 组成的
- Cell中的数据是没有类型的，全部是字节码形式存储
  - 因为HDFS上的数据都是字节数组



### Timestamp：

- HBASE 中通过 rowkey和column family，column qualifier 确定的一个存储单元称为cell。每个cell都保存着同一份数据的多个版本。
- 版本通过时间戳来索引。
  - 时间戳的类型是 64 位整型
    - 默认时间戳是精确到毫秒的当前系统时间。时间戳也可以由客户显式赋值。
    - 如果应用程序要避免数据版本冲突，就必须自己生成具有唯一性的时间戳。
- 每个cell中，不同版本的数据按照时间倒序排序，即最新的数据排在最前面。
  - 查询数据的时候，如果不指定版本数，默认显示版本号最新（高）的数据
- 为了避免数据存在过多版本中造成管理(包括存贮和索引)负担，HBASE提供了两种数据版本回收方式。
  - 一是保存数据的最后n个版本
  - 二是保存最近一段时间内的版本（比如最近七天）。

![image-20221123030259129](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/image-20221123030259129.png)

HBase是一个稀疏的、分布式、持久、多维、排序的映射，它以行键（row key），列键(column key ---> cf:q) 和 时间戳(timestamp) 为索引。

Hbase在存储数据的时候，有两个SortedMap，首先按照rowkey进行字典排序，然后再对Column进行字典排序。

![4553977-b15b97aa91e88231](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/4553977-b15b97aa91e88231.jpg)





## HBase架构模型

![20180821125818844](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/20180821125818844.png)

HBase有三个主要组成部分：客户端库，主服务器和区域服务器。区域服务器可以按要求添加或删除

架构细解

![QQ截图20221123095611](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/QQ%E6%88%AA%E5%9B%BE20221123095611.png)



### Client

- 客户端负责发送请求到数据库
- 客户端连接的方式有很多种
  - hbase shell
  - 类JDBC
- 发送的请求主要包括
  - DDL：数据库定义语言(表的建立，删除，添加删除列族，控制版本)
  - DML：数据库操作语言(增删改)
  - DQL：数据库查询语言(查询--全表扫描--基于主键--基于过滤器)
- client维护着一些cache来加快对hbase的访问，比如regione的位置信息。



### zookeeper

- 保证任何时候，集群中只有一个master
- 存贮所有Region的寻址入口，存储所有的的元数据信息。
- 实时监控Region Server的状态，将Region server的上线和下线信息实时通知给Master
- 存储Hbase的schema，包括有哪些table，每个table有哪些column family



### HMaster

- HBase集群的主节点，HMaster也可以实现高可用(active--standby)
  - 通过zookeeper来维护主副节点的切换
- 为Region server分配region
- 负责region server的负载均衡
- 管理用户对table的结构创建，删除，修改（ddl）操作
  - 表的元数据信息--》Zookeeper上面
  - 表的数据--》HRegionServer上
- 当HRegionServer下线的时候，HMaster会将当前HRegionServer上的Region转移到其他的HRegionServer



### HRegionServer

- Region server属于HBase具体数据的管理者
- Region server维护Master分配给它的region，处理对这些region的IO请求
-  
- 会实时的和HMaster保持心跳，汇报当前节点的信息
- 当接收到Hmaster命令创建表的时候，分配一个Region对应一张表
- Region server负责切分在运行过程中变得过大的region
-  
- 当客户端发送DML和DQL操作的时候，HRegionServer负责和客户端建立连接
- 当意外关闭的时候，当前节点的Region会被其他HRegionServer管理

![image-20221124011244464](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/image-20221124011244464.png)



### HRegion

- HRegion是HBase中分布式存储和负载均衡的最小单元。最小单元就表示不同的HRegion可以分布在不同的HRegion server上。

- HBase自动把表水平划分成多个区域(region)，每个region会保存一个表里面某段连续的数据

  ![image-20221124012058364](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/image-20221124012058364.png)

- 每个表一开始只有一个region，一个Region只属于一张表，随着数据不断插入表，region不断增大，当增大到一个阀值的时候，region就会等分会两个新的region（裂变）

  - hbase.hregion.max.filesize
  
    ![image-20221124012559933](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/image-20221124012559933.png)
  
- 当table中的行不断增多，就会有越来越多的region。这样一张完整的表被保存在多个Regionserver上。

- 一张表后期有可能有多个Region

  - 因为随着时间的推移，Region会越来越大

  - Region达到阈值10G的时候会被平分（逻辑上平分,尽量保证数据的完整性）

  - 会将切分后的其中一个Region转移到其他的HRegionServer上管理

    ![image-20221124012944348](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/image-20221124012944348.png)

- 为了防止前期数据的处理都集中在一个HRegionServer，我们可以根据自己的业务进行预分区

  ![image-20221124013120033](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/image-20221124013120033.png)



### Store

- HRegion是表获取和分布的基本元素，由一个或者多个Store组成，每个store保存一个columns family。

- 每个Store又由1个memStore和0或多个StoreFile组成。

  - ```
    Table	(HBase table)
    	Region	(Regions for the table)
    		Store	(Store per ColumnFamily for each Region for the table)
    			MemStore	(MemStore for each Store for each Region for the table)
    			StoreFile	(StoreFiles for each Store for each Region for the table)
    			Block		(Blocks within a StoreFile within a Store for each Region for the table)
    ```

  - ![Store](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/476922-589b0fd4f19dc2d3.png)
  
- HFile是Hbase在HDFS中存储数据的格式，它包含多层的索引，这样在Hbase检索数据的时候就不用完全的加载整个文件。

- 索引的大小(keys的大小，数据量的大小)影响block的大小，在大数据集的情况下，block的大小设置为每个RegionServer 1GB也是常见的。

> 探讨数据库的数据存储方式，其实就是探讨数据如何在磁盘上进行有效的组织。因为我们通常以如何高效读取和消费数据为目的，而不是数据存储本身。

![img](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/u=935881185,2900341726&fm=253&app=138&f=PNG&fmt=auto&q=75)

![img](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/u=1923426000,815629917&fm=253&app=138&f=PNG&fmt=auto&q=75)

#### Storefile

- Data Blocks 段-保存表中的数据，这部分可以被压缩
- Meta Blocks 段(可选的)-保存用户自定义的kv对，可以被压缩。
- File lnfo段-Hfile的元信息，不被压缩，用户也可以在这一部分添加自己的元信息。
- Data Block Inex段-Data Block的索引。每条索引的key是被索引的block的第一条记录的key。
- Meta Block Index段（可选的）- Meta Block的索引。
- Trailer - 这一段是定长的。
  - 保存了每一段的偏移量，读取一个HFile时，会首先读取Trailer
  - Trailer保存了每个段的起始位置(段的Magic Number用来做安全check)
  - 然后，DataBlock Index会被读取到内存中，这样，当检索某个key时，不需要扫描整个HFile，而只需从内存中找到key所在的block
  - 通过一次磁盘io将整个block读取到内存中，再找到需要的key。DataBlock Index采用LRU机制淘汰。

#### MemStore

- HFile中并没有任何Block，数据首先存在于MemStore中。Flush发送时，创建HFile Writer

- 当操作数据的时候，第一个空的Data Block初始化，初始化后的Data Block中为Header部分预留了空间。

- Header部分用来存放一个Data Block的元数据信息。

- 位于MemStore中的KeyValues被一个个append到位于内存中的第一个Data Block中：

  - 注：如果配置了Data Block Encoding，则会在Append KeyValue的时候进行同步编码，编码后的数据不再是单纯的KeyValue模式。

  - Data Block Encoding是HBase为了降低KeyValue结构性膨胀而提供的内部编码机制。

    ![img](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/u=2319799596,83476706&fm=253&app=138&f=PNG&fmt=auto&q=75)



### Hlog

- Write After Log做任何操作之前先写日志

- HLog文件就是一个普通的Hadoop Sequence File
  - SequeceFile的Key是HLogKey对象，
    - HLogKey中记录了写入数据的归属信息，除了table和region名字外，同时还包括sequence number和timestamp
      - timestamp是 “ 写入时间 ”，
      - sequence number的起始值为0，或者是最近一次存入文件系统中sequence number。
  - SequenceFile的Value是HBase的KeyValue对象，即对应HFile中的KeyValue。
  
- 日志直接存放到HDFS上，写入的数据就是
  - 当前的日志序列号（Sequence）
  - 本次的操作
  
- 当memStore达到阈值的时候开始写出文件之后，会在日志中对应的位置标识一个检查点

- 一个HRegionServer只有一个Log文档

- MasterProcWAL：HMaster记录管理操作，比如解决冲突的服务器，表创建和其它DDLs等操作到它的WAL文件中，这个WALs存储在MasterProcWALs目录下，它不像RegionServer的WALs，HMaster的WAL也支持弹性操作，就是如果Master服务器挂了，其它的Master接管的时候继续操作这个文件。

- WAL记录所有的Hbase数据改变，如果一个RegionServer在MemStore进行Flush的时候挂掉了，WAL可以保证数据的改变被应用到。如果写WAL失败了，那么修改数据的完整操作就是失败的。

  - 通常情况，每个RegionServer只有一个WAL实例。在2.0之前，WAL的实现叫做HLog
  - WAL位于 */hbase/WALs/* 目录下
  - MultiWAL：如果每个RegionServer只有一个WAL，由于HDFS必须是连续的，导致必须写WAL连续的，然后出现性能问题。MultiWAL可以让RegionServer同时写多个WAL并行的，通过HDFS底层的多管道，最终提升总的吞吐量，但是不会提升单个Region的吞吐量。

- WAL的配置：

  ```jsx
  // 启用multiwal
  <property>
      <name>hbase.wal.provider</name>
      <value>multiwal</value>
  </property>
  ```

  ![20140405161209421](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/20140405161209421.png)





## HBase集群搭建

### 高可用搭建

|        | HMasterActive | HMasterStandBy | HRegionServer | Zookeeper |
| ------ | ------------- | -------------- | ------------- | --------- |
| node01 | *             |                | *             | *         |
| node02 |               | *              | *             | *         |
| node03 |               |                | *             | *         |

前提：Hadoop集群正常运行 和 Zookeeper集群正常运行，时间同步

#### 准备安装环境

```shell
# [root@node01 ~]#
tar -zxvf hbase-2.2.5-bin.tar.gz
mv hbase-2.2.5 /opt/
cd /opt/hbase-2.2.5/conf/
```

#### 修改集群环境

- [root@node01 conf]# vim hbase-env.sh

- ```sh
  export HBASE_LOG_DIR=${HBASE_HOME}/logs
  export JAVA_HOME=/usr/java/jdk1.8.0_231-amd64
  export HBASE_MANAGES_ZK=false
  export HADOOP_HOME=/opt/hadoop-3.1.2/
  ```

#### 修改配置环境

- [root@node01 conf]# vim hbase-site.xml

- ```xml
  <!-- 31dd -->
  <!--设置HBase表数据，也就是真正的HBase数据在hdfs上的存储根目录-->
  <property>
      <name>hbase.rootdir</name>
      <value>hdfs://bdp/hbase</value>
  </property>
  <!--是否为分布式模式部署，true表示分布式部署-->
  <property>
      <name>hbase.cluster.distributed</name>
      <value>true</value>
  </property>
  <!--zookeeper集群的URL配置，多个host中间用逗号-->
  <property>
      <name>hbase.zookeeper.quorum</name>
      <value>Node1:2181,Node2:2181,Node3:2181</value>
  </property>
  <!--HBase在zookeeper上数据的根目录znode节点-->
  <property>
      <name>zookeeper.znode.parent</name>
      <value>/hbase</value>
  </property>
  <!-- 本地文件系统tmp目录，一般配置成local模式的设置一下，但是最好还是需要设置一下，因为很多文件都会默认设置成它下面的-->
  <property>
      <name>hbase.tmp.dir</name>
      <value>/var/hbase</value>
  </property>
  <!--使用本地文件系统设置为false，使用hdfs设置为true -->
  <property>
      <name>hbase.unsafe.stream.capability.enforce</name>
      <value>false</value>
  </property>
  ```

- vim regionservers

- ```
  Node1
  Node2
  Node3
  ```

#### 备用Master节点

- [root@Node1 conf]# vim backup-masters

- ```
  Node2
  ```

#### 拷贝Hadoop配置文件

- [root@Node1 conf]# cp /opt/hadoop-3.1.2/etc/hadoop/core-site.xml /opt/hbase-2.2.5/conf/
- [root@Node1 conf]# cp /opt/hadoop-3.1.2/etc/hadoop/hdfs-site.xml /opt/hbase-2.2.5/conf/

#### 拷贝分发软件

- [root@Node2 ~]# scp -r root@Node1:/opt/hbase-2.2.5 /opt/
- [root@Node3 ~]# scp -r root@Node1:/opt/hbase-2.2.5 /opt/

#### 修改环境变量

- [root@Node1 conf]# vim /etc/profile

- ```sh
  export HBASE_HOME=/opt/hbase-2.2.5
  export PATH=$HBASE_HOME/bin:$PATH
  ```

- 拷贝到其他节点
  - [root@Node1 conf]# scp /etc/profile root@Node2:/etc/profile
  - [root@Node1 conf]# scp /etc/profile root@Node3:/etc/profile
- 让配置文件生效
  - 【123】 source /etc/profile

#### 启动集群

- 【123】 zkServer.sh start
- [root@Node1 conf]# start-all.sh
- [root@Node1 conf]# start-hbase.sh

#### web界面

- 可以看到服务器1启动和HMaster 和HRegionServer进程，服务器2和服务器3启动和HRegionServer进程。

- hbase集群安装和启动完成，此时可以通过web页面查看Hbase集群情况：http://Node1:16010

  ![image-20221130040730057](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/image-20221130040730057.png)

  ![image-20221130040806774](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/image-20221130040806774.png)





## HBase使用方式

### HBase shell

我们可以先用Hbase提供的命令行工具，位于hbase的/bin/目录下

进入退出

```shell
hbase shell

exit
```

查看帮助信息

```shell
help
```



#### 通用操作

1. 查询服务器状态

   ```shell
   status
   ```

2. 查询hbase版本

   ```shell
   version
   ```

3. 如果有kerberos认证，需要事先使用相应的keytab进行一下认证（使用kinit命令)，认证成功之后再使用hbase shell进入可以使用whoami命令可查看当前用户

   ```shell
   whoami
   ```
   
   

#### DDL操作

1. 创建一个表

   语法：create <table>,{NAME =><family>,VERSIONS =><VERSIONS>}

   例如：创建表t1，有两个family name：f1，f2，且版本数均为2

   ```sql
   create 't1',{NAME => 'f1',VERSIONS =>2},{NANE => 'f2',VERSIONS =>2}
   ```

   还有一种非标准创建的语法，创建表member，列族是member_id，address，info，版本为1。如下：

   ```sql
   create 'member' , 'member_id' , 'address' , 'info'
   ```

2. 获得表的描述

   语法：describe <table>

   ```sql
   list
   describe 'member'
   ```

3. 删除一个列族，alter，disable，enable

   我们之前建了3个列族，但是发现member_id这个列族是多余的，因为它就是主键，所以我们要将其删除。

   ```sql
   alter 'member',{NAME=>'member_id',METHOD=>'delete'}
   ```

   将表enable

   ```sql
   enable 'member'
   ```

4. 列出所有的表

   ```sql
   list
   ```

5. drop 一个表

   ```sql
   create 'temp_table','member_id','address','info'
   
   disable 'temp_table'
   
   drop 'temp_table'
   ```

6. 查询表是否存在

   ```sql
   exists 'member'
   ```

7. 判断表是否enable

   ```sql
   is_enabled 'member'
   ```

8. 判断表是否disable

   ```sql
   is_disabled 'member'
   ```

9. 查看文件存储路径

   ```sql
   hdfs dfs -ls /hbase/data/default/t1
   ```

10. truncate 此命令将删除并重新创建一个表

    ```sql
    truncate 't1'
    ```

11. create_namespace 创建名称空间；传递名称空间名称，并可选地传递名称空间配置的字典。

    ```sql
    create_namespace 'ns1'
    ```

12. list_namespace 列出hbase中的所有命名空间，可选的正则表达式参数可用于筛选输出。

    ```sql
    list_namespace
    ```

13. describe_namespace  描述命名空间

    ```sql
    describe_namespace 'ns1'
    ```

14. drop_namespace  删除命名空间

    ```sql
    drop_namespace 'ns1'
    ```

15. list_namespace_tables  列出属于命名空间成员的所有表

    ```sql
    list_namespace_tables 'default'
    ```



#### DML操作

1. 插入几条数据

   ```shell
   put 'member','guojing','info:age','24'
   put 'member','guojing','info:birthday','1987-06-17'
   put 'member','guojing','info:company','alibaba'
   put 'member','guojing','address:contry','china'
   put 'member','guojing','address:province','zhejiang'
   put 'member','guojing','address:city','hangzhou'
   
   
   put 'member','yangkang','info:birthday','1987-4-17'
   put 'member','yangkang','info:favorite','movie'
   put 'member','yangkang','info:company','alibaba'
   put 'member','yangkang','address:contry','china'
   put 'member','yangkang','address:province','guangdong '
   put 'member','yangkang','address:city','jieyang'
   put 'member','yangkang','address:town','xianqiao'
   ```

   > 注：现在表的数据都在内存中，并没有落地到磁盘。如果这时候想要落地到磁盘只能手动落地
   >
   > 命令：flush 'tablename'
   >
   > 例如：flush 'member'

2. 获取一条数据

   获取一个id的所有数据

   ```sql
   get 'member','scutshuxue'
   ```

   获取一个id，一个列族的所有数据

   ```sql
   get 'member','scutshuxue','info'
   ```

   获取一个id，一个列族中一个列的所有数据

   ```sql
   get 'member','scutshuxue','info:age'
   ```

3. 更新一条记录

   将scutshuxue的年龄改成99

   ```sql
   put 'member','scutshuxue','info:age','99'
   
   get 'member','scutshuxue','info:age'
   ```

4. 通过timestamp来获取两个版本的数据

   ```shell
   get 'member','scutshuxue',{COLUMN=>'info:age',TIMESTAMP=>1321586238965}
   
   get 'member','scutshuxue',{COLUMN=>'info:age',TIMESTAMP=>1321586571843}
   ```

5. 全表扫描

   ```sql
   scan 'member'
   ```

   > 注：全表扫描的时候一般都和过滤器一起使用
   >
   > 例如：
   >
   > ```sql
   > --扫描时指定列族
   > scan 'member',{COLUMNS => 'info'}
   > 
   > --(3) 扫描时指定列族，并限定显示最新的5个版本的内容
   > scan 'member',{COLUMNS => 'info',VERSIONS =>5}
   > 
   > --(4) 设置开启Raw模式，开启Raw模式会把那些已添加删除标记但是未实际删除的数据也显示出来
   > scan 'member',{COLUMNS => 'info',RAW => true}
   > 
   > --(5) 列的过滤
   > # 查询user表中列族为info和data的信息
   > scan 'member',{COLUMNS => ['info','address']}
   > 
   > # 查询user表中列族为info，列名为name，列族为data，列名为pic的信息
   > scan 'member',{COLUMNS => ['info:name','address:pic']}
   > 
   > # 查询user表中列族为info，列名为name的信息，并且版本最新的5个
   > scan 'member',{COLUMNS => 'info:name',VERSIONS =>5}
   > 
   > # 查询user表中列族为info和data且列名含有a字符的信息
   > scan 'member',{COLUMNS =>['info','data'],FILTER =>"(QualifierFilter(=,'substring:a'))"}
   > 
   > # 查询user表中列族为info，rk范围是[rk0001,rk0003)的数据
   > scan 'member',{COLUMNS =>'info',STARTROW =>'00001',ENDROW =>'00003'}
   > 
   > # 查询user表中row key以rk字符开头的
   > scan 'member',{FILTER=>"PrefixFilter('0')"}
   > 
   > # 查询user表中指定时间范围的数据
   > scan 'member',{TIMERANGE => [1392368783980,1392380169184]}
   > ```

6. 删除id为temp的值的 'info:age' 字段

   ```sql
   delete 'member','temp','info:age'
   ```

7. 删除整行

   ```sql
   deleteall 'member','xiaofeng'
   ```

8. 查询表中有多少行

   ```sql
   count 'member'
   ```

9. 给 ‘xiaofeng’ 这个id增加 ‘info:age’ 字段，并使用counter实现递增

   ```sql
   incr 'member','xiaofeng','info:age'
   
   get 'member','xiaofeng','info:age'
   
   incr 'member','xiaofeng','info:age'
   
   get 'member','xiaofeng','info:age'
   ```

   获取当前count的值

   ```sql
   get_counter 'member','xiaofeng','info:age'
   ```

10. 将整张表清空

    ```sql
    truncate 'member'
    ```

11. 可以使用count命令计算表的行数量

    ```sql
    count 'member'
    ```



#### Region管理

1）移动region

语法：move ‘encodeRegionName’,‘ServerName’

encodeRegionName指的regionName后面的编码，ServerName指的是master-status的Region Server列表

示例：

```sql
move '4343995a58be8e5bbc739af1e91cd72d','db-41.xx×.Xxx.org,60820,1398274516739'
```

2）开启/关闭region

HBase是一种支持自动负载均衡的分布式Kv数据库，在开启balance的开关(balance_switch)后，HBase的HMaster进程会自动根据指定策略挑选出一些Region，并将这些Region分配给负载比较低的RegionServer上。

语法：balance_switch true|false

```sql
balance_switch true
```

3）手动split

语法：split ‘regionName’,'splitKey'

```sql
split '8ea39ae715882c7469dcac4981c47b43','001'
```

4）手动触发major compaction

语法：压缩表中的所有区域：

```sql
major_compact 't1'
```

压缩整个区域：

```sql
major_compact 'r1'
```

在region中压缩单列族：

```sql
major_compact 'r1','c1'
```

在表中压缩单列族：

```sql
major_compact 't1','c1'
```

测试数据

```sql
create 'user','info','ship'

put 'user','524382618264914241', 'info:name','zhangsan '
put 'user','524382618264914241', 'info:age',30
put 'user','524382618264914241', 'info:height',168
put 'user','524382618264914241', 'info:weight',168
put 'user','524382618264914241', 'info:phone','13212321424'
put 'user','524382618264914241', 'ship:addr','beijing'
put 'user','524382618264914241', 'ship:email','sina@sina.com '
put 'user','524382618264914241', 'ship:salary',3000

put 'user', '224382618261914241', 'info:name','lisi'
put 'user', '224382618261914241', 'info:age',24
put 'user', '224382618261914241', 'info:height',158
put 'user', '224382618261914241', 'info:weight',128
put 'user', '224382618261914241', 'info:phone','13213921424'
put 'user', '224382618261914241', 'ship:addr','chengdu'
put 'user', '224382618261914241', 'ship:email','qq@sina.com'
put 'user', '224382618261914241', 'ship:salary',5000

put 'user', '673782618261019142', 'info:name','zhaoliu'
put 'user', '673782618261019142', 'info:age',19
put 'user', '673782618261019142', 'info:height',178
put 'user', '673782618261019142', 'info:weight',188
put 'user', '673782618261019142', 'info:phone','17713921424'
put 'user', '673782618261019142', 'ship:addr','shenzhen'
put 'user', '673782618261019142', 'ship:email','126@sina.com '
put 'user', '673782618261019142', 'ship:salary',8000

put 'user', '813782218261011172', 'info:name','wangmazi'
put 'user', '813782218261011172', 'info:age',19
put 'user', '813782218261011172', 'info:height',158
put 'user', '813782218261011172', 'info:weight',118
put 'user', '813782218261011172', 'info:phone','12713921424'
put 'user', '813782218261011172', 'ship:addr','xian'
put 'user', '813782218261011172', 'ship:email','139@sina.com'
put 'user', '813782218261011172', 'ship:salary',10000

put 'user', '510824118261011172', 'info:name','yangyang'
put 'user', '518824118261011172', 'info:age',18
put 'user', '510824118261011172', 'info:height',188
put 'user', '510824118261011172', 'info:weight',138
put 'user', '510824118261011172', 'info:phone','18013921626'
put 'user', '510824118261011172', 'ship:addr','shanghai'
put 'user', '510824118261011172', 'ship:email','199@sina.com '
put 'user', '510824118261011172', 'ship:salary',50000
```



#### 通用命令

- status：提供HBase的状态，例如，服务器的数量。
- version：提供正在使用HBase版本。
- table_help：表引用命令提供帮助。
- whoami：提供有关用户的信息。



#### 数据操纵语言

- put：把指定列在指定的行中单元格的值放在一个特定的表。
- get：取行或单元格的内容。
- delete：删除表中的单元格值。
- deleteall：删除给定行的所有单元格。
- scan：扫描并返回表数据。
- count：计数并返回表中的行的数目。
- truncate：禁用，删除和重新创建一个指定的表。



#### 数据定义语言

- create：创建一个表

- list：列出HBase的所有表

- disable：禁用表

  要删除表或改变其设置，首先需要使用disable命令关闭表。使用enable命令，可以重新启用它。

  语法：

  ```sql
  disable 'user'
  ```

  例子：

  ```sql
  hbase(main):025:0> disable 'user'
  0 row(s) in 1.2760 seconds
  ```

  验证：禁用表之后，仍然可以通过list和exists命令查看到。无法扫描到它存在，它会给下面的错误。

  ```sql
  hbase(main):028:0> scan 'user '
  ROw COLUMN+CELL
  
  ERROR: user is disabled .
  ```

- disable_all

  此命令用于禁用所有匹配给定正则表达式的表。disable_all命令的语法如下：

  ```sh
  hbase> disable_all 'r.*'
  ```

  假设有5个表在HBase，即user, student, teacher, test和t1。下面的代码将禁用所有以t开始的表。

  ```sql
  hbase(main):002:0> disable_all 't,*'
  
  teacher
  test
  t1
  Disable the above 3 tables (y/n)?
  
  y
  
  3 tables successfully disabled
  ```

- is_disabled：验证表是否被禁用。

  这个命令是用来查看表是否被禁用。它的语法如下。

  ```sql
  hbase> is_disabled 'table name'
  ```

  下面的例子验证表名为user是否被禁用。如果禁用，它会返回true，如果没有，它会返回false。

  ```sql
  hbase(main):031:0> is_disabled 'user'
  
  true
  
  0 row(s) in 0.0440 seconds
  ```

- enable：启用一个表

  语法：

  ```sql
  enable 'user'
  ```

  例子：

  ```sql
  hbase(main):005:0> enable 'user'
  0 row(s) in 0.4580 seconds
  ```

  验证：启用表之后，扫描。如果能看到的模式，那么证明表已成功启用。

  ```sql
  hbase(main):052∶0> scan 'user'
  ROW									cOLUMN+CELL
  224382618261914241					column=info:age, timestamp=1593484524928, value=24
  224382618261914241					column=info:height, timestamp=1593484524954， value=158
  224382618261914241					column=info:name, timestamp=1593484524854, value=lisi
  224382618261914241					column=info:phone, timestamp=1593484525076, value=13213921424
  224382618261914241					column=info:weight, timestamp=1593484525836，value=128
  224382618261914241					column=ship:addr, timestamp=1593484525118, value=chengdu
  224382618261914241					column=ship:email, timestamp=1593484525181,value=qq@sina.com
  224382618261914241					column=ship:salary, timestamp=1593484525215, value=5000
  510824118261011172					column=info:age,timestamp=1593484525918, value=18
  ```

- is_enabled：验证表是否已启用。

  此命令用于查找表是否被启用。它的语法如下：

  ```sql
  hbase> is_enabled 'table name'
  ```

  下面的代码验证表user是否启用。如果启用，它将返回true，如果没有，它会返回false。

  ```sql
  hbase(main):031∶0> is_enabled 'user'
  true
  0 row(s) in 0.0440 seconds
  ```

- describe：提供了一个表的描述。

  语法：

  ```sql
  hbase> describe 'table name'
  ```

  下面给出的是对user表的describe命令的输出。

  ```sql
  hbase(main):053:0> describe 'user'
  Table user is ENABLED
  user
  cOLUMN FAMILIES DESCRIPTION
  {NAME => 'info'，BLOOMFILTER => 'ROW',VERSIONS => '1'，IN_MEMORY => 'false ',KEEP_DELETED_CELLS =>'FALSE', DATA_BLOCK_ENCODING =>'NONE'，TTL =>‘FOREVER',COMPRESSION =>‘NONE'，MIN_VERSIONS => '0'，BLOCKCACHE =>'true',BLOCKSIZE =>'65536',REPLICATION_SCOPE => '0'}
  {NAME => 'ship',BLOOMFILTER => 'ROw',VERSIONS => '1'，IN_MEMORY => 'false '，KEEP_DELETED_CELLS =>'FALSE ', DATA_BLOCK_ENCODING =>‘NONE'，TTL =>‘FOREVER'，COMPRESSION =>'NONE'，MIN_VERSIONS => '0'，BLOCKCACHE =>'true',BLOCKSIZE =>'65536',REPLICATION_SCOPE => '0'}
  2 row(s) in 0.0940 seconds
  ```

- alter：改变一个表

  alter是用于更改现有表的命令。使用此命令可以更改列族的单元，设定最大数量和删除表范围运算符，并从表中删除列族。

  1）更改列族单元格的最大数目

  语法：

  ```sql
  hbase> alter 't1',NAME => 'f1', VERSIONS =>5
  ```

  例子：单元的最大数目设置为5。

  ```sql
  hbase(main ):003:0> alter 'user ', NAME => 'info', VERSIONS => 5
  Updating all regions with the new schema . . .
  0/1 regions updated.
  1 /1 regions updated.
  Done.
  0 row(s) in 2.3050 seconds
  ```

  2）表范围运算符

  使用alter，可以设置和删除表范围，运算符，如MAX_FILESIZE，READONLY，MEMSTORE_FLUSHSIZE,DEFERRED_LOG_FLUSH等。

  设置只读：

  ```sql
  hbase>alter 't1',READONLY(option)
  ```

  在下面的例子中，我们已经设置表user为只读。
  
  ```sql
  hbase(main):086:0> alter 'user', READONLY
  Updating all regions with the new schema . . .
  0/1 regions updated .
  1/1 regions updated .
  Done .
  0 row(s) in 2.2140 seconds
  ```
  
  3）删除表范围运算符
  
  也可以删除表范围运算。下面给出的是语法，从user表中删除“MAX_FILESIZE”。
  
  ```sql
  hbase> alter 't1', METHOD => 'table_att_unset',NAME =>'MAX_FILESIZE'
  ```
  
  4）删除列族
  
  语法：
  
  ```sql
  alter 'table name','delete' => 'column family'
  ```
  
  例子：从 “user” 表中删除指定的 info 列族。
  
  ```sql
  hbase(main):007∶0> alter 'user' , 'delete' => 'info'
  Updating all regions with the new schema . . .
  0/1 regions updated .
  1 /1 regions updated.
  Done.
  0 row(s) in 2.2380 seconds
  ```
  
  验证：观察更改后的表数据，发现列族“info”也没有了，因为前面已经被删除了。
  
  ```sql
  hbase(main):061:0> scan 'user'
  ROW									COLUMN+CELL
  224382618261914241					column=ship:addr,timestamp=1593484525118, value=chengdu
  224382618261914241					column=ship:email,timestamp=1593484525181,value=qq@sina.com
  
  224382618261914241					column=ship:salary, timestamp=1593484525215,value=5000
  510824118261011172					column=ship:addr,timestamp=1593484526071，value=shanghai
  510824118261011172					column=ship:email, timestamp=1593484526119,value=199@sina.com
  
  510824118261011172					column=ship:salary, timestamp=1593484526774, value=50000
  524382618264914241					column=ship:addr， timestamp=1593484524712, value=beijing
  524382618264914241					column=ship:email,timestamp=1593484524764,value=sina@sina.com
  
  524382618264914241					column=ship:salary,timestamp=1593484524799, value=3000
  673782618261019142					column=ship:addr，timestamp=1593484525475, value=shenzhen
  673782618261019142					column=ship:email, timestamp=1593484525510,value=126@sina.com
  
  673782618261019142					column=ship:salary, timestamp=1593484525536,value=8000
  813782218261011172					column=ship:addr， timestamp=1593484525766，value=xian
  813782218261011172					column=ship:email, timestamp=1593484525807,value=139@sina.com
  
  813782218261011172					column=ship:salary, timestamp=1593484525838, value=10000
  5 row(s) in 0.0220 seconds
  ```
  
- exists：验证表是否存在。

  例子：

  ```sql
  hbase(main):024∶0> exists 'user'
  Table user does exist
  
  0 row(s) in 0.0750 seconds
  ```

  ```sql
  hbase(main):015:0> exists 't111'
  Table t111 does not exist
  
  0 row(s) in 0.0400 seconds
  ```

- drop：从HBase中删除表。

  用drop命令可以删除表，但在删除表之前必须先将其禁用。

  ```sql
  hbase(main):018:0> disable 'user'
  0 row(s) in 1.4580 seconds
  
  
  hbase(main):019:0> drop 'user'
  0 row(s) in 0.3060 seconds
  ```

  验证：使用exists命令

  ```sql
  hbase(main):020∶0>exists 'user'
  Table user does not exist
  
  0 row(s) in 0.0730 seconds
  ```

- drop_all：批量删除表

  例子：

  ```sql
  hbase> drop_all 't.*'
  t
  test1
  
  Drop the above 2 tables (y/n)?
  
  y
  2 tables successfully dropped
  ```

  **注意：**所有的表在被删除之前，都要先被禁用。

  

ava Admin API：在此之前所有的上述命令，Java提供了一个通过APl编程来管理实现DDL功能。

在这个org.apache.hadoop.hbase.client包中有 Admin 和 TableDescriptor 这两个重要的类提供DDL功能。



### JAVA API

- Hbase有多种不同的客户端，如REST客户端，Thift客户端，ORM框架Kundera等等。Hbase也提供了Java的API来操作表与列簇等信息，它的shell就是对Java的API做了一层封装。
- Hbase的Java API提供了很多高级的特性：
  - 元数据管理，列簇的数据压缩，region分隔
  - 创建，删除，更新，读取 rowkey

#### 添加依赖

```xml
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <!--Hadoop版本控制-->
    <hadoop.version>3.1.2</hadoop.version>
    <!-- HBase版本控制-->
    <hbase.version>2.2.5</hbase.version>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-auth</artifactId>
      <version>${hadoop.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-client</artifactId>
      <version>${hadoop.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-hdfs</artifactId>
      <version>${hadoop.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.hbase</groupId>
      <artifactId>hbase-client</artifactId>
      <version>${hbase.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.hbase</groupId>
      <artifactId>hbase-server</artifactId>
      <version>${hbase.version}</version>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
    </dependency>
  </dependencies>
```



#### 添加配置

- 导出Hadoop的配置文件

- 导出HBase的配置文件

  ![image-20221204204132129](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/image-20221204204132129.png)



#### Java代码

- DDL

  - ```java
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.hbase.HBaseConfiguration;
    import org.apache.hadoop.hbase.NamespaceDescriptor;
    import org.apache.hadoop.hbase.TableName;
    import org.apache.hadoop.hbase.client.*;
    import org.apache.hadoop.hbase.util.Bytes;
    import org.junit.After;
    import org.junit.Before;
    import org.junit.Test;
    
    import java.io.IOException;
    import java.util.ArrayList;
    import java.util.List;
    
    public class HelloHBaseDDL {
        //获取HBase管理员类
        private Admin admin;
        //获取表信息
        private Table table;
        // 获取数据库连接
        private Connection connection;
    
        /**
         * 命名空间
         *
         * @throws IOException
         */
        @Test
        public void createNameSpace() throws IOException {
            NamespaceDescriptor mkNameSpace = NamespaceDescriptor.create("xxx").build();
            this.admin.createNamespace(mkNameSpace);
            System.out.println("HelloHBase.createNameSpace[表空间创建完成]");
        }
    
        /**
         * 创建单列族的表
         *
         * @throws IOException
         */
        @Test
        public void createOneColumnFamilyTable() throws IOException {
            TableDescriptorBuilder table =
                    TableDescriptorBuilder.newBuilder(TableName.valueOf("xxx:teacher"));
            ColumnFamilyDescriptorBuilder columnBuilder =
                    ColumnFamilyDescriptorBuilder.newBuilder(Bytes.toBytes("info"));
            ColumnFamilyDescriptor familyDescriptor = columnBuilder.build();
            table.setColumnFamily(familyDescriptor);
            admin.createTable(table.build());
        }
    
        /**
         * 创建多列族的表
         *
         * @throws IOException
         */
        @Test
        public void createMultiPartColumnFamilyTable() throws IOException {
            TableDescriptorBuilder table =
                    TableDescriptorBuilder.newBuilder(TableName.valueOf("xxx:student"));
            ColumnFamilyDescriptorBuilder infoCF =
                    ColumnFamilyDescriptorBuilder.newBuilder(Bytes.toBytes("info"));
            ColumnFamilyDescriptorBuilder scoreCF =
                    ColumnFamilyDescriptorBuilder.newBuilder(Bytes.toBytes("score"));
            List<ColumnFamilyDescriptor> columnFamilyDescriptors = new ArrayList<>();
            columnFamilyDescriptors.add(infoCF.build());
            columnFamilyDescriptors.add(scoreCF.build());
            table.setColumnFamilies(columnFamilyDescriptors);
            admin.createTable(table.build());
        }
    
        /**
         * 查询所有表的信息
         *
         * @throws IOException
         */
        @Test
        public void listTable() throws IOException {
            TableName[] tableNames = admin.listTableNames();
            for (TableName tableName : tableNames) {
                System.out.println(tableName);
            }
        }
    
        /**
         * 列出指定命名空间的表信息
         *
         * @throws IOException
         */
        @Test
        public void listTableByNameSpace() throws IOException {
            TableName[] tableNames = admin.listTableNamesByNamespace("xxx");
            for (TableName tableName : tableNames) {
                System.out.println(tableName);
            }
        }
    
        @Before
        public void init() throws IOException {
            Configuration configuration = HBaseConfiguration.create();
            this.connection = ConnectionFactory.createConnection(configuration);
            this.admin = connection.getAdmin();
            //获取数据库表信息
            table = this.connection.getTable(TableName.valueOf("xxx:student"));
        }
    
        @After
        public void destroy() throws IOException {
            if (admin != null) {
                admin.close();
            }
            if (table != null) {
                table.close();
            }
            if (connection != null) {
                connection.close();
            }
        }
    
    }
    ```

    

- DML

  - ```java
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.hbase.HBaseConfiguration;
    import org.apache.hadoop.hbase.TableName;
    import org.apache.hadoop.hbase.client.*;
    import org.apache.hadoop.hbase.util.Bytes;
    import org.junit.After;
    import org.junit.Before;
    import org.junit.Test;
    
    import java.io.IOException;
    import java.util.ArrayList;
    import java.util.List;
    
    public class HelloHBaseDML {
        // 获取HBase管理员类
        private Admin admin;
        // 获取数据库连接
        private Connection connection;
        // 获取表信息
        private Table table;
    
        /**
         * 向数据库添加单行数据
         *
         * @throws IOException
         */
        @Test
        public void putOneRowData() throws IOException {
            Put put = new Put(Bytes.toBytes("s2020111701"));
            put.addColumn(Bytes.toBytes("info"), Bytes.toBytes("name"), Bytes.toBytes("lg"));
            put.addColumn(Bytes.toBytes("info"), Bytes.toBytes("age"), Bytes.toBytes(16));
            put.addColumn(Bytes.toBytes("info"), Bytes.toBytes("sex"), Bytes.toBytes("male"));
            put.addColumn(Bytes.toBytes("info"), Bytes.toBytes("money"), Bytes.toBytes(2000.0));
    
            put.addColumn(Bytes.toBytes("score"), Bytes.toBytes("Chinese"), Bytes.toBytes(98));
            put.addColumn(Bytes.toBytes("score"), Bytes.toBytes("English"), Bytes.toBytes(96));
            put.addColumn(Bytes.toBytes("score"), Bytes.toBytes(" Math"), Bytes.toBytes(95));
    
            table.put(put);
            table.close();
        }
    
        /**
         * 向数据库添加多行数据
         *
         * @throws IOException
         */
        @Test
        public void putMoreRowData() throws IOException {
            List<Put> puts = new ArrayList<>();
            for (int i = 0; i < 1000; i++) {
                Put put = new Put(Bytes.toBytes("s205012" + i));
                put.addColumn(Bytes.toBytes("info"), Bytes.toBytes("name"), Bytes.toBytes("name" + i));
                put.addColumn(Bytes.toBytes("info"), Bytes.toBytes("age"), Bytes.toBytes(i % 10 + 18));
                put.addColumn(Bytes.toBytes("info"), Bytes.toBytes("sex"), Bytes.toBytes(i % 9 == 0 ? "male" : "female"));
                put.addColumn(Bytes.toBytes("info"), Bytes.toBytes("money"), Bytes.toBytes(i % 90 + 10));
    
                put.addColumn(Bytes.toBytes("score"), Bytes.toBytes("Chinese"), Bytes.toBytes(i % 100));
                put.addColumn(Bytes.toBytes("score"), Bytes.toBytes("English"), Bytes.toBytes(i % 100));
                put.addColumn(Bytes.toBytes("score"), Bytes.toBytes("Math"), Bytes.toBytes(i % 100));
    
                puts.add(put);
            }
            table.put(puts);
            table.close();
        }
    
        /**
         * 删除行信息
         *
         * @throws IOException
         */
        @Test
        public void deleteRowData() throws IOException {
            Delete delete = new Delete("s2020111701".getBytes());
            table.delete(delete);
            System.out.println("删除数据! s2020111701");
        }
    
        @Before
        public void init() throws IOException {
            Configuration configuration = HBaseConfiguration.create();
            this.connection = ConnectionFactory.createConnection(configuration);
            this.admin = connection.getAdmin();
            //获取数据库表信息
            table = this.connection.getTable(TableName.valueOf("xxx:student"));
        }
    
        @After
        public void destroy() throws IOException {
            if (admin != null) {
                admin.close();
            }
            if (table != null) {
                table.close();
            }
            if (connection != null) {
                connection.close();
            }
        }
    
    }
    ```

    

- DQL

  - ```java
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.hbase.Cell;
    import org.apache.hadoop.hbase.HBaseConfiguration;
    import org.apache.hadoop.hbase.TableName;
    import org.apache.hadoop.hbase.client.*;
    import org.apache.hadoop.hbase.util.Bytes;
    import org.junit.After;
    import org.junit.Before;
    import org.junit.Test;
    
    import java.io.IOException;
    import java.util.ArrayList;
    import java.util.Arrays;
    import java.util.List;
    
    public class HelloHBaseDQL {
        // 获取HBase管理员类
        private Admin admin;
        // 获取数据库连接
        private Connection connection;
        // 获取表信息
        private Table table;
    
        /**
         * 根据RK获取表信息
         *
         * @throws IOException
         */
        @Test
        public void getRowData() throws IOException {
            //获取Get
            Get get = new Get(Bytes.toBytes("s205012999"));
            Result rs = table.get(get);
            Cell[] cells = rs.rawCells();
            for (Cell cell : cells) {
                String columnName = Bytes.toString(cell.getQualifierArray(),
                        cell.getQualifierOffset(), cell.getQualifierLength());
                String columnValue = null;
                if (columnName.equals("money") || columnName.equals("age")) {
                    int result = Bytes.toInt(cell.getValueArray());
                    columnValue = Integer.toString(result);
                } else {
                    columnValue = Bytes.toString(cell.getValueArray(),
                            cell.getValueOffset(), cell.getValueLength());
                }
                System.out.println(columnName + ":" + columnValue);
            }
        }
    
        /**
         * 获取多行数据
         *
         * @throws IOException
         */
        @Test
        public void getRowDataList() throws IOException {
            List<Get> gets = new ArrayList<>();
            gets.add(new Get(Bytes.toBytes("s001")));
            gets.add(new Get(Bytes.toBytes("s002")));
            gets.add(new Get(Bytes.toBytes("s003")));
            gets.add(new Get(Bytes.toBytes("s004")));
            Result[] rs = table.get(gets);
            for (Result r : rs) {
                Cell[] cells = r.rawCells();
                for (Cell c : cells) {
                    String columnName = Bytes.toString(c.getQualifierArray(),
                            c.getQualifierOffset(), c.getQualifierLength());
                    String columnValue = null;
                    if (columnName.equals("money") || columnName.equals("age")) {
                        int result = Bytes.toInt(c.getValueArray());
                        columnValue = Integer.toString(result);
                    } else {
                        columnValue = Bytes.toString(c.getValueArray(), c.getValueOffset(), c.getValueLength());
                    }
                    System.out.println(columnName + ":" + columnValue);
                }
            }
        }
    
        /**
         * 扫描整个表空间
         *
         * @throws IOException
         */
        @Test
        public void scanWholeTable() throws IOException {
            Scan scan = new Scan();
            scan.withStartRow(Bytes.toBytes("s01"));
            scan.withStopRow(Bytes.toBytes("s999999999999"));
            ResultScanner scanner = table.getScanner(scan);
            for (Result rs : scanner) {
                System.out.println("rowKey:" + Bytes.toString(rs.getRow()));
                Cell[] cells = rs.rawCells();
                for (Cell cell : cells) {
                    String columnName = Bytes.toString(cell.getQualifierArray(),
                            cell.getQualifierOffset(), cell.getQualifierLength());
                    String columnValue = null;
                    if (Arrays.asList("age", "money ", "Chinese", "English", "Math").contains(columnName)) {
                        int result = Bytes.toInt(cell.getValueArray());
                        columnValue = Integer.toString(result);
                    } else {
                        columnValue = Bytes.toString(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength());
                    }
                    System.out.println(columnName + ":" + columnValue);
                }
            }
        }
    
        @Before
        public void init() throws IOException {
            Configuration configuration = HBaseConfiguration.create();
            this.connection = ConnectionFactory.createConnection(configuration);
            this.admin = connection.getAdmin();
            //获取数据库表信息
            table = this.connection.getTable(TableName.valueOf("xxx:student"));
        }
    
        @After
        public void destroy() throws IOException {
            if (admin != null) {
                admin.close();
            }
        }
    
    }
    ```





## HBase读写流程

### 公共流程(三层索引)

- HBase中单表的数据量通常可以达到TB级或PB级，但大多数情况下数据读取可以做到毫秒级。HBase是如何做到的呢？要想实现表中数据的快速访问，通用的做法是数据保持有序并尽可能的将数据保存在内存里。HBase也是这样实现的。
- 对于海量级的数据，首先要解决存储的问题。数据存储上，HBase将表切分成小一点的数据单位region，托管到RegionServer上，和以前关系数据库分区表类似。但比关系数据库分区、分库易用。这一点在数据访问上，HBase对用户是透明的。数据表切分成多个Region，用户在访问数据时，如何找到该条数据对应的region呢？

#### HBase 0.96以前

- 有两个特殊的表，-Root- 和 .Meta. ，用来查找各种表的region位置在哪里。-Root- 和 .Meta. 也像HBase中其他表一样会切分成多个region。-Root- 表比 .Meta. 更特殊一些，永远不会切分超过一个region。-ROOT- 表的region位置信息存放在Zookeeper中，通过Zookeeper可以找到 -ROOT- region托管的RegionServer。通过 -ROOT- 表就可以找到 .META. 表region位置。.META. 表中存放着表切分region的信息。

- 当客户端访问一个表的时候，首先去询问Zookepper
  - Zookepper会告诉客户端 -Root- 表的Region所在的RegionServer
  
- 公共表 .meta.
  - 是一张普通表，但是由HBase自己维护
  
  - ```
    Key:
    
    	Region key的格式是: [table],[region start key],[region id]
    	
    Values:
    
    	info:regioninfo: 序列化的当前region的HRegionInfo实例。
    	
    	info:server: 存储这个region的regionserver的server:port
    	
    	info:serverstartcode: 该Regionserver拥用该region的起始时间
    ```
  
    ```
    - | RowKey		      | Info:regioninfo | info:server | info:serverstartcode |
      | ----------------- | --------------- | ----------- | -------------------- |
      | student-201800-r1 | r1:info         | rs03:16020  | s:201800 e:201899    |
      | student-201900-r2 | r2:info         | rs04:16020  | s:201900 e:201999    |
      | student-202000-r3 | r3:info         | rs05:16020  | s:202000 e:202099    |
      | teacher-201800-r1 | r4:info         | rs01:16020  | s:201800 e:201899    |
    
    - 假如我现在查询rowkey=student:201811
    
    - 假设有3000张表，每张表3000个Region，于是.meta.有900W条记录
    
    - 但是:
    
    - meta表数据量多的时候也会被切分成多个Region，位于多个RegionServer上
    - 当我们询问Zookepper的时候，也会给我们多个数据让我们去找结果
    ```
  
- 公共表 -Root-

  - 是一张普通的表，但是由HBase自己维护

  - ```
    Key:
    
    	Key的格式为: .META.
    	
    Values:
    
    	info:regioninfo: 序列化的hbase:meta表的HRegionInfo实例。
    	
    	info:server: 存储hbase:meta表的regionserver的server:port
    	
    	info:serverstartcode: 该Regionserver拥用hbase:meta表的起始时间
    ```

  - 它的结构和meta—模一样，但是它只维护meta表的切分信息

  - 理论上 -Root- 表不会被切分（数据量)

  - | RowKey                | info:regioninfo         | info:server | info:serverstartcode |
    | --------------------- | ----------------------- | ----------- | -------------------- |
    | .META.-student2018-ts | s:201800 e:201899 score | rs03        | s:201800 e:201899    |
    | .META.-teacher2018-ts | s:201900 e:201999 score | rs04        | s:201900 e:201999    |

     

- 查询流程

  - client--------->zookeeper---------> -ROOT- (单region)-----> .META. ----------->用户表region
  - ![img](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/u=1579991902,996963971&fm=253&app=138&f=PNG&fmt=auto&q=75)

- ![image-20221207170014533](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/image-20221207170014533.png)



#### HBase 0.96以后

- -ROOT-表被移除，直接将.Meta.表region位置信息存放在Zookeeper中。Meta表更名为hbase:meta，部分内容如下：

  ```shell
  hbase(main):003:0> scan 'hbase:meta'
  ROW                                                   COLUMN+CELL                                      hbase:namespace                                      column=table:state, timestamp=1667493149118, value=\x08\x00
  
  hbase:namespace,,1667493148075.e0f8762676145305a5440 column=info:regioninfo, timestamp=1669785868151, value={ENCODED => e0f8762676145305a544053504313483, NAME=>'hbase:namespace,,1667493148075.e0f87626761453053504313483.5a544053504313483.', STARTKEY => '', ENDKEY => ''}
  
  hbase:namespace,,1667493148075.e0f8762676145305a5440 column=info:seqnumDuringOpen,timestamp=1669785868151, value=\x00\x00\x00\x00\x00\x00\x00\x1453504313483.
  
  hbase:namespace,,1667493148075.e0f8762676145305a5440 column=info:server, timestamp=1669785868151, value=Node3:1602053504313483.
  
  hbase:namespace,,1667493148075.e0f8762676145305a5440 column=info:serverstartcode, timestamp=1669785868151, value=166976965140053504313483.
  
  hbase:namespace,,1667493148075.e0f8762676145305a5440 column=info:sn, timestamp=1669785867149, value=node3,16020,166976965140053504313483.
  
  hbase:namespace,,1667493148075.e0f8762676145305a5440 column=info:state, timestamp=1669785868151, value=OPEN53504313483.
  
  xxx:student                                          column=table:state, timestamp=1669785801906, value=\x08\x00
  
  xxx:student,,1669785800999.821bb6b8d34c63ebba83273bb column=info:regioninfo, timestamp=1669785801886, value={ENCODED => 821bb6b8d34c63ebba83273bb2e0369b, NAME => 'xxx:student,,1669785800999.821bb6b8d34c63ebba82e0369b.3273bb2e0369b.', STARTKEY => '', ENDKEY => ''}
  
  xxx:student,,1669785800999.821bb6b8d34c63ebba83273bb column=info:seqnumDuringOpen, timestamp=1669785801886, value=\x00\x00\x00\x00\x00\x00\x00\x022e0369b.
  
  xxx:student,,1669785800999.821bb6b8d34c63ebba83273bb column=info:server, timestamp=1669785801886, value=Node2:160202e0369b.
  
  xxx:student,,1669785800999.821bb6b8d34c63ebba83273bb column=info:serverstartcode, timestamp=1669785801886, value=16697696514582e0369b.
  
  xxx:student,,1669785800999.821bb6b8d34c63ebba83273bb column=info:sn, timestamp=1669785801510, value=node2,16020,16697696514582e0369b.
  
  xxx:student,,1669785800999.821bb6b8d34c63ebba83273bb column=info:state, timestamp=1669785801886, value=OPEN2e0369b.
  
  xxx:teacher                                          column=table:state, timestamp=1669785751772, value=\x08\x00                                                                                                 
  xxx:teacher,,1669785750158.c8eefc8dda9aeb5be16b5956a column=info:regioninfo, timestamp=1669785751643, value={ENCODED => c8eefc8dda9aeb5be16b5956a2675af8, NAME => 'xxx:teacher,,1669785750158.c8eefc8dda9aeb5be162675af8.b5956a2675af8.', STARTKEY => '', ENDKEY => ''}
  
  xxx:teacher,,1669785750158.c8eefc8dda9aeb5be16b5956a column=info:seqnumDuringOpen, timestamp=1669785751643, value=\x00\x00\x00\x00\x00\x00\x00\x022675af8.
  
  xxx:teacher,,1669785750158.c8eefc8dda9aeb5be16b5956a column=info:server, timestamp=1669785751643, value=Node1:160202675af8.
  
  xxx:teacher,,1669785750158.c8eefc8dda9aeb5be16b5956a column=info:serverstartcode, timestamp=1669785751643, value=16697696566252675af8.
  
  xxx:teacher,,1669785750158.c8eefc8dda9aeb5be16b5956a column=info:sn, timestamp=1669785751027, value=node1,16020,16697696566252675af8.
  
  xxx:teacher,,1669785750158.c8eefc8dda9aeb5be16b5956a column=info:state, timestamp=1669785751643, value=OPEN2675af8.
  
  6 row(s)
  Took 0.2512 seconds 
  ```

  | RowKey                           | Info                                                         | Info:server |
  | -------------------------------- | ------------------------------------------------------------ | ----------- |
  | student,,1594049755757.6faa831a8 | regioninfo: {STARTKEY => ",ENDKEY => "} serverstartcode:1594048338976 | Node3:16020 |

- 查询流程

  - client--------->zookeeper-------->hbase:meta--------->用户表region



### 读取数据流程

![img](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/u=204843296,1942922580&fm=253&app=138&f=PNG&fmt=auto&q=75)

- 首先客户端和RegionServer建立连接
- 查找对应的Region，然后查找对应的列族（Store）
- 查找memstore中是否存在对应的数据,然后找到直接返回
- 如果找不到去RegionServer（BlockCache）
- 开始去查找strorefile，因为storefile中的数据是有序的，所以查找速度比较快
- 如果查找到数据就将数据返回到客户端，客户端会保留数据的缓存信息



### 写入数据流程

- 首先客户端和RegionServer建立连接
- 然后将DML要做的操作写入到日志wal-log
- 然后将数据的修改更新到memstore中，然后本次操作结束
  - 一个region由多个store组成，一个store对应一个CF（列族），store包括位于内存中的memstore和位于磁盘的storefile，写操作先写入memstore。
- 当memstore数据写到阈值之后，创建一个新的memstore
- 旧的memstore写成一个独立的storefile，regionserver会启动flashcache进程写入storefile，每次写入形成单独的一个storefile,存放到hdfs
  - ![img](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/u=1773348331,2118146325&fm=253&app=138&f=PNG&fmt=auto&q=75)
- 当storefile文件的数量增长到一定阈值后，系统会进行合并（minor、major、compaction）
- 在合并过程中会进行版本合并和删除工作，形成更大的storefile
- 当一个region所有storefile的大小和数量超过一定阈值后，会把当前的region分割为两个，并由hmaster分配到相应的regionserver服务器，实现负载均衡
- Store负责管理当前列族的数据
  - 1 *memstore+n* storefile
  - memstore
  - storefile
- 当我们进行数据DML的时候，以插入数据为例
  - 我们会将数据先存储到memStore中，当memStore达到阈值（128M）
  - 首先创建一个新的memstore
  - 然后会将memStore中的数据写成一个storefile，storefile会存储到hdfs上(hfile)
- 随着时间的推移：
  - HFile中会存放大量的失效数据（删除，修改）
  - 会产生多个HFile
  - 等达到阈值（时间、数量）会进行合并
    - 多个HFile合并成一个大的HFile
    - 合并会触发连锁反应，相邻的store也会进行合并
- 在Hbase中，表被分割成多个更小的块然后分散的存储在不同的服务器上，这些小块叫做Regions，存放
  Regions的地方叫做RegionServer。Master进程负责处理不同的RegionServer之间的Region的分发。在Hbase实现中HRegionServer和HRegion类代表RegionServer和Region。HRegionServer除了包含一些HRegions之外，还处理两种类型的文件用于数据存储
  - HLog 预写日志文件，也叫做WAL（write-ahead log）
  - HFile 是HDFS中真实存在的数据存储文件





## HBase经典设计案例

### Hbase表设计因素

- 只要是数据库都存在，模式设计的问题，关系型中有模式设计的范式，Hbase作为列式存储数据库，其模式设计也非常重要。
- Hbase关键概念：表，rowkey，列簇，时间戳
  - 这个表应该有多少列簇
  - 列簇使用什么数据
  - 每个列簇有有多少列
  - 列名是什么，尽管列名不必在建表时定义，但读写数据是要知道的
  - 单元应该存放什么数据
  - 每个单元存储多少时间版本
  - 行健(rowKey)结构是什么，应该包含什么信息



### Hbase表设计要点

#### 行键设计

- 关键部分，直接关系到后续服务的访问性能。如果行健设计不合理，后续查询服务效率会成倍的递减。
- 行健短到可读即可，因为查询短键不比长键性能好多少，所以设计时要权衡长度。
- 行键不能改变，**唯一可以改变的方式是先删除后插入**

#### 行键打散

- 避免单调的递增行键
- Hbase的行健是有序排列的，这样可能导致一段时间内大部分写入集中在某一个Region上进行操作，负载都在一台节点上。
- 可以设计成：不同的metric_type可以将压力分散到不同的region上，如果本次存储使用时间的毫秒数作为rowkey
- 数据会连续性的存放到hbase，如果客户对数据的关注点主要是最近的记录
- 那么会导致存放最近数据的节点压力大
- 如果我们的数据确实不需要连续查询，一般可以让当前时间毫秒数逆序
  - 但是这样会不利于数据的连续读取
- 减轻单节点的压力

#### 列簇设计

- 列簇是一些列的集合，一个列簇的成员有相同的前缀，以冒号(:)作为分隔符。
- 现在Hbase不能很好处理2~3个以上的列簇，所以尽可能让列簇少一些，如果表有多个列簇，列簇A有100万行数据，列簇B有10亿行，那么列簇B会分散到很多的Region导致扫描列簇B的时候效率低下。
- 列簇名的长度要尽量小，一个为了节省空间，另外加快效率，比如d表示data，v表示value

#### 列簇属性

- Hfile数据块，默认是64KB，数据库的大小影响数据块索引的大小。

- 数据块大的话，一次加载进内存的数据越多，扫描查询效果越好。但是数据块小的话，随机查询性能更好

  ```shell
  > create 'mytable',{NAME => 'cf1', BLOCKSIZE => '65536'}
  ```

- 数据块缓存，数据块缓存默认是打开的，如果一些比较少访问的数据可以选择关闭缓存

  ```shell
  > create 'mytable',{NAME => 'cf1', BLOCKCACHE => 'FALSE'}
  ```

- 数据压缩，压缩会提高磁盘利用率，但是会增加CPU的负载，看情况进行控制

  ```shell
  > create 'mytable',{NAME => 'cf1', COMPRESSION => 'SNAPPY'}
  ```

Hbase表设计是和需求相关的，但是遵守表设计的一些硬性指标对性能的提升还是很有帮助的，这里整理了一些设计时用到的要点。



### 单表RowKey设计

#### 移动通话记录

![image-20221219023659863](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/image-20221219023659863.png)

- 数据量统计
  - 数据量：1E * 100 * 365 = 36500E 通话记录/年
  - 查询方式：从开始日期到结束日期
  - 实现思想：尽量将要查询的数据体现在主键上，借助于RowKey实现快速定位
- 方案1
  - 身份证号_手机号码，所有的数据都存放到一行，通过版本进行控制
  - 缺点：
    - 只能保存一条记录，如果想扩充数据只能修改version迭代
    - 每个人通话记录数是不同的，version数目受限
- 方案2
  - 时间戳_random
  - 20191128133852123_uuid
  - 缺点：
    - 相当于从100E数据中心获取到100条，要整体遍历数据，效率低
- 方案3
  - 手机号码_时间
  - 首先按照手机号码排序，手机号相同的会排在一起
  - 然后按照时间排序，会按照时间先后顺序排序

#### 京东订单

![QQ截图20221219024948](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/QQ%E6%88%AA%E5%9B%BE20221219024948.png)

| 名称    |                                                              |
| ------- | ------------------------------------------------------------ |
| RowKey  | uname_time stamp 将相同用户的订单全部排列到一起，查询的时候只是从里面取出一小部分 |
| 列簇    | order                                                        |
| 列      | datetime、orderid、usernamegoodimges、title、number、money   |
| version | 1                                                            |



### 多表关联关系设计

#### 人员职位映射表

- 身份：

  - 语文课代表  体育委员  音乐课代表  学生会主席

  - | ID   | job        |
    | ---- | ---------- |
    | job1 | 语文课代表 |
    | job2 | 数学课代表 |
    | job3 | 体育课代表 |
    | job4 | 音乐课代表 |
    | job5 | 学生会主席 |

- 人员：

  - 小红  小黄  小蓝  小黑

  - | id   | name |
    | ---- | ---- |
    | p1   | 小红 |
    | p2   | 小黄 |
    | p3   | 小蓝 |
    | p4   | 小黑 |

- 规则：

  - 一个人可以有多个身份
  - 一个身份可以被多个人拥有
  - 一个人可以指定他的主要身份

- 要求：

  - 通过这个人的ID可以获取身份
  - 通过身份可以获取个人
  - 可以修改人的身份
  - 可以修改身份对应的人员
  - 人员可以设置主身份



- 方案1

  - mysql --中间表(中间表数据量过大)

  - | id   | jobid | pid  | type |
    | ---- | ----- | ---- | ---- |
    | uuid | job1  | p1   | a    |
    | uuid | job2  | p2   | a    |
    | uuid | job1  | p2   | s    |

- 方案2

  - 将用户的身份直接放在用户表
  - 将身份的用户直接放在身份表
  - 重新构建一个列族，列族中存在一个属性叫做array，专门存放数据
  - 当用户新增数据的数据的时候直接添加到数组
  - 但是当用户取消修改的时候，需要遍历数组找到对应的记录
  - 缺点：
    - 如果我们在添加的数据的时候就维护一个有序的数组，成本太高

- 方案3

  - HBase --冗余存储
  - 此方案基于方案2：
    - 构建一个列族，将和自己关联到一起的数据全部以列名的方式存放
  - 身份表
  - | rowkey | job        | person |
    | ------ | ---------- | ------ |
    | job1   | 语文       | p1, p3 |
    | job2   | 数学       | p2, p4 |
    | job3   | 体育       | p1     |
    | job4   | 音乐课代表 | p3     |
    | job5   | 学生会主席 | p2     |
  
  - 人员表
  
  - | rowkey | name | job        |
    | ------ | ---- | ---------- |
    | p1     | 小红 | job1，job3 |
    | p2     | 小黄 | job2，job5 |
    | p3     | 小蓝 | job1，job4 |
    | p4     | 小黑 | job2       |



#### 微博关系处理

![image-20221219171010970](https://cdn.jsdelivr.net/gh/moondream69/TyporaImages/images/image-20221219171010970.png)

人员表(rowkey，列族(个人信息, 安全, 统计信息(关注数，被关注数，发帖数) ) ) 

关注表(rowkey，列族(关注:rowkey) ) 

粉丝表(rowkey，列族(被关注:rowkey) ) 

发帖表(人员rowkey_timestamp)

| RowKey | 关注：G          | 粉丝：F           |
| ------ | ---------------- | ----------------- |
| lbb    | _count: 2 cl,wyf | _count: 1 lyf     |
| cl     | _count: 1 lyf    | _count: 2 lbb,lyf |
| wyf    | _count: 1 lyf    | _count: 1 lbb     |
| lyf    | _count: 2 lbb,cl | _count: 2 cl,wyf  |





## HBase常用优化

一个系统上线之后，开发和调优将一直贯穿系统的生命周期中，HBase也不列外。这里主要说一些Hbase的调优

### HBase表优化

#### 预分区

- Pre-Creating Regions
- 默认情况下，在创建HBase表的时候会自动创建一个region分区，当导入数据的时候，所有的 HBase客户端都向这一个region写数据，直到这个region足够大了才进行切分。
- 有种加快批量写入速度的方法是通过预先创建一些空的regions，这样当数据写入HBase时，会按 照region分区情况，在集群内做数据的负载均衡。

```java
public void createTableRegions() throws IOException {
	TableDescriptorBuilder table =
TableDescriptorBuilder.newBuilder(TableName.valueOf("bdp:calllog"));
	ColumnFamilyDescriptorBuilder info =
ColumnFamilyDescriptorBuilder.newBuilder(Bytes.toBytes("info"));
	table.setColumnFamily(info.build());
	/**
		* 1、中国移动号段为：134、135、136、137、138、139、147、148、150、151、152、
157、158、159、178、182、183、184、187、188、198、1440、1703、1705、1706
		*
		* 2、中国联通号段为：130、131、132、155、156、185、186、145、146、166、167、
175、176、1704、1707、1708、1709、1710、1711、1712、1713、1714、1715、1716、1717、
1718、1719
		*
		* 3、中国电信号段为：133、153、177、180、181、189、191、199、1349、1410、
1700、1701、1702、1740 。
		*/
    byte[][] splits = {
        Bytes.toBytes("134"),
        Bytes.toBytes("135"),
        Bytes.toBytes("136"),
        Bytes.toBytes("137"),
        Bytes.toBytes("138"),
        Bytes.toBytes("139"),
        Bytes.toBytes("181"),
        Bytes.toBytes("182"),
        Bytes.toBytes("183"),
        Bytes.toBytes("186"),
    };
	admin.createTable(table.build(), splits);
}
```



#### rowkey

- HBase中row key用来检索表中的记录，支持以下三种方式：

  - 通过单个row key访问：即按照某个row key键值进行get操作；
  - 通过row key的range进行scan：即通过设置startRowKey和endRowKey，在这个范围内进行扫描； 
  - 全表扫描：即直接扫描整张表中所有行记录。

- row key可以是任意字符串，最大长度64KB，实际应用中一般为10~100bytes，存为byte[]字节数组，**一般设计成定长的**。

- row key是按照**字典序**存储，因此，设计row key时，将经常一起读取的数据存储到一块，将最近可能会被访问的数据放在一块。

  - 举个例子：如果最近写入HBase表中的数据是最可能被访问的，可以考虑将时间戳作为row key的一部分，由于是字典序排序，所以可以使用Long.MAX_VALUE - timestamp作为row key，这样能保证新写入的数据在读取时可以被快速命中。

- Rowkey规则：

  1. 越小越好

  2. Rowkey的设计是要根据实际业务来

  3. 定长

  4. 散列性

     ​	a) 取反 001 002 100 200 

     ​	b) Hash 

     ​	c) 加盐（在RowKey的前面或后面加上一个随机数）

  > （1）数据的持久化文件HFile中是按照KeyValue存储的，如果Rowkey过长比如100个字节，1000万列数据光Rowkey就要占用100*1000万=10亿个字节，将近1G数据，这会极大影响 Hfile的存储效率； 
  >
  > （2）MemStore将缓存部分数据到内存，如果Rowkey字段过长内存的有效利用率降低，系 统将无法缓存更多的数据，这会降低检索效率，因此Rowkey的字节长度越短越好。 
  >
  > （3）目前操作系统一般都是64位系统，内存8字节对齐，空值在16个字节，8字节的整数倍 利用操作系统的最佳特性。



#### ColumnFamily

- 不要在一张表里定义太多的column family。目前Hbase并不能很好的处理超过2~3个column family的表。因为某个column family在flush的时候，它邻近的column family也会因关联效应被触发flush，最终导致系统产生更多的I/O。



#### Version

- 创建表的时候，可以通过HColumnDescriptor.setMaxVersions(int maxVersions)设置表中数据的最大版本 
- 如果只需要保存最新版本的数据，那么可以设置setMaxVersions(1) 
- Time To Live 
  - 创建表的时候，可以通过HColumnDescriptor.setTimeToLive(int timeToLive)设置表中数据的存储生命期，过期数据将自动被删除，例如如果只需要存储最近两天的数据，那么可以设置 setTimeToLive(2 * 24 * 60 * 60)。



#### compact & Split

- 在HBase中，数据在更新时首先写入WAL 日志(HLog)和内存(MemStore)中，MemStore中的数据是排序的，当MemStore累计到一定阈值时，就会创建一个新的MemStore，并且将老的 MemStore添加到flush队列，由单独的线程flush到磁盘上，成为一个StoreFile。于此同时， 系统 会在zookeeper中记录一个redo point，表示这个时刻之前的变更已经持久化了(minor compact)。
- StoreFile是只读的，一旦创建后就不可以再修改。因此Hbase的更新其实是不断追加的操作。当一 个Store中的StoreFile数量达到一定的阈值后，就会进行一次合并(major compact)，将对同一个 key的修改合并到一起，形成一个大的StoreFile，当Region的大小达到一定阈值后，又会对 StoreFile进行分割(split)，逻辑等分为两个StoreFile。
- 由于对表的更新是不断追加的，处理读请求时，需要访问Store中全部的StoreFile和MemStore， 将它们按照row key进行合并，由于StoreFile和MemStore都是经过排序的，并且StoreFile带有内 存中索引，通常合并过程还是比较快的。
- 实际应用中，可以考虑必要时手动进行major compact，将同一个row key的修改进行合并形成一 个大的StoreFile。同时，可以将StoreFile设置大些，减少split的发生。
- minor compaction：较小、很少文件的合并
  - 数量 大小 的设置
- major compaction：完整性合并
  - hbase.hregion.majorcompaction 默认7天
- In Memory
  - 创建表的时候，通过HColumnDescriptor.setInMemory(true)将表放到RegionServer的缓存 中，保证在读取的时候被cache命中



### HBase写入优化

#### 多Table并发写

创建多个Table客户端用于写操作，提高写数据的吞吐量，一个例子：

```java
	public void bdpTables() throws IOException {
		Configuration conf = HBaseConfiguration.create();
        
		Connection conn = ConnectionFactory.createConnection(conf);
		String table_log_name = "tets11";
        
		Table[] wTableLog = new Table[2];
        
		Put p = new Put(Bytes.toBytes("row1"));
        
		p.addColumn(Bytes.toBytes("info"),Bytes.toBytes("age"),Bytes.toBytes("12"));
        
		Put p2 = new Put(Bytes.toBytes("row2"));
        
		p2.addColumn(Bytes.toBytes("info"),Bytes.toBytes("age"),Bytes.toBytes("12"));
        
		for (int i = 0; i < 2; i++) {
			wTableLog[i] = conn.getTable(TableName.valueOf(table_log_name));
		}
        
        wTableLog[0].put(p);
        wTableLog[1].put(p2);

        wTableLog[1].close();
        wTableLog[0].close();
        
	}
```

#### WAL Flag

在HBae中，客户端向集群中的RegionServer提交数据时（Put/Delete操作），首先会先写WAL（Write Ahead Log）日志（即HLog，一个RegionServer上的所有Region共享一个HLog），只有当WAL日志写 成功后，再接着写MemStore，然后客户端被通知提交数据成功；如果写WAL日志失败，客户端则被通 知提交失败。这样做的好处是可以做到RegionServer宕机后的数据恢复。

因此，对于相对不太重要的数据，可以在Put/Delete操作时，通过调用Put.setWriteToWAL(false)或 Delete.setWriteToWAL(false)函数，放弃写WAL日志，从而提高数据写入的性能。

值得注意的是：谨慎选择关闭WAL日志，因为这样的话，一旦RegionServer宕机，Put/Delete的数据将 会无法根据WAL日志进行恢复。

#### 批量写

通过调用Table.put(Put)方法可以将一个指定的row key记录写入HBase，同样HBase提供了另一个方法：通过调用Table.put(List<Put>)方法可以将指定的row key列表，批量写入多行记录，这样做的好处是批量 执行，只需要一次网络I/O开销，这对于对数据实时性要求高，网络传输RTT高的情景下可能带来明显的 性能提升。

#### HTable参数设置

- Auto Flush

通过调用HTable.setAutoFlush(false)方法可以将HTable写客户端的自动flush关闭，这样可以批量写入 数据到HBase，而不是有一条put就执行一次更新，只有当put填满客户端写缓存时，才实际向HBase服 务端发起写请求。默认情况下auto flush是开启的。

- Write Buffer

通过调用HTable.setWriteBufferSize(writeBufferSize)方法可以设置HTable客户端的写buffer大小，如 果新设置的buffer小于当前写buffer中的数据时，buffer将会被flush到服务端。其中，writeBufferSize 的单位是byte字节数，可以根据实际写入数据量的多少来设置该值。

- 多线程并发写

在客户端开启多个HTable写线程，每个写线程负责一个HTable对象的flush操作，这样结合定时flush和 写buffer（writeBufferSize），可以既保证在数据量小的时候，数据可以在较短时间内被flush（如1秒 内），同时又保证在数据量大的时候，写buffer一满就及时进行flush。下面给个具体的例子：

```java
for (int i = 0; i < threadN; i++) {
	Thread th = new Thread() {
		public void run() {
			while (true) {
				try {
					sleep(1000); //1 second
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
synchronized (wTableLog[i]) {
					try {
						wTableLog[i].flushCommits();
					} catch (IOException e) {
						e.printStackTrace();
					}
				}
			}
}
	};
	th.setDaemon(true);
	th.start();
}
```



### HBase读取优化

作为NoSQL数据库，增删改查是其最基本的功能，其中查询是最常用的一项。

#### 显示的指定列

当使用Scan或者GET获取大量的行时，最好指定所需要的列，因为服务端通过网络传输到客户端，数据 量太大可能是瓶颈。如果能有效过滤部分数据，能很大程度的减少网络I/O的花费。

```java
/**
 * Get all columns from the specified family.
 * <p>
 * Overrides previous calls to addColumn for this family.
 * @param family family name
 * @return this
 * 获取指定列簇的所有列
 */
public Scan addFamily(byte [] family) {
	familyMap.remove(family);
	familyMap.put(family, null);
	return this;
}

/**
 * Get the column from the specified family with the specified qualifier.
 * <p>
 * Overrides previous calls to addFamily for this family.
 * @param family family name
 * @param qualifier column qualifier
 * @return this
 * 获取指定列簇的特定列
 */
public Scan addColumn(byte [] family, byte [] qualifier) {
	NavigableSet<byte []> set = familyMap.get(family);
	if(set == null) {
		set = new TreeSet<byte []>(Bytes.BYTES_COMPARATOR);
	}
	if (qualifier == null) {
		qualifier = HConstants.userTY_BYTE_ARRAY;
	}
	set.add(qualifier);
	familyMap.put(family, set);
	return this;
}
```

一般用：<br>scan.addColumn(...)



#### 查询结果

对于频繁查询HBase的应用场景，可以考虑在应用程序和Hbase之间做一层缓存系统，新的查询先去缓 存查，缓存没有再去查Hbase。

- 多Table并发写

- scanner cache

  - hbase.client.scanner.caching配置项可以设置HBase scanner一次从服务端抓取的数据条 数，默认情况下一次一条。通过将其设置成一个合理的值，可以减少scan过程中next()的时间 开销，代价是scanner需要通过客户端的内存来维持这些被cache的行记录
  - HBase的conf配置文件中进行配置-->整个集群 
  - Table.setScannerCaching(int scannerCaching)进行配置-->本次对表的链接 
  - Scan.setCaching(int caching)-->本次查询 
  - 优先级 从本次查询--》本次链接--》整个集群

- Scan指定列族或者列

  - scan时指定需要的Column Family，可以减少网络传输数据量

- Close ResultScanner

  - 关闭ResultScanner

    如果在使用table.getScanner之后，忘记关闭该类，它会一直和服务端保持连接，资源无法释放，从而 导致服务端的某些资源不可用。

    所以在用完之后，需要执行关闭操作，这点与JDBS操作MySQL类似

    scanner.close()

- 批量读



### HBase缓存优化

#### 设置Scan缓存

HBase中Scan查询可以设置缓存，方法是setCaching()，这样可以有效的减少服务端与客户端的交互， 更有效的提升扫描查询的性能。

```java
/**
 * Set the number of rows for caching that will be passed to scanners.
 * If not set, the default setting from {@link Table#getScannerCaching()} will apply.
 * Higher caching values will enable faster scanners but will use more memory.
 * @param caching the number of rows for caching
 * 设置scanners缓存的行数
 */
public void setCaching(int caching) {
	this.caching = caching;
}
```



#### 禁用块缓存

如果批量进行全表扫描，默认是有缓存的，如果此时有缓存，会降低扫描的效率。 

scan.setCacheBlocks(true|false); 

对于经常读到的数据，建议使用默认值，开启块缓存

#### 缓存查询结果

对于频繁查询HBase的应用场景，可以考虑在应用程序和Hbase之间做一层缓存系统，新的查询先去缓 存查，缓存没有再去查Hbase。

- Blockcache
  - 隶属于RegionServer
  - mem刷新的时机
    - 当前mem达到128M
    - 集群mem总内存使用量达到阈值
  - 读请求先到Memstore中查数据，查不到就到BlockCache中查，再查不到就会到磁盘上读，并把读的结果放入BlockCache。
  - Regionserver上有一个BlockCache和N个Memstore，它们的大小之和 ! >= heapsize * 0.8
- 默认BlockCache为0.2，而Memstore为0.4。**对于注重读响应时间的系统，可以将** BlockCache设 大些，比如设置BlockCache=0.4，Memstore=0.39，以加大缓存的命中率





## HBase和MapReduce整合

### 公共代码

- 先加入core-site.xml hdfs-site.xml yarn-site.xml mapred-site.xml hbase-site.xml  log4j.properties

- 加入依赖

  - ```xml
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <!-- Hadoop版本控制 -->
        <hadoop.version>3.1.2</hadoop.version>
        <!-- HBase版本控制 -->
        <hbase.version>2.2.5</hbase.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>${hadoop.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>${hadoop.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-auth</artifactId>
            <version>${hadoop.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>${hbase.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-server</artifactId>
            <version>${hbase.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-mapreduce</artifactId>
            <version>${hbase.version}</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
        </dependency>
    </dependencies>
    ```



###  Hdfs数据存入Hbase

- 编写job类

  - ```java
    package com.xxxx.hdfs2hbase;
    
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.Path;
    import org.apache.hadoop.hbase.HBaseConfiguration;
    import org.apache.hadoop.hbase.mapreduce.TableMapReduceUtil;
    import org.apache.hadoop.io.IntWritable;
    import org.apache.hadoop.io.Text;
    import org.apache.hadoop.mapreduce.Job;
    import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
    
    import java.io.IOException;
    
    public class WordCountJob {
    	public static void main(String[] args) throws IOException,ClassNotFoundException, InterruptedException {
    		Configuration configuration = HBaseConfiguration.create();
    		configuration.set("mapreduce.framework.name", "local");
    		Job job = Job.getInstance(configuration, "hdfs2hbase-" +
    		System.currentTimeMillis());
            
    		//设置Mapper
            job.setJarByClass(WordCountJob.class);
            job.setNumReduceTasks(2);
            job.setMapOutputKeyClass(Text.class);
            job.setMapOutputValueClass(IntWritable.class);
            FileInputFormat.setInputPaths(job, new Path("/bdp/harry.txt"));
            job.setMapperClass(WordCountMapper.class);
            
    		//设置Reduce
    		TableMapReduceUtil.initTableReducerJob("wordcount",WordCountReducer.class, job, null, null, null, null, false);
            
    		job.waitForCompletion(true);
    	}
    }
    ```

- 编写map类

  - ```java
    package com.xxxx.hdfs2hbase;
    
    import org.apache.hadoop.io.IntWritable;
    import org.apache.hadoop.io.LongWritable;
    import org.apache.hadoop.io.Text;
    import org.apache.hadoop.mapreduce.Mapper;
    
    import java.io.IOException;
    
    public class WordCountMapper extends Mapper<LongWritable, Text, Text,IntWritable> {
        
    	@Override
    	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
    		//拆分字符串
    		String[] ss = value.toString().replaceAll("[^\\w\\s'-]", "").split("\\s+");
            
        	//开始拆分
    		for (int i = 0; i < ss.length; i++) {
                Text outputKey = new Text(ss[i]);
                context.write(outputKey, new IntWritable(1));
    		}
    	}
        
    }
    ```

- 编写reduce类

  - ```java
    package com.xxxx.hdfs2hbase;
    
    import org.apache.hadoop.hbase.client.Mutation;
    import org.apache.hadoop.hbase.client.Put;
    import org.apache.hadoop.hbase.io.ImmutableBytesWritable;
    import org.apache.hadoop.hbase.mapreduce.TableReducer;
    import org.apache.hadoop.io.IntWritable;
    import org.apache.hadoop.io.Text;
    import org.apache.hadoop.mapreduce.Reducer;
    
    import java.io.IOException;
    
    public class WordCountReducer extends TableReducer<Text,IntWritable, ImmutableBytesWritable> {
        
    	@Override
    	protected void reduce(Text key, Iterable<IntWritable> values,Reducer<Text, IntWritable, ImmutableBytesWritable, Mutation>.Context context) throws IOException, InterruptedException {
            //创建一个累加器
            int count = 0;
            
            //开始迭代value
            for (IntWritable value : values) {
            count += value.get();
            }
            
            //创建插入数据对象put
            Put put = new Put("harry".getBytes());
            put.addColumn("info".getBytes(), key.toString().getBytes(),String.valueOf(count).getBytes());
            //写出
            context.write(null, put);
    	}
        
    }
    ```



### HBase文件导出Hdfs

- 编写job类

  - ```java
    package com.xxxx.hbase2hdfs;
    
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.Path;
    import org.apache.hadoop.hbase.HBaseConfiguration;
    import org.apache.hadoop.hbase.client.Scan;
    import org.apache.hadoop.hbase.mapreduce.TableMapReduceUtil;
    import org.apache.hadoop.io.IntWritable;
    import org.apache.hadoop.io.Text;
    import org.apache.hadoop.mapreduce.Job;
    import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
    
    import java.io.IOException;
    
    public class PhoneJob {
        
    	public static void main(String[] args) throws IOException,ClassNotFoundException, InterruptedException {
        
            //获取配置文件
            Configuration configuration = HBaseConfiguration.create();
            configuration.set("mapreduce.framework.name", "local");
            //创建任务对象
            Job job = Job.getInstance(configuration, "hbase2hdfs-" +
            System.currentTimeMillis());
            
            //设置当前任务的主类
            job.setJarByClass(PhoneJob.class);
            //设置任务的其他信息
            job.setNumReduceTasks(2);
            
            //设置扫描器
            Scan scan = new Scan();
            scan.withStartRow("10".getBytes());
            scan.withStopRow("15".getBytes());
            
            //设置数据的map
            TableMapReduceUtil.initTableMapperJob("phone", scan,
            PhoneMapper.class, Text.class, IntWritable.class, job, false);
            
            //设置Reduce的处理类
            FileOutputFormat.setOutputPath(job, new
            Path("/bdp/result/hbase2hdfs" + System.currentTimeMillis()));
            job.setReducerClass(PhoneReducer.class);
            
            //提交任务
            job.waitForCompletion(true);
    	}
    }
    ```

- 编写map类

  - ```java
    package com.xxxx.hbase2hdfs;
    
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.hbase.HBaseConfiguration;
    import org.apache.hadoop.hbase.client.Result;
    import org.apache.hadoop.hbase.io.ImmutableBytesWritable;
    import org.apache.hadoop.hbase.mapreduce.TableMapper;
    import org.apache.hadoop.io.IntWritable;
    import org.apache.hadoop.io.Text;
    
    import java.io.IOException;
    
    /**
     * 13812341234_
     */
    public class PhoneMapper extends TableMapper<Text, IntWritable> {
        @Override
        protected void map(ImmutableBytesWritable key, Result value, Context context) throws IOException, InterruptedException {
            //获取手机号码
            String rowkey = new String(value.getRow());
            String phoneNum = rowkey.split("_")[0];
            
    		context.write(new Text(phoneNum), new IntWritable());
    	}
    }
    ```

- 编写reduce类

  - ```java
    package com.xxxx.hbase2hdfs;
    
    import org.apache.hadoop.io.IntWritable;
    import org.apache.hadoop.io.Text;
    import org.apache.hadoop.mapreduce.Reducer;
    
    import java.io.IOException;
    
    public class PhoneReducer extends Reducer<Text, IntWritable, Text,IntWritable> {
    	@Override
        protected void reduce(Text key, Iterable<IntWritable> iterable,Reducer<Text, IntWritable, Text, IntWritable>.Context context) throws IOException, InterruptedException {
            
    		context.write(key, new IntWritable());
    	}
    }
    ```





## Protobuf数据压缩

### 简介

- protocol buffers 是一种语言无关、平台无关、可扩展的序列化结构数据的方法，它可用于（数 据）通信协议、数据存储等。

- Protocol Buffers 是一种灵活，高效，自动化机制的结构数据序列化方法－可类比 XML，但是比 XML 更小（3 ~ 10倍）、更快（20 ~ 100倍）、更为简单。

- 你可以定义数据的结构，然后使用特殊生成的源代码轻松的在各种数据流中使用各种语言进行编写 和读取结构数据。你甚至可以更新数据结构，而不破坏由旧数据结构编译的已部署程序。

  > 什么是 Google Protocol Buffer？ 假如您在网上搜索，应该会得到类似这样的文字介绍：
  >
  > Google Protocol Buffer( 简称 Protobuf) 是 Google 公司内部的混合语言数据标准，目前已 经正在使用的有超过 48,162 种报文格式定义和超过 12,183 个 .proto 文件。他们用于 RPC 系统和持续数据存储系统。
  >
  > Protocol Buffers 是一种轻便高效的结构化数据存储格式，可以用于结构化数据串行化，或 者说序列化。它很适合做数据存储或 RPC 数据交换格式。可用于通讯协议、数据存储等领域 的语言无关、平台无关、可扩展的序列化结构数据格式。目前提供了 C++、Java、Python 三 种语言的 API。



### 安装

- 上传解压

  - ```shell
    [root@node01 ~]# tar -zxvf protobuf-2.5.0.tar.gz
    [root@node01 ~]# mkdir -p /opt/bdp/protobuf-2.5.0
    ```

- 配置安装

  - ```shell
    [root@node01 ~]# yum install gcc-c++ -y
    [root@node01 ~]# cd protobuf-2.5.0
    [root@node01 ~]# ./configure --prefix=/opt/bdp/protobuf-2.5.0
    [root@node01 ~]# make && make install
    ```



### 使用

- 流程

  - 先编写protobuf语言代码，设置压缩的字段和范围 
  - 执行bin下的proto程序，将protobuf的程序编译为java程序 
  - 使用hbaseapi的时候（加载或者读取），使用protobuf里边的方法

- protobuf代码实现

  - window本地新建一个文件，命名为 PhoneRecordProtos.proto，新增以下内容

  - ```java
    package com.xxxx.util;
    
    option java_outer_classname = "PhoneRecordProtos";
    
    //创建一个消息对象
    message PhoneRecord
    {
    required string otherphone = 1;
    optional int32 time = 2;
    optional int64 date = 3;
    optional string type = 4;
    }
    //每日消息
    message PhoneRecordDay
    {
    repeated PhoneRecord phoneRecord=1;
    }
    //每月消息
    message PhoneRecordMonth
    {
    repeated PhoneRecordDay phoneRecordDay=1;
    }
    ```

- 上传并执行

  - 上传脚本文件PhoneRecord.proto到/root目录下 

  - cd到protobuf的bin目录下

  - ```shell
    # cd /opt/bdp/protobuf-2.5.0/bin
    # ./protoc --java_out=/root --proto_path=/root/
    /root/PhoneRecordProtos.proto
    ```

  - 将编译后的获取到的java文件get到本地，然后将java类，放到项目里边，最后使用protobuf 加载数据和查询数据。

  - ```java
    package com.xxxx.jdbc;
    
    import com.xxxx.util.PhoneRecordProtos;
    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.hbase.HBaseConfiguration;
    import org.apache.hadoop.hbase.TableName;
    import org.apache.hadoop.hbase.client.*;
    import org.apache.hadoop.hbase.util.Bytes;
    import org.junit.After;
    import org.junit.Before;
    import org.junit.Test;
    
    import java.io.IOException;
    import java.util.Date;
    import java.util.List;
    
    public class Hello03PhoneRecordProtoBuf {
        
        //获取数据库连接
        private Connection connection;
        //获取HBase管理员类
        private Admin admin;
        //获取表信息
        private Table table;
        //声明表的名字
        private final static String TABLE_NAME = "t_phone";
        //声明列簇的名字
        private final static String COLUMN_FAMILY_NAME = "info";
        
        
    	@Test
    	public void gets() throws IOException {
            Get get = new Get("18001128288_202003010000000000".getBytes());
            Result result = table.get(get);
            
    		byte[] value = result.getValue(COLUMN_FAMILY_NAME.getBytes(),"day".getBytes());
            
            PhoneRecordProtos.PhoneRecordDay phoneRecordDay = PhoneRecordProtos.PhoneRecordDay.parseFrom(value);
            
            List<PhoneRecordProtos.PhoneRecord> phoneDetailList = phoneRecordDay.getPhoneRecordList();
            
            for (PhoneRecordProtos.PhoneRecord phoneRecord : phoneDetailList) {
                long date = phoneRecord.getDate();
                String otherphone = phoneRecord.getOtherphone();
                int time = phoneRecord.getTime();
    
                System.out.println(otherphone + " - " + time + " - " + date);
    		}
    	}
        
    	@Test
    	public void puts() throws IOException, InterruptedException {
    		//创建一个插入对象
            Put put = new Put("18001128288_202003010000000000".getBytes());
            //插入100条通话记录
            PhoneRecordProtos.PhoneRecordDay.Builder phoneRecordDay = PhoneRecordProtos.PhoneRecordDay.newBuilder();
            //开始批量生成数据
    		for (int i = 100; i < 200; i++) {
                //创建每一条数据对象
                PhoneRecordProtos.PhoneRecord.Builder phoneRecord =
                PhoneRecordProtos.PhoneRecord.newBuilder();
                phoneRecord.setOtherphone("13812345" + i);
                phoneRecord.setTime(i);
                phoneRecord.setDate(new Date().getTime());
                phoneRecord.setType(String.valueOf(i % 2));
                //将对象添加到每日记录
    			phoneRecordDay.addPhoneRecord(phoneRecord);
    		}
    		//将数据写出
    		put.addColumn(Bytes.toBytes(COLUMN_FAMILY_NAME),Bytes.toBytes("day"), phoneRecordDay.build().toByteArray());
            //插入数据
            table.put(put);
            table.close();
    	}
        
        @Before
        public void init() throws IOException {
            //获取配置文件
            Configuration configuration = HBaseConfiguration.create();
            //创建数据库连接
            this.connection = ConnectionFactory.createConnection(configuration);
            this.admin = connection.getAdmin();
            //获取数据库表信息
            this.table = this.connection.getTable(TableName.valueOf(TABLE_NAME));
        }
        
        @After
        public void destory() throws IOException {
            if (table != null) {
            	table.close();
            }
            if (admin != null) {
            	admin.close();
            }
            if (connection != null) {
            	connection.close();
            }
    	}
    }
    ```



### 性能

Total Time 指一个对象操作的整个时间，包括创建对象，将对象序列化为内存中的字节序列，然后再反 序列化的整个过程。从测试结果可以看到 Protobuf 的成绩很好，感兴趣的可以自行到网站 [http://code. google.com/p/thrift-protobuf-compare/wiki/Benchmarking](http://code. google.com/p/thrift-protobuf-compare/wiki/Benchmarking)上了解更详细的测试结果。

**Protobuf 的优点**

Protobuf 有如 XML，不过它更小、更快、也更简单。你可以定义自己的数据结构，然后使用代码生成 器生成的代码来读写这个数据结构。你甚至可以在无需重新部署程序的情况下更新数据结构。只需使用 Protobuf 对数据结构进行一次描述，即可利用各种不同语言或从各种不同数据流中对你的结构化数据轻 松读写。

它有一个非常棒的特性，即“向后”兼容性好，人们不必破坏已部署的、依靠“老”数据格式的程序就可以对 数据结构进行升级。这样您的程序就可以不必担心因为消息结构的改变而造成的大规模的代码重构或者 迁移的问题。因为添加新的消息中的 field 并不会引起已经发布的程序的任何改变。

Protobuf 语义更清晰，无需类似 XML 解析器的东西（因为 Protobuf 编译器会将 .proto 文件编译生成 对应的数据访问类以对 Protobuf 数据进行序列化、反序列化操作）。

使用 Protobuf 无需学习复杂的文档对象模型，Protobuf 的编程模式比较友好，简单易学，同时它拥有 良好的文档和示例，对于喜欢简单事物的人们而言，Protobuf 比其他的技术更加有吸引力。

**Protobuf 的不足**

Protbuf 与 XML 相比也有不足之处。它功能简单，无法用来表示复杂的概念。

XML 已经成为多种行业标准的编写工具，Protobuf 只是 Google 公司内部使用的工具，在通用性上还差 很多。

由于文本并不适合用来描述数据结构，所以 Protobuf 也不适合用来对基于文本的标记文档（如 HTML）建模。另外，由于 XML 具有某种程度上的自解释性，它可以被人直接读取编辑，在这一点上 Protobuf 不行，它以二进制的方式存储，除非你有 .proto 定义，否则你没法直接读出 Protobuf 的任何内容









