# 引入

官方文档： https://clickhouse.com/docs/zh

视频： https://www.bilibili.com/video/BV1xg411w7AP?p=7&spm_id_from=pageDriver&vd_source=6cd527c3a43bcb0943d3d64a7923b3bc





数据库的演化

```
单机

一主多从

多主多从 （一致性问题）

分表分库（业务层面上）

 	1. 水平拆分
 	2. 垂直拆分

针对TP,PB级别的数据，传统的mysql力不从心， 尤其是以数据分析这种典型的“全盘”扫描的统计业务
大规模扫盘已经让mysql不堪重负，为了解决这类问题，从数据应用场景角度分为OLTP,  OLAP
```





什么是OLTP?  **(支持事务)**

- 全称 OnLine Transaction Processing，联机事务处理系统,就是对数据的增删改查等操作
- 存储的是业务数据，来记录某类业务事件的发生，比如下单、支付、注册等等
- 典型代表有Mysql、 Oracle等数据库，对应的网站、系统应用后端数据库
- 针对事务进行操作，对响应时间要求高，面向前台应用的，应用比较简单，数据量相对较少，是GB级别的
- 面向群体:业务人员

当数据积累到一定的程度，需要对过去发生的事情做一个总结分析时，就需要把过去一段时间内产生的数据拿出来进行统计分析，从中获取想要的信息，为公司做决策提供支持，这个就是做OLAP了





什么是OLAP? （分析性能更好）

- 全称OnLine Analytical Processing，联机分析处理系统
- 存储的是历史数据，对应的风控平台、BI平台、数据可视化
- OLAP是数据仓库系统的主要应用，支持复杂的分析操作，侧重决策，并且提供直观易懂的查询结果
- 典型代表有 Hive、ClickHouse
- 针对基于查询的分析系统，基础数据来源于生产系统中的操作数据，数据量非常大，常规是TB级别的
- 面向群体:分析决策人员





Clickhouse 等 OLAP 数据库通常用于回答诸如"昨天有多少人访问掘金？昨天有多少人访问 CSDN"之类的业务问题。如果使用传统的 OLTP 数据库来处理，可能需要几分钟甚至几个小时。而如果使用 OLAP 数据库，我们几毫秒就得到结果。OLTP 和 OLAP 之间的巨大速度差异是因为使用的底层存储结构不同，OLTP 数据库通常使用**行式存储**，OLAP 则通常使用**列式存储**。



为了进行多维分析 OLAP分为三类

- ROLAP， 关系型，数据没有进行预处理，效率非常低， 因为要进行全表扫描，全表扫描使用因为要检索很多字段，不可能每个字段都建立索引, 缺点效率非常低，优点是比较实时。
    - 代表有SparkSQL,  hive

- MOLAP， 预先进行聚合结果，以空间换时间，但是这样需要的空间太多，因为数据会跟着聚合字段（维度）的增加而爆炸性增长，缺点存储空间要很多， 且需要提前预处理。 优点是查询时速度比较快
    - 依托mapreduce， spark

- HOLAP， 混合ROLAP和MOLAP





到底为什么我们需要 Clickhouse?

参考文档： https://juejin.cn/post/7158458151786774559

为了兼顾ROLAP和MOLAP， 就引入了clickhouse ? 

他是列式存储的 OLAP 数据库， 目标是处理数万亿行的数据，并快速执行分析查询



## 列式存储

mysql 是按行进行存储，将一行数据存储到B+数的叶子节点上。

clickhouse 存储数据的方式是按照列来存储 ， 将来自不同列的数据进行单独存储。 

clickhouse 存储一个表数据时，就是以一列为一个文件进行存储





## 选型

为什么mysql 不能被clickhouse 替代

因为mysql是支持事务的，强调数据的完整性， 是强一致性的， 修改数据和删除数据性能好

clickhouse的有点是列式存储，不支持事务，擅长快速的批量查询分析， 修改数据和删除数据性能差， 不能连表查询

因为clickhouse增删改查一行数据， 不同的列要去不同的文件进行增删改查， 比如5个字段要去5个文件中找

做数据分析都是针对列来找， 所以特别适用于clickhouse， 这样就不用一行行地查数据了





# ClickHouse初识

字段很多的情况下，性能依然很好



1. 不支持事务

2. 不擅长根据主键按行粒度查询

3. 不擅长按行修改和删除数据





## 安装

### docker安装

```shell
docker pull clickhouse/clickhouse-server

# The container exposes port 8123 for the HTTP interface and port 9000 for the native client.
docker run -d -p 18123:8123 -p19000:9000 \
-v /opt/data/clickhouse/ch_data:/var/lib/clickhouse/ \
-v /opt/data/clickhouse/ch_logs:/var/log/clickhouse-server/ \
--name some-clickhouse-server --ulimit nofile=262144:262144 clickhouse/clickhouse-server
```



可以用过http接口进行查看

