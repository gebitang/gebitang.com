+++
title = "skywalking practice"
description = "the way to skywalking"
tags = [
    "skywalking",
    "tech"
]
date = "2021-11-13"
topics = [
    "skywalking",
    "tech"
]
draft = false
toc = true
+++

[SkyWalking 极简入门](https://skywalking.apache.org/zh/2020-04-19-skywalking-quick-start/)收录了[个人站点的搭建说明](https://www.iocoder.cn/Elasticsearch/install) 

- 搭建ES服务
- 搭建SkyWalkingOAP服务
- 搭建SkyWalking UI服务
- 测试SkyWalking Agent


### 搭建ES服务

[ES服务手动搭建](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html)，类似SonarQube服务，本地搭建很容易，下载对应的[版本](https://www.elastic.co/downloads/elasticsearch)，查看版本[兼容性](https://www.elastic.co/support/matrix)。

```shell
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.15.2-linux-x86_64.tar.gz
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.15.2-linux-x86_64.tar.gz.sha512
shasum -a 512 -c elasticsearch-7.15.2-linux-x86_64.tar.gz.sha512 
tar -xzf elasticsearch-7.15.2-linux-x86_64.tar.gz
cd elasticsearch-7.15.2/  #这是ES_HOME的地址

# 启动
./bin/elasticsearch
# 指定进程启动
./bin/elasticsearch  -d -p pid  # pid for file name, actual pid number is the content of the file
# 检查启动效果，本地访问
curl -X GET "localhost:9200/?pretty"

{
  "name" : "guazistf",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "f6zoFv1QTNmG9uSaplNTIg",
  "version" : {
    "number" : "7.15.2",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "93d5a7f6192e8a1a12e154a2b81bf6fa7309da0c",
    "build_date" : "2021-11-04T14:04:42.515624022Z",
    "build_snapshot" : false,
    "lucene_version" : "8.9.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}

#配置 $ES_HOME/config/elasticsearch.yml 
#通过命令配置
./bin/elasticsearch -d -Ecluster.name=my_cluster -Enode.name=node_1
```

### 搭建SkyWalking服务

#### OAP服务

都是Java项目，相同的套路。下载对应的压缩包，解压后，使用脚本启动，通常默认的配置(具体配置在`config/application.yml`)至少可以保证启动成功。

```shell 
$./bin/oapService.sh
SkyWalking OAP started successfully!
```

#### UI服务

```shell
bin/webappService.sh
SkyWalking Web Application started successfully!
```

查看对应的log信息`logs/logs/webapp.log`，配置文件`webapp/webapp.yml`通常只需要修改下端口

#### agent服务

```shell
# SkyWalking Agent 配置
export SW_AGENT_NAME=demo-application # 配置 Agent 名字。一般来说，我们直接使用 Spring Boot 项目的 `spring.application.name` 。
export SW_AGENT_COLLECTOR_BACKEND_SERVICES=127.0.0.1:11800 # 配置 Collector 地址 相当于gRPCPort的值
export SW_AGENT_SPAN_LIMIT=2000 # 配置链路的最大 Span 数量。一般情况下，不需要配置，默认为 300 。主要考虑，有些新上 SkyWalking Agent 的项目，代码可能比较糟糕。
export JAVA_AGENT=-javaagent:/home/stf/es/apache-skywalking-apm-bin/agent/skywalking-agent.jar # SkyWalking Agent jar 地址。

# Jar 启动
java -jar $JAVA_AGENT -jar lab-39-demo-2.2.2.RELEASE.jar
```

`服务(Service) --> 服务实例(ServiceInstance) --> 端点(EndPoint)`