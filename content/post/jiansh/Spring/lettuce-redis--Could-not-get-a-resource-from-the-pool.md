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

使用VPN链接公司内网的环境。本地调试链接Redis时是不是会提示：

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

