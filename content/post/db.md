+++
title = "db"
description = "about database"
tags = [
    "sql",
    "db"
]
date = "2020-11-26"
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

## Lealone basic

[用户文档 快速入门](https://github.com/lealone/Lealone-Docs/blob/master/%E5%BA%94%E7%94%A8%E6%96%87%E6%A1%A3/%E7%94%A8%E6%88%B7%E6%96%87%E6%A1%A3.md)

安装这个说明进行编译。

下载源码之后，执行 `mvn clean package assembly:assembly -Dmaven.test.skip=true` 


遇到的几个小问题：

- 1. ~~Maven编译需要暂时去掉 `<module>lealone-test</module>`模块，否则会出现测试脚本失败的情况~~
这是因为我在Powershell环境下执行` mvn clean package assembly:assembly -Dmaven.test.skip=true`这个目录导致的。[参考](https://stackoverflow.com/a/6351739/1087122)这里，可以看到实际上跳过test的命令并没有生效。需要使用`mvn clean package assembly:assembly '-Dmaven.test.skip=true'`的方式执行才会生效。

效果还是很直接的——

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

