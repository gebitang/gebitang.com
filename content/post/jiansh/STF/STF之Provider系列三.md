+++
title = "STF之Provider系列三"
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



[link on JianShu](https://www.jianshu.com/p/c956e0ca5163)

### websocket的连接问题~~还没有~~解决。 

昨天出现的现象是连接设备时，图像连接到了“其他”设备上。 ——今天定位到是因为独立到provider提供设备时，分配到ws连接地址与也有设备重复。

并没有一个沟通机制决定当前provider到设备端口应该如何分配，需要指定起始大小
```
# 根据已有设备数量进行调整min-port到指，每个设备占据4个端口
--min-port 7416 --max-port 7700 
```
这样可以保证分配的端口是正确的——
```
# r.db('stf').table('devices').withFields('serial', 'display','manufacturer') 
{

    "display": {
        "density": 3 ,
        "fps": 60.000003814697266 ,
        "height": 1920 ,
        "id": 0 ,
        "rotation": 0 ,
        "secure": true ,
        "size": 5.147105693817139 ,
        "url": "ws://test.stf.com:7416" ,
        "width": 1080 ,
        "xdpi": 428.625 ,
        "ydpi": 427.78900146484375
    } ,
    "manufacturer": "XIAOMI" ,
    "serial": "7bf567ed"

}
{

    "display": {
        "density": 2.75 ,
        "fps": 60.000003814697266 ,
        "height": 1920 ,
        "id": 0 ,
        "rotation": 0 ,
        "secure": true ,
        "size": 6.416727542877197 ,
        "url": "ws://test.stf.com:7408" ,
        "width": 1080 ,
        "xdpi": 342.8999938964844 ,
        "ydpi": 343.4360046386719
    } ,
    "manufacturer": "XIAOMI" ,
    "serial": "5cc1d6ae"

} 
```

但这样独立但provider提供的设备出现 无法建立websock连接的问题： `Firefox can’t establish a connection to the server at ws://test.stf.com:7416/.` 

这个端口实际上是远程的provider设备提供的服务；对应的主站上当然没有这个服务。

不经过转发显然是无法连接。STF是如何处理这个场景的？还是我哪里没有设置正确？

这块相关的代码一直还没定位到- -||

----
provider的启动参数里到 --public-ip 针对的是 [当前provider的ip](https://github.com/openstf/stf/issues/84#issuecomment-141104579)

> even though the option is called --public-ip, it actually means "public within your network" (i.e. an internal IP all of your users are able to access).

