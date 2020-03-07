+++
title = "STF之ZeroMQ"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "STF"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/029db9da070b)

>STF consists of multiple independent processes communicating via [ZeroMQ](http://zeromq.org/) and [Protocol Buffers](https://github.com/google/protobuf).

使用ZeroMQ进行zmq socket通信，使用Protocol Buffers定义通信内容。

[官方指南](http://zguide.zeromq.org) ，正式权威官方详细版本，啃下这个剩下的都可以忽略了。 
[官方指南中文版本](https://github.com/anjuke/zguide-cn) 官方版本的中文版本，针对ZMQ 2.1版本
[另外一个中文版本](https://www.cnblogs.com/neooelric/p/8978720.html) [另外一个中文版本2](https://www.cnblogs.com/neooelric/p/9020872.html) 个人理解版本，可参考
[example in NodeJS](https://rastating.github.io/using-zeromq-with-node-js/) 实例讲解基础版本

[官方 socket types说明](http://api.zeromq.org/4-0:zmq-socket)
[The Socket Types](https://sachabarbs.wordpress.com/2014/08/21/zeromq-2-the-socket-types-2/) 解释了各个socket的类型含义
[Messaging patterns](https://mytrile.github.io/blog/2010/09/20/zeromq-messaging-patterns/)，简单解释了下列模式的含义

1.  **[Pipeline](http://api.zeromq.org/zmq_socket.html#_pipeline_pattern "Pipeline pattern")** - used for distributing data to nodes arranged in a pipeline
2.  **[Request-Reply](http://api.zeromq.org/zmq_socket.html#_request_reply_pattern "Request-reply pattern")** - used for sending requests from a client to one or more instances of a service, and receiving subsequent replies to each request sent
3.  **[Publish-Subscribe](http://api.zeromq.org/zmq_socket.html#_publish_subscribe_pattern "Publish-subscribe pattern")** - used for sending requests from a client to one or more instances of a service, and receiving subsequent replies to each request sent
4.  **[Exclusive Pair](http://api.zeromq.org/zmq_socket.html#_exclusive_pair_pattern "Exclusive Pair pattern")** - advanced pattern used for communicating exclusively between two peers

[A quick and dirty introduction to ZeroMQ](https://blog.scottlogic.com/2015/03/20/ZeroMQ-Quick-Intro.html) C# JAVA Node配合解释ZeroMQ





