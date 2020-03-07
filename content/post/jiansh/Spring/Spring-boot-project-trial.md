+++
title = "Spring-boot-project-trial"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "Spring"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/d527d6be358d)

…… 

涉及到的问题：

1. Spring boot工程基础
使用bootcase工程完成练习。

2. yaml文件类型的配置使用，可参考[spring yaml](https://www.baeldung.com/spring-yaml) 
默认位置为`/myApplication/src/main/resources/application.yml` 支持多环境配置

3. 使用druid管理数据库连接
参考[官方文档](https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter)
使用依赖工程里的pom依赖。配置优化为了yml文件格式
```
<dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>druid-spring-boot-starter</artifactId>
   <version>1.1.17</version>
</dependency>
```

4. kafka配置以及基本使用
consumer & producer在两个不同的工程下。
使用yml文件指定对应的 property文件名称，
```
kafka:
  topic: q_test_topic
  consumer:
    properties-file: kafka-consumer-test.properties
```

读取property文件构造对象
```
ClassPathResource resource = new ClassPathResource(kafkaPropFile);
// utils来自spring boot --> spring core
Properties onlineConsumerProperties = PropertiesLoaderUtils.loadProperties(resource);
KafkaConsumer kafkaConsumer = new KafkaConsumer(onlineConsumerProperties);
```

5. redis配置使用
Redis官方提供的Redis client部分Java端的包括Jedis和Lettuce。springboot 2.x版本中默认客户端是用lettuce实现。

Jedis
>使用阻塞的I/O，且其方法调用都是同步的，程序流需要等到sockets处理完I/O才能执行，不支持异步。Jedis客户端实例不是线程安全的，所以需要通过连接池来使用Jedis。

Lettuce	
>高级Redis客户端，用于线程安全同步，异步和响应使用，支持集群，Sentinel，管道和编码器。基于Netty框架的事件驱动的通信层，其方法调用是异步的。Lettuce的API是线程安全的，所以可以操作单个Lettuce连接来完成各种操作

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
默认情况下的模板只能支持 RedisTemplate<String,String>，只能存入字符串。目前工程使用的`StringRedisTemplate`正式默认支持的模板，所以只需要提供默认配置即可——
```
spring:
  redis:
    # Redis服务器地址
    host: redis.dns.guazi.com
    # 端口号   
    port: 10037
    # Redis数据库索引，默认为0
    database: 0
    # Redis服务器连接密码（默认为空）
    password:
    # 连接超时时间（毫秒）
    timeout: 1000
    poll:
      # 连接池最大连接数（使用负值表示没有限制）
      max-active: 8
      # 连接池最大阻塞等待时间（使用负值表示没有限制）
      max-wait: -1
      # 连接池最大空闲连接
      max-idle: 8
      # 连接池最小空闲连接
      min-idle: 0
```
指定了cluster的方式——
```
spring:
  redis:
    timeout: 6000ms
    database: 0
    cluster:
      nodes:
        - 10.16.9.191:4567
        - 10.16.9.192:4567
        - 10.16.9.193:4567
      max-redirects: 3 # 获取失败 最大重定向次数
    lettuce:
      pool:
        max-active: 1000  #连接池最大连接数（使用负值表示没有限制）
        max-idle: 10 # 连接池中的最大空闲连接
        min-idle: 5 # 连接池中的最小空闲连接
        max-wait: -1 # 连接池最大阻塞等待时间（使用负值表示没有限制）
```
