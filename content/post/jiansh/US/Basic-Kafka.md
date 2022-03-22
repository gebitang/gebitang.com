+++
title = "Basic-Kafka"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "US"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/fbaaa7fe7f62)

[https://kafka.apache.org/quickstart](https://kafka.apache.org/quickstart)

- Java8 + Windows环境 
- Zookeeper环境

### 启动Server时，会提示`java.lang.NumberFormatException: For input string: "initial.rebalance.delay.ms"`错误，根据[这里](https://stackoverflow.com/a/57513323/1087122)的提示，在window环境下执行：`.\bin\windows\kafka-server-start.bat .\config\server.properties` 

可以正常启动——这里启动的还是kafka的server，用来连接zookeeper的server，需要先启动zk。否则会超时自动结束。
```
PS D:\tools\kafka_2.12-2.3.0> .\bin\windows\kafka-server-start.bat .\config\server.properties

[2019-11-19 11:24:32,075] INFO Registered kafka:type=kafka.Log4jController MBean (kafka.utils.Log4jControllerRegistration$)
[2019-11-19 11:24:32,429] INFO starting (kafka.server.KafkaServer)
[2019-11-19 11:24:32,430] INFO Connecting to zookeeper on localhost:2181 (kafka.server.KafkaServer)
[2019-11-19 11:24:32,446] INFO [ZooKeeperClient Kafka server] Initializing a new session to localhost:2181. (kafka.zookeeper.ZooKeeperClient) [2019-11-19 11:24:32,453] INFO Client environment:zookeeper.version=3.4.14-4c25d480e66aadd371de8bd2fd8da255ac140bcf, built on 03/06/2019 16:18 GMT (org.apache.zookeeper.ZooKeeper)
[2019-11-19 11:24:32,453] INFO Client environment:host.name=Gebitang (org.apache.zookeeper.ZooKeeper)
...
[2019-11-19 11:24:40,813] ERROR Fatal error during KafkaServer startup. Prepare to shutdown (kafka.server.KafkaServer)
kafka.zookeeper.ZooKeeperClientTimeoutException: Timed out waiting for connection while in state: CONNECTING
        at kafka.zookeeper.ZooKeeperClient.$anonfun$waitUntilConnected$3(ZooKeeperClient.scala:258)
        at scala.runtime.java8.JFunction0$mcV$sp.apply(JFunction0$mcV$sp.java:23)
        at kafka.utils.CoreUtils$.inLock(CoreUtils.scala:253)
        at kafka.zookeeper.ZooKeeperClient.waitUntilConnected(ZooKeeperClient.scala:254)
        at kafka.zookeeper.ZooKeeperClient.<init>(ZooKeeperClient.scala:112)
        at kafka.zk.KafkaZkClient$.apply(KafkaZkClient.scala:1826)
        at kafka.server.KafkaServer.createZkClient$1(KafkaServer.scala:364)
        at kafka.server.KafkaServer.initZkClient(KafkaServer.scala:387)
        at kafka.server.KafkaServer.startup(KafkaServer.scala:207)
        at kafka.server.KafkaServerStartable.startup(KafkaServerStartable.scala:38)
        at kafka.Kafka$.main(Kafka.scala:84)
        at kafka.Kafka.main(Kafka.scala)
[2019-11-19 11:24:40,818] INFO shutting down (kafka.server.KafkaServer)
[2019-11-19 11:24:40,822] INFO shut down completed (kafka.server.KafkaServer)
[2019-11-19 11:24:40,823] ERROR Exiting Kafka. (kafka.server.KafkaServerStartable)
[2019-11-19 11:24:40,825] INFO shutting down (kafka.server.KafkaServer)
```

[Windows环境搭建参考](https://dzone.com/articles/running-apache-kafka-on-windows-os) 有错误部分，以下文为准。

[https://zookeeper.apache.org/](https://zookeeper.apache.org/) -->[https://zookeeper.apache.org/doc/current/index.html](https://zookeeper.apache.org/doc/current/index.html)
-->[https://zookeeper.apache.org/doc/current/zookeeperStarted.html](https://zookeeper.apache.org/doc/current/zookeeperStarted.html)

### 提示`找不到或无法加载主类 org.apache.zookeeper.server.quorum.quorumpeermain`

使用下载的bin压缩包，例如`apache-zookeeper-3.5.6-bin.tar.gz `而不是 `apache-zookeeper-3.5.6.tar.gz `，后者是源码，非编译后的bin包。

### 提示`无法创建数据文件 

修改的conf文件里的目录地址使用`/`分割，不要使用window默认的`\`格式，后者导致路径错误，作为转移符号处理了

```
2019-11-19 12:00:44,800 [myid:] - ERROR [main:ZooKeeperServerMain@75] - Unable to access datadir, exiting abnormally
org.apache.zookeeper.server.persistence.FileTxnSnapLog$DatadirException: Unable to create data directory D:     oolsapache-zookeeper-3.5.6data\version-2
        at org.apache.zookeeper.server.persistence.FileTxnSnapLog.<init>(FileTxnSnapLog.java:115)
        at org.apache.zookeeper.server.ZooKeeperServerMain.runFromConfig(ZooKeeperServerMain.java:124)
        at org.apache.zookeeper.server.ZooKeeperServerMain.initializeAndRun(ZooKeeperServerMain.java:106)
        at org.apache.zookeeper.server.ZooKeeperServerMain.main(ZooKeeperServerMain.java:64)
        at org.apache.zookeeper.server.quorum.QuorumPeerMain.initializeAndRun(QuorumPeerMain.java:128)
        at org.apache.zookeeper.server.quorum.QuorumPeerMain.main(QuorumPeerMain.java:82)
Unable to access datadir, exiting abnormally
```

---
这样，环境搭建算OK了。

1. 创建topic `.\kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test`

```
PS D:\tools\kafka_2.12-2.3.0\bin\windows> .\kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
Created topic test.
PS D:\tools\kafka_2.12-2.3.0\bin\windows> .\kafka-topics.bat --list --zookeeper localhost:2181
test
```

2. 创建producer和consumer发送消息

```
# producer
 .\kafka-console-producer.bat --broker-list localhost:9092 --topic test

# consumer
.\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test --from-beginning
```

###[总结]

[安装zookeeper：](https://zookeeper.apache.org/doc/current/zookeeperStarted.html)

- 下载bin压缩包，配置环境变量
- 修改config文件 `zoo.cfg`
- 启动server `zkserver`

使用kafka

1. 启动zkserver  `zkserver` （独立安装zookeeper之后）
2. 启动kafka broker ` .\bin\windows\kafka-server-start.bat .\config\server.properties `
3. 启动producer和consumer

### 获取topic的所有consumer信息

[参考](https://stackoverflow.com/a/55938325/1087122)，或者这里[Kafka获取订阅某topic的所有consumer group](https://www.cnblogs.com/huxi2b/p/10638008.html)

```java
Properties props = ...//here you put your properties
AdminClient kafkaClient = AdminClient.create(props);

//Here you get all the consumer groups
List<String> groupIds = kafkaClient.listConsumerGroups().all().get().
                       stream().map(s -> s.groupId()).collect(Collectors.toList()); 

//Here you get all the descriptions for the groups
Map<String, ConsumerGroupDescription> groups = kafkaClient.
                                               describeConsumerGroups(groupIds).all().get();
for (final String groupId : groupIds) {
    ConsumerGroupDescription descr = groups.get(groupId);
    //find if any description is connected to the topic with topicName
    Optional<TopicPartition> tp = descr.members().stream().
                                  map(s -> s.assignment().topicPartitions()).
                                  flatMap(coll -> coll.stream()).
                                  filter(s -> s.topic().equals(topicName)).findAny();
            if (tp.isPresent()) {
                //you found the consumer, so collect the group id somewhere
            }
} 
```

### header信息会被保存

[源码 @param headers the headers that will be included in the record](https://github.com/apache/kafka/blob/trunk/clients/src/main/java/org/apache/kafka/clients/producer/ProducerRecord.java#L67)

```java
 /**
     * Creates a record with a specified timestamp to be sent to a specified topic and partition
     * 
     * @param topic The topic the record will be appended to
     * @param partition The partition to which the record should be sent
     * @param timestamp The timestamp of the record, in milliseconds since epoch. If null, the producer will assign
     *                  the timestamp using System.currentTimeMillis().
     * @param key The key that will be included in the record
     * @param value The record contents
     * @param headers the headers that will be included in the record
     */
    public ProducerRecord(String topic, Integer partition, Long timestamp, K key, V value, Iterable<Header> headers) {
        if (topic == null)
            throw new IllegalArgumentException("Topic cannot be null.");
        if (timestamp != null && timestamp < 0)
            throw new IllegalArgumentException(
                    String.format("Invalid timestamp: %d. Timestamp should always be non-negative or null.", timestamp));
        if (partition != null && partition < 0)
            throw new IllegalArgumentException(
                    String.format("Invalid partition: %d. Partition number should always be non-negative or null.", partition));
        this.topic = topic;
        this.partition = partition;
        this.key = key;
        this.value = value;
        this.timestamp = timestamp;
        this.headers = new RecordHeaders(headers);
    }
```