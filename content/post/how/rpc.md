+++
title = "RPC"
description = "Leaning about RPC"
tags = [
    "rpc"
]
date = "2020-12-21"
topics = [
    "rpc",
    "tech"
]
toc=true
+++


推荐[深入浅出RPC原理](https://ketao1989.github.io/2016/12/10/rpc-theory-in-action/#%E9%80%9A%E7%94%A8rpc%E6%9E%B6%E6%9E%84) 

[grpc：](https://github.com/grpc/grpc/blob/master/doc/service_config.md)


RMI: Remote Method Invocation，远程方法调用 ——RPC界的原型机

![RMI](https://upload-images.jianshu.io/upload_images/3296949-68c8a9173348f901.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>**stub(桩)：** stub实际上就是远程过程在客户端上面的一个代理proxy。当我们的客户端代码调用API接口提供的方法的时候，RMI生成的stub代码块会将请求数据序列化，交给远程服务端处理，然后将结果反序列化之后返回给客户端的代码。这些处理过程，对于客户端来说，基本是透明无感知的。
>
>**remote：** 这层就是底层网络处理了，RMI对用户来说，屏蔽了这层细节。stub通过remote来和远程服务端进行通信。
>
>**skeleton(骨架)：** 和stub相似，skeleton则是服务端生成的一个代理proxy。当客户端通过stub发送请求到服务端，则交给skeleton来处理，其会根据指定的服务方法来反序列化请求，然后调用具体方法执行，最后将结果返回给客户端。
>
>**registry(服务发现)：** rmi服务，在服务端实现之后需要注册到rmi server上，然后客户端从指定的rmi地址上lookup服务，调用该服务对应的方法即可完成远程方法调用。registry是个很重要的功能，当服务端开发完服务之后，要对外暴露，如果没有服务注册，则客户端是无从调用的，即使服务端的服务就在那里。

![](https://upload-images.jianshu.io/upload_images/3296949-4fd6050485200f02.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>**serviceClient：** 这个模块主要是封装服务端对外提供的API，让客户端像使用本地API接口一样调用远程服务。一般，我们使用动态代理机制，当客户端调用api的方法时，serviceClient会走代理逻辑，去远程服务器请求真正的执行方法，然后将响应结果作为本地的api方法执行结果返回给客户端应用。类似RMI的stub模块。
>
>**processor：** 在服务端存在很多方法，当客户端请求过来，服务端需要定位到具体对象的具体方法，然后执行该方法，这个功能就由processor模块来完成。一般这个操作需要使用反射机制来获取用来执行真实处理逻辑的方法，当然，有的RPC直接在server初始化的时候，将一定规则写进Map映射中，这样直接获取对象即可。类似RMI的skeleton模块。
>
>**protocol：** 协议层，这是每个RPC组件的核心技术所在。一般，协议层包括编码/解码，或者说序列化和反序列化工作；当然，有的时候编解码不仅仅是对象序列化的工作，还有一些通信相关的字节流的额外解析部分。序列化工具有：hessian，protobuf，avro,thrift，json系，xml系等等。在RMI中这块是直接使用JDK自身的序列化组件。
>
>**transport：** 传输层，主要是服务端和客户端网络通信相关的功能。这里和下面的IO层区分开，主要是因为传输层处理server/client的网络通信交互，而不涉及具体底层处理连接请求和响应相关的逻辑。
>
>**I/O：** 这个模块主要是为了提高性能可能采用不同的IO模型和线程模型，当然，一般我们可能和上面的transport层联系的比较紧密，统一称为remote模块。

生产级别的RPC实例——美团[OCTO Dorado通信框架介绍](https://github.com/Meituan-Dianping/octo-rpc/wiki/OCTO-Dorado%E9%80%9A%E4%BF%A1%E6%A1%86%E6%9E%B6%E4%BB%8B%E7%BB%8D)

![](https://upload-images.jianshu.io/upload_images/3296949-a5b8f904d051b180.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
[svg原图](https://raw.githubusercontent.com/Meituan-Dianping/octo-rpc/master/dorado/dorado-doc/img/FrameworkDesign.svg)

导入项目后，在demo模块下可以直接进行验证，例如执行 `ThriftProvider `和`ThriftConsumer`的main方法。就完成了一次完整的RPC调用。其中，使用测试的ZooKeeperServerMain作为注册中心；发布HelloService；远程调用此服务。

使用xml定义的方式参考[手册](https://github.com/Meituan-Dianping/octo-rpc/blob/master/dorado/dorado-doc/Manual.md)


在windows环境下，注意本地网络环境。本地包含多个虚拟网卡时，查找的服务地址有问题，导致执行失败——

发布服务时，注册的信息包括`info.setIp(NetUtil.getLocalHost());`其中com.meituan.dorado.common.util.NetUtil的`getLocalHost()`方法在获取本地IP时，只排除了回送地址

```
2020/12/21 14:46:18.982 NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2200 [INFO] NIOServerCnxnFactory (NIOServerCnxnFactory.java:215) Accepted socket connection from /127.0.0.1:6543
2020/12/21 14:46:18.991 NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2200 [INFO] ZooKeeperServer (ZooKeeperServer.java:948) Client attempting to establish new session at /127.0.0.1:6543
2020/12/21 14:46:19.022 SyncThread:0 [INFO] ZooKeeperServer (ZooKeeperServer.java:693) Established session 0x1001fd7146f0001 with negotiated timeout 6000 for client /127.0.0.1:6543
2020/12/21 14:46:20.462 ProcessThread(sid:0 cport:2200): [INFO] PrepRequestProcessor (PrepRequestProcessor.java:487) Processed session termination for sessionid: 0x1001fd7146f0001
2020/12/21 14:46:20.486 NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2200 [INFO] NIOServerCnxn (NIOServerCnxn.java:1040) Closed socket connection for client /127.0.0.1:6543 which had sessionid 0x1001fd7146f0001
2020/12/21 14:46:24.087 DoradoServerBizWorker-com.meituan.dorado.test.thrift.api.HelloService-pool-1-thread-10 [ERROR] ProviderChannelHandler (ProviderChannelHandler.java:137) Provider do method invoke failed, TransportException:Failed to send message DefaultResponse, cause: Channel[id: 0x994cd231, L:192.168.194.1/192.168.194.1:9001 ! R:/192.168.194.1:6592] is not connected.
com.meituan.dorado.common.exception.TransportException: Failed to send message DefaultResponse, cause: Channel[id: 0x994cd231, L:192.168.194.1/192.168.194.1:9001 ! R:/192.168.194.1:6592] is not connected.
	at com.meituan.dorado.transport.AbstractChannel.send(AbstractChannel.java:37) ~[classes/:?]
	at com.meituan.dorado.transport.support.ProviderChannelHandler.send(ProviderChannelHandler.java:161) ~[classes/:?]
	at com.meituan.dorado.transport.support.ProviderChannelHandler$1.run(ProviderChannelHandler.java:134) [classes/:?]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [?:1.8.0_181]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [?:1.8.0_181]
	at java.lang.Thread.run(Thread.java:748) [?:1.8.0_181]
2020/12/21 14:46:25.490 DoradoServerNettyWorkerGroup-5-1 [ERROR] ProviderChannelHandler (ProviderChannelHandler.java:277) ExceptionCaught(client IP:192.168.194.1)
java.io.IOException: 远程主机强迫关闭了一个现有的连接。
	at sun.nio.ch.SocketDispatcher.read0(Native Method) ~[?:1.8.0_181]
	at sun.nio.ch.SocketDispatcher.read(SocketDispatcher.java:43) ~[?:1.8.0_181]
	at sun.nio.ch.IOUtil.readIntoNativeBuffer(IOUtil.java:223) ~[?:1.8.0_181]
	at sun.nio.ch.IOUtil.read(IOUtil.java:192) ~[?:1.8.0_181]
	at sun.nio.ch.SocketChannelImpl.read(SocketChannelImpl.java:380) ~[?:1.8.0_181]
	at io.netty.buffer.PooledUnsafeDirectByteBuf.setBytes(PooledUnsafeDirectByteBuf.java:288) ~[netty-all-4.1.25.Final.jar:4.1.25.Final]
	at io.netty.buffer.AbstractByteBuf.writeBytes(AbstractByteBuf.java:1108) ~[netty-all-4.1.25.Final.jar:4.1.25.Final]
	at io.netty.channel.socket.nio.NioSocketChannel.doReadBytes(NioSocketChannel.java:345) ~[netty-all-4.1.25.Final.jar:4.1.25.Final]
	at io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:148) [netty-all-4.1.25.Final.jar:4.1.25.Final]
	at io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:647) [netty-all-4.1.25.Final.jar:4.1.25.Final]
	at io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:582) [netty-all-4.1.25.Final.jar:4.1.25.Final]
	at io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:499) [netty-all-4.1.25.Final.jar:4.1.25.Final]
	at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:461) [netty-all-4.1.25.Final.jar:4.1.25.Final]
	at io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:884) [netty-common-4.1.25.Final.jar:4.1.25.Final]
	at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30) [netty-common-4.1.25.Final.jar:4.1.25.Final]
	at java.lang.Thread.run(Thread.java:748) [?:1.8.0_181]
```

调用端报错类似——

```
2020/12/21 14:46:19.293 main [INFO] ZookeeperDiscoveryService (ZookeeperDiscoveryService.java:79) Subscribe on zookeeper[127.0.0.1:2200]: remoteAppkey=com.meituan.octo.dorado.server, serviceName=com.meituan.dorado.test.thrift.api.HelloService
2020/12/21 14:46:19.293 main [INFO] RegistryPolicy (RegistryPolicy.java:72) Subscribe SubscribeInfo:{localAppkey=com.meituan.octo.dorado.client,remoteAppkey=com.meituan.octo.dorado.server,serviceName=com.meituan.dorado.test.thrift.api.HelloService,protocol=thrift,serialize=thriftCodec,env=test,attachments={registryWay=zookeeper, registryAddress=127.0.0.1:2200}} by com.meituan.dorado.registry.zookeeper.ZookeeperDiscoveryService
com.meituan.dorado.common.exception.TimeoutException: GetRequest timeout, timeout:1000, interface=com.meituan.dorado.test.thrift.api.HelloService$Iface|method=sayHello|provider=192.168.194.1:9001
	at com.meituan.dorado.rpc.handler.invoker.AbstractInvoker$1.handle(AbstractInvoker.java:134)
	at com.meituan.dorado.rpc.handler.filter.AbstractTraceFilter.filter(AbstractTraceFilter.java:38)
	at com.meituan.dorado.rpc.handler.filter.InvokeChainBuilder$1.handle(InvokeChainBuilder.java:81)
	at com.meituan.dorado.rpc.handler.invoker.AbstractInvoker.invoke(AbstractInvoker.java:71)
	at com.meituan.dorado.cluster.tolerance.FailfastClusterHandler.doInvoke(FailfastClusterHandler.java:41)
	at com.meituan.dorado.cluster.ClusterHandler.handle(ClusterHandler.java:59)
	at com.meituan.dorado.rpc.proxy.DefaultInvocationHandler.invoke(DefaultInvocationHandler.java:58)
	at com.sun.proxy.$Proxy20.sayHello(Unknown Source)
	at com.meituan.dorado.demo.thrift.apiway.ThriftConsumer.main(ThriftConsumer.java:35)
2020/12/21 14:46:20.458 main [INFO] ZookeeperDiscoveryService (ZookeeperDiscoveryService.java:94) Unsubscribe on zookeeper[127.0.0.1:2200]: remoteAppkey=com.meituan.octo.dorado.server, serviceName=com.meituan.dorado.test.thrift.api.HelloService
2020/12/21 14:46:20.459 main [INFO] RegistryPolicy (RegistryPolicy.java:79) Unsubscribe SubscribeInfo:{localAppkey=com.meituan.octo.dorado.client,remoteAppkey=com.meituan.octo.dorado.server,serviceName=com.meituan.dorado.test.thrift.api.HelloService,protocol=thrift,serialize=thriftCodec,env=test,attachments={registryWay=zookeeper, registryAddress=127.0.0.1:2200}} by com.meituan.dorado.registry.zookeeper.ZookeeperDiscoveryService
2020/12/21 14:46:20.459 Curator-Framework-0 [INFO] CuratorFrameworkImpl (CuratorFrameworkImpl.java:937) backgroundOperationsLoop exiting
2020/12/21 14:46:20.485 main-EventThread [INFO] ClientCnxn (ClientCnxn.java:521) EventThread shut down for session: 0x1001fd7146f0001
2020/12/21 14:46:20.485 main [INFO] ZooKeeper (ZooKeeper.java:687) Session: 0x1001fd7146f0001 closed
```

组件的默认实现已经[扩展](https://github.com/Meituan-Dianping/octo-rpc/blob/master/dorado/dorado-doc/manual-developer/Extension.md)

### 打包Dorado

根据[Dorado打包说明](https://github.com/Meituan-Dianping/octo-rpc/blob/master/dorado/dorado-doc/manual-developer/Compile.md)

直接打包，执行后在octo-rpc/dorado/dorado-build/target/ 下可以找到dorado的jar

在octo-rpc/dorado 目录下执行`mvn clean package -Dmaven.test.skip=true`，提示—— 找不到 `com.meituan.octo:mns-invoker:jar:1.0.0`组件

```
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  15.915 s
[INFO] Finished at: 2020-12-22T10:28:13+08:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal on project dorado: Could not resolve dependencies for project com.meituan.octo:dorado:jar:1.0.0: Failure to find com.meituan.octo:mns-invoker:jar:1.0.0 in https://repo.maven.apache.org/maven2 was cached in the local r
epository, resolution will not be reattempted until the update interval of central has elapsed or updates are forced -> [Help 1]
[ERROR]
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
```

>请到[OCTO-NS](https://github.com/Meituan-Dianping/octo-ns/blob/master/mns-invoker/README.md)获取依赖mns-invoker依赖

文档错误，上面的模块`mns-invoker`目录现在已更改到`https://github.com/Meituan-Dianping/octo-ns/tree/master/sdk/mns-invoker` 下载octo-ns模块后，编译 `mns-invoker`模块，提示缺少 `com.meituan.octo:idl-common:jar:1.0.0 `

```
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  6.412 s
[INFO] Finished at: 2020-12-22T10:37:34+08:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal on project mns-invoker: Could not resolve dependencies for project com.meituan.octo:mns-invoker:jar:1.0.0: Could not find artifact com.meituan.octo:idl-common:jar:1.0.0 in central (https://repo.maven.apache.org/maven2) -> [Help 1]
```

先编译 `idl-comman`模块：需要本地有thrift执行环境，并在pom文件中指定执行文件的目录。

安装thrift参考 [官方安装说明](https://thrift.apache.org/docs/install/)，
```
<dependencies>
	<dependency>
		<groupId>org.apache.thrift</groupId>
		<artifactId>libthrift</artifactId>
		<!-- 建议与你生成源码的Thrift版本保持一致! -->
		<version>0.9.3</version>
	</dependency>
</dependencies>
...
...

<plugin>
	<groupId>org.apache.thrift.tools</groupId>
	<artifactId>maven-thrift-plugin</artifactId>
	<version>0.1.11</version>
	<configuration>
		<!-- your thrift path -->
		<thriftExecutable>D:/tools/thrift/thrift-0.9.3.exe</thriftExecutable>
	</configuration>
	<executions>
		<execution>
			<id>thrift-sources</id>
			<phase>generate-sources</phase>
			<goals>
				<goal>compile</goal>
			</goals>
		</execution>
	</executions>
</plugin>
```
目前thrift的最新版本为 `0.13.0`，直接使用这个版本，会提示错误——`unknown option java:hashcode` 似乎是个[兼容性问题](https://github.com/pauldeschacht/impala-java-client/issues/6)，降级之后编译成功

```
[ERROR] thrift failed output: [WARNING:D:/openSources/gitee/octo-ns/common/idl-mns/idl-common/src/main/thrift/unified_protocol.thrift:35] The "byte" type is a compatibility alias for "i8". Use "i8" to emphasize the signedness of this type.
[ERROR] thrift failed error: [FAILURE:generation:1] Error: unknown option java:hashcode
```

到[存档地址](http://archive.apache.org/dist/thrift/)下载`0.9.3`版本，编译成功。

- 编译安装 `idl-common`模块
- 编译安装 `mns-invoder`模块
- 编译打包`Dorado`工程， 在`dorado-build`模块的target目录下生成dorado的jar包

```
[INFO] dorado-registry-mns 1.0.0 .......................... SUCCESS [  0.295 s]
[INFO] dorado-trace 1.0.0 ................................. SUCCESS [  0.030 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  19.953 s
[INFO] Finished at: 2020-12-22T11:25:26+08:00
[INFO] ------------------------------------------------------------------------
```
