+++
title = "lettuce-redis--Could-not-get-a-resource-from-the-pool"
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

[link to JianShu](https://www.jianshu.com/p/f998c57071ec)这篇估计因为提到了VPN，至今依然是“仅自己可见”

使用VPN链接公司内网的环境。本地调试链接Redis时是不是会提示：完整log

```
 org.springframework.data.redis.connection.PoolException: Could not get a resource from the pool; nested exception is io.lettuce.core.RedisException: Cannot retrieve initial cluster partitions from initial URIs [RedisURI [host='test-redis.dns.fxxxktest.com', port=3001], RedisURI [host='test-redis.dns.fxxxktest.com', port=3002], RedisURI [host='test-redis.dns.fxxxktest.com', port=3003]]
	at org.springframework.data.redis.connection.lettuce.LettucePoolingConnectionProvider.getConnection(LettucePoolingConnectionProvider.java:87) ~[spring-data-redis-2.1.10.RELEASE.jar:2.1.10.RELEASE]
	at org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory$SharedConnection.getNativeConnection(LettuceConnectionFactory.java:1104) ~[spring-data-redis-2.1.10.RELEASE.jar:2.1.10.RELEASE]
	at org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory$SharedConnection.getConnection(LettuceConnectionFactory.java:1085) ~[spring-data-redis-2.1.10.RELEASE.jar:2.1.10.RELEASE]
	at org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory.getClusterConnection(LettuceConnectionFactory.java:363) ~[spring-data-redis-2.1.10.RELEASE.jar:2.1.10.RELEASE]
	at org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory.getConnection(LettuceConnectionFactory.java:333) ~[spring-data-redis-2.1.10.RELEASE.jar:2.1.10.RELEASE]
	at org.springframework.data.redis.core.RedisConnectionUtils.doGetConnection(RedisConnectionUtils.java:132) ~[spring-data-redis-2.1.10.RELEASE.jar:2.1.10.RELEASE]
	at org.springframework.data.redis.core.RedisConnectionUtils.getConnection(RedisConnectionUtils.java:95) ~[spring-data-redis-2.1.10.RELEASE.jar:2.1.10.RELEASE]
	at org.springframework.data.redis.core.RedisConnectionUtils.getConnection(RedisConnectionUtils.java:82) ~[spring-data-redis-2.1.10.RELEASE.jar:2.1.10.RELEASE]
	at org.springframework.data.redis.core.RedisTemplate.execute(RedisTemplate.java:211) ~[spring-data-redis-2.1.10.RELEASE.jar:2.1.10.RELEASE]
	at org.springframework.data.redis.core.RedisTemplate.execute(RedisTemplate.java:184) ~[spring-data-redis-2.1.10.RELEASE.jar:2.1.10.RELEASE]
	at org.springframework.data.redis.core.AbstractOperations.execute(AbstractOperations.java:95) ~[spring-data-redis-2.1.10.RELEASE.jar:2.1.10.RELEASE]
	at org.springframework.data.redis.core.DefaultValueOperations.get(DefaultValueOperations.java:53) ~[spring-data-redis-2.1.10.RELEASE.jar:2.1.10.RELEASE]
	at com.fxxxktest.ConsumerProcess.run(ConsumerProcess.java:126) [classes/:?]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [?:1.8.0_181]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [?:1.8.0_181]
	at java.lang.Thread.run(Thread.java:748) [?:1.8.0_181]
Caused by: io.lettuce.core.RedisException: Cannot retrieve initial cluster partitions from initial URIs [RedisURI [host='test-redis.dns.fxxxktest.com', port=3001], RedisURI [host='test-redis.dns.fxxxktest.com', port=3002], RedisURI [host='test-redis.dns.fxxxktest.com', port=3003]]
	at io.lettuce.core.cluster.RedisClusterClient.doLoadPartitions(RedisClusterClient.java:874) ~[lettuce-core-5.1.8.RELEASE.jar:?]
	at io.lettuce.core.cluster.RedisClusterClient.loadPartitions(RedisClusterClient.java:844) ~[lettuce-core-5.1.8.RELEASE.jar:?]
	at io.lettuce.core.cluster.RedisClusterClient.initializePartitions(RedisClusterClient.java:819) ~[lettuce-core-5.1.8.RELEASE.jar:?]
	at io.lettuce.core.cluster.RedisClusterClient.connect(RedisClusterClient.java:345) ~[lettuce-core-5.1.8.RELEASE.jar:?]
	at org.springframework.data.redis.connection.lettuce.ClusterConnectionProvider.getConnection(ClusterConnectionProvider.java:85) ~[spring-data-redis-2.1.10.RELEASE.jar:2.1.10.RELEASE]
	at org.springframework.data.redis.connection.lettuce.LettucePoolingConnectionProvider.lambda$null$0(LettucePoolingConnectionProvider.java:75) ~[spring-data-redis-2.1.10.RELEASE.jar:2.1.10.RELEASE]
	at io.lettuce.core.support.ConnectionPoolSupport$RedisPooledObjectFactory.create(ConnectionPoolSupport.java:209) ~[lettuce-core-5.1.8.RELEASE.jar:?]
	at io.lettuce.core.support.ConnectionPoolSupport$RedisPooledObjectFactory.create(ConnectionPoolSupport.java:199) ~[lettuce-core-5.1.8.RELEASE.jar:?]
	at org.apache.commons.pool2.BasePooledObjectFactory.makeObject(BasePooledObjectFactory.java:58) ~[commons-pool2-2.6.2.jar:2.6.2]
	at org.apache.commons.pool2.impl.GenericObjectPool.create(GenericObjectPool.java:889) ~[commons-pool2-2.6.2.jar:2.6.2]
	at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:424) ~[commons-pool2-2.6.2.jar:2.6.2]
	at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:349) ~[commons-pool2-2.6.2.jar:2.6.2]
	at io.lettuce.core.support.ConnectionPoolSupport$1.borrowObject(ConnectionPoolSupport.java:122) ~[lettuce-core-5.1.8.RELEASE.jar:?]
	at io.lettuce.core.support.ConnectionPoolSupport$1.borrowObject(ConnectionPoolSupport.java:117) ~[lettuce-core-5.1.8.RELEASE.jar:?]
	at org.springframework.data.redis.connection.lettuce.LettucePoolingConnectionProvider.getConnection(LettucePoolingConnectionProvider.java:81) ~[spring-data-redis-2.1.10.RELEASE.jar:2.1.10.RELEASE]
	... 15 more
```

Cannot retrieve initial cluster partitions from initial URIs
…………
……
…
Could not get a resource from the pool 

类似的错误。

相关的配置文件类似——
```
spring:
  redis:
    timeout: 6000ms
    database: 0
    cluster:
      nodes:
        - test-redis.dns.xxx.com:3333
        - test-redis.dns.xxx.com:3222
        - test-redis.dns.xxx.com:3111
      max-redirects: 3 # 获取失败 最大重定向次数
    lettuce:
      pool:
        max-active: 1000  #连接池最大连接数（使用负值表示没有限制）
        max-idle: 10 # 连接池中的最大空闲连接
        min-idle: 5 # 连接池中的最小空闲连接
        max-wait: -1 # 连接池最大阻塞等待时间（使用负值表示没有限制）
```
尝试增大空闲链接的配置和超时时间，暂时解决，后续跟踪。

---

- [Could not get a resource from the pool 错误解决](https://developers-youcong.github.io/2019/03/09/Could-not-get-a-resource-from-the-pool-%E9%94%99%E8%AF%AF%E8%A7%A3%E5%86%B3/)
从这里看，配置是没有问题的。

- 一说法是[版本问题](https://blog.csdn.net/wd2014610/article/details/80568947)，但这个链接配置大部分时是正常的，不是这个情况。

- 这些应该还使用的jdis项目，我的环境是lettuce
[spring整合redis使用RedisTemplate的坑Could not get a resource from the pool](https://www.cnblogs.com/DDgougou/p/10268206.html)这里说：问题的关键在于redis的配置文件中enableTransactionSupport的设置。

有效链接——阿里云redis的常见异常文档
 1、[Jedis常见异常汇总](https://yq.aliyun.com/articles/236384)
 2、[JedisPool资源池优化](https://yq.aliyun.com/articles/236383)

---
[从Redis连接池获取连接失败的原因说起](https://www.jianshu.com/p/bb42f3aa91b1)这篇硬核说明，有空可以研究研究。
结论是
>“由于redis所在服务器磁盘不足导致，由于是测试服务器，也没有配置磁盘的监控。腾出空间后即可恢复。”

