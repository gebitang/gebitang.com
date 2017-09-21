+++
title = "Socket Exception Handler"
description = "deal with socket exception"
tags = [
    "socket"
]
date = "2017-09-21"
topics = [
    "socket", "work"
]
toc = true
+++

SocketException异常

[参考原文](http://developer.51cto.com/art/201003/189724.htm)

<!--more-->

## Exception Types

### 1. java.net.BindException:Address already in use: JVM_Bind

>>该异常发生在服务器端进行new ServerSocket(port)（port是一个0，65536的整型值）操作时。异常的原因是以为与port一样的一个端口已经被启动，并进行监听。

>>此时用netstat –an命令，可以看到一个Listending状态的端口。只需要找一个没有被占用的端口就能解决这个问题。

### 2. java.net.SocketException: Connection refused: connect

>>该异常发生在客户端进行 new Socket(ip, port)操作时，该异常发生的原因是或者具有ip地址的机器不能找到（也就是说从当前机器不存在到指定ip路由），或者是该ip存在，但找不到指定的端口进行监听。

>>出现该问题，首先检查客户端的ip和port是否写错了，如果正确则从客户端ping一下服务器看是否能ping通，如果能ping通（服务服务器端把ping禁掉则需要另外的办法），则看在服务器端的监听指定端口的程序是否启动，这个肯定能解决这个问题。

### 3. java.net.SocketException: Socket is closed

>>该异常在客户端和服务器均可能发生。异常的原因是己方主动关闭了连接后（调用了Socket的close方法）再对网络连接进行读写操作

### 4. java.net.SocketException: Connection reset / Connect reset by peer / Socket write error

>>该异常在客户端和服务器端均有可能发生，引起该异常的原因有两个，第一个就是如果一端的Socket被关闭（或主动关闭或者因为异常退出而引起的关闭），另一端仍发送数据，发送的第一个数据包引发该异常(Connect reset by peer)。另一个是一端退出，但退出时并未关闭该连接，另一端如果在从连接中读数据则抛出该异常（Connection reset）。简单的说就是在连接断开后的读和写操作引起的。

### 5. java.net.SocketException: Broken pipe

>>该异常在客户端和服务器均有可能发生。在第4个异常的第一种情况中（也就是抛出 SocketExcepton:Connect reset by peer:Socket write error后），如果再继续写数据则抛出该异常

>>前两个异常的解决方法是首先确保程序退出前关闭所有的网络连接，其次是要检测对方的关闭连接操作，发现对方关闭连接后自己也要关闭该连接。


## 需要注意的问题

### 1. 正确区分长、短连接

所谓的长连接是一经建立就永久保持。短连接就是在以下场景下，准备数据—>建立连接— >发送数据—>关闭连接。

### 2. 对长连接的维护

包括两个方面，首先是检测对方的主动断连（既调用 Socket的close方法），其次是检测对方的宕机、异常退出及网络不通。这是一个健壮的通信程序必须具备的。检测对方的主动断连很简单，主要一方主动断连，另一方如果在进行读操作，则此时的返回值只-1，一旦检测到对方断连，则应该主动关闭己方的连接（调用Socket的close方法）

检测对方的宕机、异常退出及网络不通常用方法是用“心跳”，也就是双方周期性的发送数据给对方，同时也从对方接收“心跳”，如果连续几个周期都没有收到对方心跳，则可以判断对方或者宕机或者异常推出或者网络不通，此时也需要主动关闭己方连接，如果是客户端可在延迟一定时间后重新发起连接。虽然Socket有一个keep alive选项来维护连接，如果用该选项，一般需要两个小时才能发现对方的宕机、异常退出及网络不通

### 3. 处理效率问题

不管是客户端还是服务器，如果是长连接一个程序至少需要两个线程，一个用于接收数据，一个用于发送心跳，写数据不需要专门的线程，当然另外还需要一类线程（俗称Worker线程）用于进行消息的处理，也就是说接收线程仅仅负责接收数据，然后再分发给Worker进行数据的处理。如果是短连接，则不需要发送心跳的线程，如果是服务器还需要一个专门的线程负责进行连接请求的监听。 
