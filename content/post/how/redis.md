+++
title = "Redis Basic"
description = "redis practice"
tags = [
    "redis"
]
date = "2022-02-02"
topics = [
    "redis",
    "tech"
]
draft = false
toc = true
+++

Rdis基础

当调用方发起注册请求时，注册中心会将当前的调用方IP与第一步生成的客户端实例绑定，同时在redis中增加调用方IP信息，将调用方IP信息以及service+method+branch组成的api信息记录到redis中；

### ZADD 

ZADD： 将一个或多个成员元素及其分数值加入到有序集当中   
https://www.runoob.com/redis/sorted-sets-zadd.html  
Zcard 命令用于计算集合中元素的数量。  
ZRANGE myzset 0 -1 WITHSCORES 返回key有序集中满足分数值区间的所有value  

### SADD 

Sadd 命令将一个或多个成员元素加入到集合中(添加成功返回1 )，已经存在于集合的成员元素将被忽略（返回0 ）。  
假如集合 key 不存在，则创建一个只包含添加的元素作成员的集合。  
https://www.runoob.com/redis/sets-sadd.html   

### SMEMBERS 

SMEMBERS 可以查看对应key下的集合中有哪些具体的值 对应于query逻辑  

Srem 命令用于移除集合中的一个或多个成员元素，不存在的成员元素会被忽略。  
https://www.runoob.com/redis/sets-srem.html  

```
#注册时： 添加一个有序集server， key使用时间戳，value为agent的ip地址 AddNode  
ZADD server 1646559335 address  

# checkin操作时，先执行注册动作AddNode，同时将对应ip地址提供的所有api注册 Registe(ctx, req.Address, api.Service, api.Method, api.Branch)。每个api作为key，所属的ip为value

# http://redisdoc.com/script/eval.html 
# 0表示无键名参数； 
# address表示为 ARGV[1] Lua下表从1开始
# apiKey 表示为 ARGV[2]  得到的值为 SADD api:service/method/branch
CallScript(RegisterApiScript, 0, address, core.GenerateApiKey(service, method, branch))

const RegisterApiScript = `
local api_key = "api:"..ARGV[2]
local node_key = "node:"..ARGV[1]
local res = redis.call("SADD", api_key, ARGV[1])
redis.call("SADD", node_key, ARGV[2])
if res == 1 then 
	redis.call("PUBLISH", ARGV[2], "A|"..ARGV[1])
end
return 1
`
如果是新加的api，则进行publish，订阅方将更新api列表

# 删除node： checkout或断开连接？ 
CallScript(UnregisterScript, 0, address)
const UnregisterScript = `
local apis = redis.call("SMEMBERS", "node:"..ARGV[1])
for _, api in ipairs(apis) do
	local res = redis.call("SREM", "api:"..api, ARGV[1])
	if res == 1 then
		redis.call("PUBLISH", api, "D|"..ARGV[1])
	end
end
redis.call("del",  "node:"..ARGV[1])
redis.call("ZREM", "server", ARGV[1])
return 1
`

Zrem 命令用于移除有序集中的一个或多个成员，不存在的成员将被忽略。  
https://www.runoob.com/redis/sorted-sets-zrem.html

```

https://www.runoob.com/redis/redis-pub-sub.html   
当调用方发起订阅请求时，注册中心会将调用方service+method+branch构建的key注册到redis channel中，后续在该channel上发布信息时，能够通知到调用方，这里主要的目的就是监控服务信息的变化；  
当调用方发起取消订阅请求时，注册中心会将调用方service+method+branch构建的key取消redis channel中的订阅。 (某个node不再提供此api的服务)  


```
# 查看有多少server注册
ZRANGE server 0 -1 WITHSCORES
# 对应server有哪些api
SMEMBERS  node:10.19.36.241:9002
# api有哪些node提供服务
SMEMBERS  api:loancommon.LoanList/AuditList/stable

# 订阅 api
SUBSCRIBE loancommon.LoanList/AuditList/stable

```

### LRANGE vs. RPUSH 

Redis Lrange 返回列表中指定区间内的元素，区间以偏移量 START 和 END 指定。 其中 0 表示列表的第一个元素， 1 表示列表的第二个元素，以此类推。 你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。

```
redis 127.0.0.1:6379> LRANGE KEY_NAME START END

redis> RPUSH mylist "one"
```