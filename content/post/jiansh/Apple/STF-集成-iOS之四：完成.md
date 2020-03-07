+++
title = "STF-集成-iOS之四：完成"
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



[link on JianShu](https://www.jianshu.com/p/42f8a8ceb0d6)

谨慎起见，开了个分支集成STF下的iOS远程真机。

事实上，[STF 集成 iOS之三：对接](https://www.jianshu.com/p/9829abb40149)里提到的前端集成关键的就是图片传输协议——

- `res/app/components/stf/control/control-service.js`新增加了 `screendump`方法发送zmq消息通知，
- `lib/units/device/plugins/screen/dump.js`，接收到消息后完成截图动作。

昨天图像没显示出来，有可能是WDA还没有build成功——需要等待一段时间。

另外就是 需要确保WDA已经正常启动起来。 

>需要先手动启动WDA服务（不确认是否是这样，测试实践的结果是这样的）
如果没有手动先启动WDA，则只有一次 “ xcodebuild构建成功”的log输出。 设备一直处于 waiting状态。

---
总结一下：移植完成后，使用provider方式启动，提供的最终效果类似这样。——

![](https://upload-images.jianshu.io/upload_images/3296949-ae0ed5563a71ec86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