```shell
# 会下载一个文件，结果存放在文件中
http://192.168.0.111:18123/?query=show%20databases
```



### java客户端驱动

官方demo 参考地址

https://github.com/ClickHouse/clickhouse-java/tree/main/examples/client/src/main/java/com/clickhouse/examples/jdbc

第三方

```
<dependency>
    <groupId>cc.blynk.clickhouse</groupId>
    <artifactId>clickhouse4j</artifactId>
    <version>1.4.4</version>
</dependency>
```





### tar包安装

下载地址： https://packages.clickhouse.com/tgz/stable/

版本 21.9.6.24

需要四个包

1. server
2. common-static-dgb
3. common-statc
4. client

解压这四个包后执行安装脚本， 执行doinst.sh

doinst.sh 脚本会创建 clickhouse Linux用户 

doinst.sh 会提示设置server密码



重要的几个目录

```shell
# 服务器配置目录
/etc/clickhouse-server
# 服务器日志目录
/var/log/clickhouse-server/
# 服务器数据目录， 运行时所有的数据文件都放在这里
/var/lib/clickhouse/
/var/lib/clickhouse/data
# 20200601_1_1_0表示按时间进行分区，然后1_1_0表示，开始的块号是1， 结束的块号是1， 第0次合并分区（理解为版本）
/var/lib/clickhouse/data/test/t_stock/20200601_1_1_0
/var/lib/clickhouse/metadata  # 通过sql文件存表结构
```

对外提供连接，需要去/etc/clickhouse-server修改config.xml， 反注释掉listen_host





## DBeaver客户端

dbeaver.io





## 基本命令

```sql
# 和mysql基本一样
# 创建数据库
# 创建表（两种方法，一种是直接创建， 另一种是复制已存在的表）


create database test;


show create database test;


use test;

-- 定义一个列x,  1表示hello, 2表示world
create table t_enum(x Enum8('hello'=1, 'world'=2)) Engine=TinyLog;

drop table t_enum;

-- 插入三行数据
insert into t_enum values ('hello'),('world'),('hello');

insert into t_enum values (1),(2),(2);

-- 原子性， 如果一行数据插入失败， 则所有插入的数据都失败
insert into t_enum values ('hello'),('world'),('hi');


select CAST(x, 'Int8') from t_enum;

select * from t_enum;

select x from t_enum;

create table t_stock(
	id UInt32,
	sku_id String,
	total_amount Decimal(16,2),
	create_time Datetime
)engine=MergeTree()
partition by toYYYYMMDD(create_time)
primary key (id)
order by (id,sku_id);

-- 为什么order by 是必须的？
-- 因为保证了数据的有序，才能进行快速访问

-- 主键，不要求具有唯一性
-- 先通过order by找数据，然后再根据主键进行索引

-- index granularity 索引粒度， 默认8193个
-- 稀疏索引 每隔 8193 个数据建立一次索引

-- 按时间进行分区， 每个区又有多个块
insert into t_stock values
(101, 'sku_002', 2000.00, '2020-06-01 11:00:00'),
(102, 'sku_004', 2500.00, '2020-06-01 12:00:00'),
(103, 'sku_002', 2000.00, '2020-06-02 13:00:00'),
(104, 'sku_002', 12000.00, '2020-06-03 13:00:00'),
(105, 'sku_002', 600.00, '2020-06-04 12:00:00');

select * from t_stock;

-- 分区合并
optimize table t_stock final;

-- 建立二级索引
create table t_stock2(
	id UInt32,
	sku_id String,
	total_amount Decimal(16,2),
	create_time Datetime,
    INDEX secondIndex total_amount TYPE minmax GRANULARITY 5
)engine=MergeTree()
partition by toYYYYMMDD(create_time)
primary key (id)
order by (id,sku_id);

--TTL（可以在字段上指定ttl, 也可以在表上指定ttl, 不能用于主键， 必须指定一个时间列）

```



## 引擎

### 数据库引擎

默认的引擎 Atomic (插入一批数据的时候，如果有一个插入失败， 全部都插入失败)

常见的其他引擎有

- MySQL : clickhouse把请求转发给实际的mysql数据库
- SQLite：同上
- PostgreSQL: 同上	
- MaterializedMySQL： clickhouse 作为 mysql的从节点
- MaterializedPostgreSQL
- Lazy 



### 表引擎

官网： https://clickhouse.com/docs/zh/engines/table-engines

表引擎（即表的类型）决定了：

- **数据的存储方式和位置，写到哪里以及从哪里读取数据 (怎么存的)**
- **支持哪些查询以及如何支持。（怎么查的）**
- 并发数据访问。
- **索引的使用（如果存在，怎么用索引）。**
- 是否可以执行多线程请求。
- 数据复制参数



主要四类

- **日志**
    - TinyLog
    - Log
    - StripeLog
