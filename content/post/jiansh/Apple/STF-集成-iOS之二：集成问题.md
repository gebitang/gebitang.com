+++
title = "STF-集成-iOS之二：集成问题"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "Apple"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/cfe66350c5fa)

[STF 集成 iOS之一：环境准备](https://www.jianshu.com/p/8cf6bdb5a091)里完成里环境搭建。 实际上出现的错误——是属于模拟器设备的:(

```
Setup had an error TypeError: Cannot read property 'sdk' of null
    at Object.getDeviceInfo (/Users/gebitang/projects/tryout/stf/lib/units/ios-device/support/deviceinfo.js:56:28)
    at solve (/Users/gebitang/projects/tryout/stf/lib/units/ios-device/plugins/util/identity.js:13:33)
  ...
```

正常的真机可以正常地发现。 需要注意的地方是：需要先手动启动WDA服务（不确认是否是这样，测试实践的结果是这样的）

```
xcodebuild -project WebDriverAgent.xcodeproj -scheme WebDriverAgentRunner -destination 'id=fe6bfa205c6298be6dc806571ee715c26cc3933d' test
```

WDA启动后，在使用 `bin/stf local  --wda-path /Users/gebitang/projects/tryout/WebDriverAgent/ --wda-port 8100`的方式启动时就可以正常初始化设备了。 stf的WDA会启动'两次'，会将手动启动的WDA结束掉。

但如果没有手动先启动WDA，则只有一次 “ xcodebuild构建成功”的log输出。 设备一直处于 waiting状态。 

---
在原生的stf上集成iOS时，按照[帖子](https://testerhome.com/topics/19548)里的说明。可以正常完成iOS设备的初始化。

但 访问  http://localhost:7100/#!/devices 这个地址时，后台会报错——
```
Error: Unknown security handler: accessTokenAuth
    at andCheck (/Users/gebitang/projects/project-remote/RemoteDevice/node_modules/swagger-node-runner/fittings/swagger_security.js:47:39)
    at /Users/gebitang/projects/project-remote/RemoteDevice/node_modules/swagger-node-runner/node_modules/async/lib/async.js:356:13
    at async.forEachOf.async.eachOf (/Users/gebitang/projects/project-remote/RemoteDevice/node_modules/swagger-node-runner/node_modules/async/lib/async.js:233:13)
    at _asyncMap (/Users/gebitang/projects/project-remote/RemoteDevice/node_modules/swagger-node-runner/node_modules/async/lib/async.js:355:9)
    at Object.map (/Users/gebitang/projects/project-remote/RemoteDevice/node_modules/swagger-node-runner/node_modules/async/lib/async.js:337:20)
    at orCheck (/Users/gebitang/projects/project-remote/RemoteDevice/node_modules/swagger-node-runner/fittings/swagger_security.js:40:15)
    at /Users/gebitang/projects/project-remote/RemoteDevice/node_modules/swagger-node-runner/node_modules/async/lib/async.js:356:13
    at async.forEachOf.async.eachOf (/Users/gebitang/projects/project-remote/RemoteDevice/node_modules/swagger-node-runner/node_modules/async/lib/async.js:233:13)
    at _asyncMap (/Users/gebitang/projects/project-remote/RemoteDevice/node_modules/swagger-node-runner/node_modules/async/lib/async.js:355:9)
    at Object.map (/Users/gebitang/projects/project-remote/RemoteDevice/node_modules/swagger-node-runner/node_modules/async/lib/async.js:337:20)
    at swagger_security (/Users/gebitang/projects/project-remote/RemoteDevice/node_modules/swagger-node-runner/fittings/swagger_security.js:36:11)
    at Runner.<anonymous> (/Users/gebitang/projects/project-remote/RemoteDevice/node_modules/bagpipes/lib/bagpipes.js:171:7)
    at bound (domain.js:301:14)
    at Runner.runBound (domain.js:314:12)
    at Runner.<anonymous> (/Users/gebitang/projects/project-remote/RemoteDevice/node_modules/pipeworks/pipeworks.js:72:17)
    at postFlight (/Users/gebitang/projects/project-remote/RemoteDevice/node_modules/bagpipes/lib/bagpipes.js:220:3)

```
导致前台调用 /devices接口时返回了 403。这有点诡异了，暂时还没定位到原因是什么。

