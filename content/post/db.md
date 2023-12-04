+++
title = "db"
description = "about database"
tags = [
    "sql",
    "db"
]
date = "2021-07-21"
topics = [
    "sql"
]
toc = true
draft = true
+++

SQL: Structured Query Language  
NoSQL: not only SQL (键值、文档、列存储、图)  
NewSQL: combine SQL and NoSQL ??  

[SQL vs NoSQL vs NewSQL](https://www.xenonstack.com/blog/sql-vs-nosql-vs-newsql)

**ACID**  Atomicity, Consistency, Isolation, Durability to maintain the reliability of transactions.   
>**Atomicity** – completion of the transaction as a whole or none at all   
>**Consistency** – assures the stable state of the database with or without changes  
>**Isolation** – multiple transactions do not interfere with each other  
>**Durability** – permanent effect on the database by the changes  

[oltp vs. olap](https://www.guru99.com/oltp-vs-olap.html)

## lealone 


[基础操作](https://github.com/lealone/Lealone/issues/165)

```sql
DROP TABLE IF EXISTS test;

CREATE TABLE IF NOT EXISTS test (f1 int primary key, f2 long);

CREATE TABLE IF NOT EXISTS test (f1 int, f2 long);


INSERT INTO test(f1, f2) VALUES(1, 1);

INSERT INTO test(f1, f2) VALUES(1, 1);

INSERT INTO test(f1, f2) VALUES(3,3);


ALTER USER root SET password 'xxx';

sqlshell.sh -user root -password 'xxx'

```


### 创建IDE Debug环境

- git clone下代码导入IDEA
- 在`lealone-main`模块下增加resources文件夹，将`lealone-dist`模块下conf文件夹的内容复制到resources下（这一步为了识别log4j的配置文件，可以调整log文件位置）
- 需要设置`lealone.config`系统变量，可以通过`System.setProperty("lealone.config", "file:///D:/path/to/resources/lealone.yaml");`完成（可以参考`lealone-dist`模块下bin文件夹的脚本内容查看需要的信息）
- 启动debug模式，配合打包编译后的`sqlshell`作为客户端。完成debug交互

第一步，修改root用户的密码。默认用户为root，密码为空。连接进入后，执行`SET PASSWORD 'your-new-password'`就可以完成管理员账号的密码设置


## How does a relational database work

原始文章： [How does a relational database work](http://coding-geek.com/how-databases-work/?print=print) 

英文版本的[book样式](https://walleipt.gitbooks.io/how-does-a-relational-database-work/content/)

中文版本备份：关于数据库原理的总结[【一】](https://zhuanlan.zhihu.com/p/48629562) [【二】](https://zhuanlan.zhihu.com/p/48633080) 

本文主要介绍核心组件以及其中应用到的算法以及使用的数据结构：

- 基础
  - 时间复杂度(决定了后续不同场景下需要采用何种算法，以及由于条件限制——时间，CPU，内存等——采用哪种工程实现)
  - 合并排序(分治法)
  - 数据结构：数组，B+树，Hash表
- 数据库组件总览(参考下图)，然后重点介绍了三个组件——
  - Client Manager
  - Query Manager
  - Data Manager

![](https://s3-img.meituan.net/v1/mss_3d027b52ec5a4d589e68050845611e68/ff/n0/0m/kg/8c_288742.jpg@596w_1l.jpg)

更通用性的框架介绍参考《数据库系统基础教程(A first course in database systems)》中提到的组成

![](https://s3-img.meituan.net/v1/mss_3d027b52ec5a4d589e68050845611e68/ff/n0/0m/kg/7n_289069.jpg@596w_1l.jpg)

- 单框表示不同的组件
- 双框表示内存中的数据结构
- 实线表示控制和数据流
- 虚线只表示数据流

文中提到的[Architecture of a Database System ](http://db.cs.berkeley.edu/papers/fntdb07-architecture.pdf)研究论文提到的组件构成——


![](https://s3-img.meituan.net/v1/mss_3d027b52ec5a4d589e68050845611e68/ff/n0/0m/kg/8v_288731.jpg@596w_1l.jpg)

[H2 DB Architecture](https://h2database.com/html/architecture.html) Top-down Overview

- JDBC driver `org.h2.jdbc, org.h2.jdbcx`
- Connection/session management `org.h2.engine.Database, org.h2.engine.SessionInterface, org.h2.engine.Session, org.h2.engine.SessionRemote`
- SQL Parser `org.h2.command.Parser`
- Command execution and planning `org.h2.command.ddl, org.h2.command.dml`
- Table/Index/Constraints `org.h2.table, org.h2.index`
- Undo log, redo log, and transactions layer 
- B-tree engine and page-based storage allocation `org.h2.store`
- Filesystem abstraction `org.h2.store.FileStore`

## OLAP VS. OLTP

### OLAP
**Online Analytical Processing**, a category of software tools which provide analysis of data for business decisions. OLAP systems allow users to analyze database information from multiple database systems at one time.

**The primary objective is data analysis and not data processing**.

### OLTP
**Online Transaction Processing** shortly known as OLTP supports transaction-oriented applications in a 3-tier architecture. OLTP administers day to day transaction of an organization.

**The primary objective is data processing and not data analysis**

![](https://upload-images.jianshu.io/upload_images/3296949-667b7d275ee82eba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以认为任何一个数据仓库(Datawarehouse)都是OLAP的。数据库存储的数据用来进行“分析”（Analyze）的。关注数据生成的“事后”分析(Analytical)。

OLTP则强调“事务”（transaction）本身，关注在“业务”进行中的逻辑。例如——在线预订火车票；银行的账单处理。

>ATM center. Assume that a couple has a joint account with a bank. One day both simultaneously reach different ATM centers at precisely the same time and want to withdraw total amount present in their bank account.

## Database VS. Schema 

[Difference between Database and Schema](https://www.javatpoint.com/database-vs-schema) 

这里的区分是针对`database.schema.table`这种情况下的区分来说的。

- database是“实体的”，schema是“表现的”。
- 前者用来存储结构化的数据；后者描述了数据库的表现形式
- 前者使用DML`data manipulation language`来操作数据；后者使用DDL`Data Definition Language`描述数据库的表现形式

只是在Mysql中，这两个概念不做区分[`In MySQL, physically, a schema is synonymous with a database.`](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_schema)

可以参考PostgresSQL中的[PostgreSQL Schema](https://www.javatpoint.com/postgresql-tutorial)，实现了上述的区别。

这样是在一个数据库中，可以创建不同的schema，然后不同的schema下面可以包含相同的table名称。只是有一个数据库连接就可以为不同用户提供相同的服务


## Lealone basic

[用户文档 快速入门](https://github.com/lealone/Lealone-Docs/blob/master/%E5%BA%94%E7%94%A8%E6%96%87%E6%A1%A3/%E7%94%A8%E6%88%B7%E6%96%87%E6%A1%A3.md)

安装这个说明进行编译。

下载源码之后，执行 `mvn clean package assembly:assembly -Dmaven.test.skip=true` 


遇到的几个小问题：

- 1. ~~Maven编译需要暂时去掉 `<module>lealone-test</module>`模块，否则会出现测试脚本失败的情况~~
这是因为我在Powershell环境下执行` mvn clean package assembly:assembly -Dmaven.test.skip=true`这个目录导致的。[参考](https://stackoverflow.com/a/6351739/1087122)这里，可以看到实际上跳过test的命令并没有生效。需要使用`mvn clean package assembly:assembly '-Dmaven.test.skip=true'`的方式执行才会生效。

效果还是很直接的—— [原文地址](https://www.jianshu.com/p/54e816217e5f)

![server 启动](https://upload-images.jianshu.io/upload_images/3296949-969f1fe012b279e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![命令行执行](https://upload-images.jianshu.io/upload_images/3296949-47e28bebab8e205e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![java main方法执行](https://upload-images.jianshu.io/upload_images/3296949-f4cc8a6aa7732925.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 2. 在linux下执行时注意Java -cp 指定classpath时才有`:`分割path的路径。同时需要设置环境变量`JAVA_HOME`。

手残，修改环境变量时，最后少打了一个H，`export PATH=${JAVA_HOME}/bin:$PATH`变成了`export PATH=${JAVA_HOME}/bin:$PAT`导致所以的命令无法使用。

正常的解决方案：

> 使用绝对路径进行操作；`/usr/bin/vi /home/geb/.zshrc`进行编辑  
>临时导出新的PATH路径`export PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin`然后继续操作

## Presto

[Presto is (OLAP) and is Not (OLTP)](https://prestosql.io/docs/current/overview/use-cases.html)

>Presto is a high performance, distributed SQL query engine for big data. Its architecture allows users to query a variety of data sources such as Hadoop, AWS S3, Alluxio, MySQL, Cassandra, Kafka, and MongoDB. One can even query data from multiple data sources within a single query. Presto is community driven open-source software released under the Apache License.

- 高性能SQL查询引擎
- 支持的数据源包括：Hadoop Distributed File System, AWS S3, Alluxio, MySQL, PostgreSQL, Microsoft SQL Server, Amazon Redshift, Apache Kudu, Apache Phoenix, Apache Kafka, Apache Cassandra, Apache Accumulo, MongoDB and Redis等，[完整列表](https://prestosql.io/docs/current/connector.html)，目前支持30个不同数据来源
- 最初由Facebook设计开发，用来查询分析内部存储在Hadoop上的数据，(数据太大，嫌弃Hive太慢）
- 2012年开发使用；2013年开源；2014年Netflix应用presto来查询存储在AWS S3上的10PB级别的数据
- 2019年1月成立Presto Software Foundation以继续开发Presto引擎；PrestoDB(presto的FB内部版本)依然由Facebook开发
- 2019年9月，FB将PrestoDB贡献给Linux基金会，建立Presto Foundation

| Site | Foundation | Source | 
| --- | --- | --- | 
|https://prestosql.io|Presto Software Foundation|https://github.com/prestosql/presto|
|https://prestodb.io|Presto Foundation|https://github.com/prestodb/presto|

- prestosql.io是正常的企业开发，开源，社区驱动
- prestdb.io是[大企业联合](https://www.linuxfoundation.org/press-release/2019/09/facebook-uber-twitter-and-alibaba-form-presto-foundation-to-tackle-distributed-data-processing-at-scale/)
- [#issue380 in prestosql.io: What is the relationship of prestosql and prestodb?](https://github.com/prestosql/presto/issues/380)
- [Presto：分布式 SQL 查询引擎](https://yuzhouwan.com/posts/200906)

>Presto 采用 MPP（Massively Parallel Processing）大规模并行处理架构，来解决大量数据分析的场景。该架构的主要特征，如下：
>
>- 任务并行执行  
>- 分布式计算
>- Shared Nothing
>- 横向扩展
>- 数据分布式存储（本地化）

>Presto 采用 SPI（Service Provider Interface）服务提供发现机制，来插件化地支持多种数据源，以实现联邦查询（Federation Query，指能够通过一条 SQL 查询，对处于完全不同的系统中的不同的数据库和模式，进行引用和使用）

### data function

官方文档：[8.13. Date and Time Functions and Operators](https://prestodb.io/docs/current/functions/datetime.html) 

The `date_trunc` function supports the following units:

|Unit|Example Truncated Value|
|---------|---------|
|second|2001-08-22 03:04:05.000|
|minute|2001-08-22 03:04:00.000|
|hour|2001-08-22 03:00:00.000|
|day|2001-08-22 00:00:00.000|
|week|2001-08-20 00:00:00.000|
|month|2001-08-01 00:00:00.000|
|quarter|2001-07-01 00:00:00.000|
|year|2001-01-01 00:00:00.000|

The above examples use the timestamp `2001-08-22 03:04:05.321` as the input.

使用实例：[DATE_TRUNC: A SQL Timestamp Function You Can Count On](https://mode.com/blog/date-trunc-sql-timestamp-function-count-on/) 

`date_format(timestamp, format)` → `varchar`. Formats timestamp as a string using format. 
`from_unixtime(unixtime)` → timestamp. Returns the UNIX timestamp unixtime as a timestamp. 

上面两个方法组合使用，得到——
[How to convert milliseconds or seconds into date format in Presto?](http://evafengeva.blogspot.com/2017/09/how-to-convert-milliseconds-or-seconds.html)

Milliseconds:  
`DATE_FORMAT(FROM_UNIXTIME(column_name /1000),'%Y-%m-%d')`
Seconds:  
`DATE_FORMAT(FROM_UNIXTIME(column_name),'%Y-%m-%d')`

Please note that '/1000' should be added when it converts milliseconds to human-readable format.

```sql
select case when wait_duration > 1800000 then '30m以上'
            when wait_duration <= 1800000 and wait_duration > 600000 then '10~30m'
            when wait_duration <= 600000 and wait_duration > 60000 then '1~10m'
            else '1m以下'
           end as wsec, count(1) AS num, substr(created_at,1,10) as date
FROM
  service_su_result_day
WHERE dt = '${date_y_m_d}'
-- https://dba.stackexchange.com/a/106798
GROUP BY substr(created_at,1,10),case when wait_duration > 1800000 then '30m以上'
            when wait_duration <= 1800000 and wait_duration > 600000 then '10~30m'
            when wait_duration <= 600000 and wait_duration > 60000 then '1~10m'
            else '1m以下'
           end
```