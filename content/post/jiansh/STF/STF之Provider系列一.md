+++
title = "STF之Provider系列一"
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



[link on JianShu](https://www.jianshu.com/p/8984575174b4)

独立服务-未完成 

目前local方式已经跑起来，如果进行分布式接入设备？官方提供都都是[docker 方式](https://github.com/openstf/stf/blob/master/doc/DEPLOYMENT.md)

>Provider role
The provider role requires the following units, which must be together on a single or more hosts.
[adbd.service](https://github.com/openstf/stf/blob/master/doc/DEPLOYMENT.md#adbdservice)
[stf-provider@.service](https://github.com/openstf/stf/blob/master/doc/DEPLOYMENT.md#stf-providerservice)

参考adbd服务的[Dockerfile](https://github.com/sorccu/docker-adb/blob/master/Dockerfile)，只是执行一个adb 命令而已。本地验证后，不清楚为什么设备一直认为是offline状态，实际`adb devices`时又是可以看到设备在线但。 


```
$ adb -a -P 5037 server nodaemon
adb I 07-17 11:27:27  1977 854167 usb_osx.cpp:308] reported max packet size for 7bf567ed is 512
adb I 07-17 11:27:27  1977 854164 auth.cpp:421] adb_auth_init...
adb I 07-17 11:27:27  1977 854164 auth.cpp:174] read_key_file '/Users/gebitang/.android/adbkey'...
adb I 07-17 11:27:27  1977 854170 transport.cpp:281] 7bf567ed: read thread spawning
adb I 07-17 11:27:27  1977 854171 transport.cpp:294] 7bf567ed: write thread spawning
adb I 07-17 11:27:27  1977 854164 auth.cpp:473] Calling send_auth_response
adb I 07-17 11:27:27  1977 854164 adb.cpp:145] 7bf567ed: offline
```

中文资料介绍也比较少，找到但这些——
[设备提供端分布式部署（adb 方式）](http://mylinuxom.blogspot.com/2018/03/stf-adb.html)
[STF 二次开发之办公机接入 STF 服务](https://testerhome.com/topics/11819)
[ [centos7][stf] 环境搭建](https://testerhome.com/topics/11055)

官方也有人最近提了这个issue——[Deploy stf service on public network. ](https://github.com/openstf/stf/issues/1071)——这个跟我目前但场景是一样的。

对比了`stf local`和独立启动 stf provider的方式。参考 local启动时provider启动的参数信息，修改后可独立启动这个服务。

```
2019-07-17T08:19:53.779Z INF/provider 6147 [*] Subscribing to permanent channel "Iz4tupotSRWjx/rLJF2HsQ=="
2019-07-17T08:19:53.799Z INF/provider 6147 [*] Sending output to "tcp://10.112.78.174:7116"
2019-07-17T08:19:53.800Z INF/provider 6147 [*] Receiving input from "tcp://10.112.78.174:7114"
2019-07-17T08:19:53.806Z INF/provider 6147 [*] Tracking devices
2019-07-17T08:19:53.807Z INF/provider 6147 [*] Found device "7bf567ed" (device)
2019-07-17T08:20:03.814Z INF/provider 6147 [*] Providing all 0 of 1 device(s); ignoring "7bf567ed"
```

如果上面的启动方式是正确的，关键点是 register的异步调用没有进入到注册环节。似乎是收到zmp的register消息后才会调用。这块逻辑还是没有理清楚- -