- **MergeTree**
    - MergeTree
    - ReplacingMergeTree
    - SummingMergeTree
    - AggregatingMergeTree
    - CollapsingMergeTree
    - VersionedCollapsingMergeTree
    - GraphiteMergeTree
- **集成引擎**
    - Kafka
    - MySQL
    - ODBC
    - JDBC
    - HDFS
- **用于其他特定功能的引擎**
    - Distributed
    - MaterializedView
    - Dictionary
    - Merge
    - File
    - Null
    - Set
    - Join
    - URL
    - View
    - Memory
    - Buffer



#### MergeTree 表引擎



MergeTree适用于高负载任务的最通用和功能最强大的表引擎。这些引擎的共同特点是可以**快速插入数据**并进行后续的后台数据处理。 MergeTree系列引擎支持数据复制（使用[Replicated*](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/replication#table_engines-replication) 的引擎版本），分区和一些其他引擎不支持的其他功能。



MergeTree 是 ClickHouse 数据库中最常用的表引擎之一，它的作用是支持高效的数据插入和查询。MergeTree 引擎将数据按照主键的顺序进行排序和分区，并在每个分区内创建多个数据块，每个数据块包含一定数量的数据行。在数据插入时，MergeTree 引擎将新数据行插入到对应的数据块中，并在数据块达到一定大小时将其刷新到磁盘上。在数据查询时，MergeTree 引擎可以利用数据的排序和分区信息，通过二分查找等算法快速定位数据行，从而实现高效的查询性能。此外，MergeTree 引擎还支持数据合并和删除操作，可以帮助用户轻松地管理数据。总的来说，MergeTree 引擎是 ClickHouse 数据库中非常重要的一个组件，它为用户提供了高效的数据存储和查询功能。

- 快速插入数据 和 查询



clickhouse 的一级索引和二级索引





参考https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree#mergetree

创建表  

```
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1] [TTL expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2] [TTL expr2],
    ...
    INDEX index_name1 expr1 TYPE type1(...) GRANULARITY value1,
    INDEX index_name2 expr2 TYPE type2(...) GRANULARITY value2
) ENGINE = MergeTree()
ORDER BY expr
[PARTITION BY expr]
[PRIMARY KEY expr]
[SAMPLE BY expr]
[TTL expr [DELETE|TO DISK 'xxx'|TO VOLUME 'xxx'], ...]
[SETTINGS name=value, ...]


```

必须要有 Order by expr





#### ReplacingMergeTree

与MergeTree不同的时， 有一个去重的功能（数据合并的时候才进行！） 也就是合并之前，也会有重复的数据





### 虚拟列

查询当中最好不要用虚拟列， 最好提前处理好放入表

select x+y from t_null ;  -- 其中x+y就是虚拟列



## 基本类型

1. 整型
    1. int8, int16 int32 int64
    2. uint8 uint16 uint32 uint64
2. boolean型
3. 浮点型
4. decimal型
5. 字符型
6. 枚举型
    1. enum8
    2. enum16
7. 数组类型
8. 时间类型
    1. Date
    2. DateTime
    3. DateTime64
9. 可为空类型
    1. Nullable















# 未整理



## 三种特殊表

1. 临时表
    1. 只支持memory引擎
    2. 不属于任何一个数据库
    3. 优先级比其他普通表， 工作用的时候比较少
2. 分区表
    1. 什么是数据分区？
        1. 基于列进行纵向切割的
        2. 对于OLAP来说是非常重要的概念，借助数据分区可以跳过不必要的数据目录，从而提升查询性能
        3. 合理地利用分区特性，还可以变相实现数的更新操作，因为数据分区支持删除、昔换和重置操作。假没数据表按照月份分区，那么数据就可以按月份的粒度被替换更新，分区虽好，但不是所有的表引擎都可以使用这项特性，**目前只有合并树(MergeTree)家族系列的表警才支持数据分区,**
    2. 数据分片？
        1. 基于行进行横向切割的
3. 视图





## 分区表



## 数据字典

数字典是ClickHouse是的一种非常简单、实用的存储媒介，它以键值和属性映射的形式定义数据。

字典中的数据会主动或者被动加载到内存（懒加载），并支持动态更新。由于字典数据常驻内存的特性，所以它非常适合保存常量或经常使用的维度表数据，以游免不以要的join查询。

数据字典分为内置与扩展两种形式，顾名思义，内置字典是ClickHouse默认自带的字典，而外部扩展字典是用户通过自定义配置实现的字典

内置字典（基本不用）

外置字典（常用）



### 扩展字典的七种布局类型

1. flat （内容使用数组存储，数据有上限）
2. hashed  (和flat基本相同， 内存使用hash表存储， 无数据上限)
3. range_hashed
4. cache
5. complex_key_hashed
6. complex_key_cache
7. ip_trie





## Distributed 存储引擎



## 一级索引的实现原理



## 数据存储



## 数据标记



## 对于分区，索引，标记和压缩数据的协同总结

