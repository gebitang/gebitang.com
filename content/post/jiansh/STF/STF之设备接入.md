+++
title = "STF之设备接入"
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



[link on JianShu](https://www.jianshu.com/p/fba47e162299)

这是个体力活。

先要把设备信息先记录下来，类似这样都信息需要先手动录入——
![](https://upload-images.jianshu.io/upload_images/3296949-32a7cb6742e5c2b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 打开USB调试
USB调试打开都开关并没有统一到“点击7次内核版本”上，需要在手机信息里试。比如小米是在MIUI到版本上多点击几次——倒是发现了隐藏属性：点击3次内核版本，打开cit测试；点击处理器多次，开启抓log。

### 注册对应账号密码
国内都华为、OPPO、VIVO、小米设备都得先注册上账号密码才能打开USB调试。小米的设备必须要有SIM卡才能激活“允许USB安装应用”的功能。 ——不过可以激活后再把SIM卡拔掉

### minitouch兼容性
目前发现多款小米设备——包括红米Note7、小米Max2、小米Max3——无法启动minitouch组件，提示类似——
```
2019-08-02T03:34:40.705Z INF/device:plugins:touch 9042 [f22b41e] minitouch says: "open: Permission denied"
2019-08-02T03:34:40.705Z INF/device:plugins:touch 9042 [f22b41e] minitouch says: "Unable to open device /dev/input/event6 for inspectionopen: Permission denied"
2019-08-02T03:34:40.705Z INF/device:plugins:touch 9042 [f22b41e] minitouch says: "Unable to open device /dev/input/event5 for inspectionopen: Permission denied"
2019-08-02T03:34:40.705Z INF/device:plugins:touch 9042 [f22b41e] minitouch says: "Unable to open device /dev/input/event2 for inspectionopen: Permission denied"
2019-08-02T03:34:40.706Z INF/device:plugins:touch 9042 [f22b41e] minitouch says: "Unable to open device /dev/input/mouse0 for inspectionopen: Permission denied"
2019-08-02T03:34:40.706Z INF/device:plugins:touch 9042 [f22b41e] minitouch says: "Unable to open device /dev/input/event1 for inspectionopen: Permission denied"
2019-08-02T03:34:40.706Z INF/device:plugins:touch 9042 [f22b41e] minitouch says: "Unable to open device /dev/input/event0 for inspectionopen: Permission denied"
2019-08-02T03:34:40.706Z INF/device:plugins:touch 9042 [f22b41e] minitouch says: "Unable to open device /dev/input/mice for inspectionopen: Permission denied"
2019-08-02T03:34:40.707Z INF/device:plugins:touch 9042 [f22b41e] minitouch says: "Unable to open device /dev/input/event3 for inspectionopen: Permission denied"

```

[stf issues 477](https://github.com/openstf/stf/issues/477) 这个看起来是通用问题，解决方式为：打开开发者选项下到 **USB调试**选项即可。


### ubuntu系统下的问题
目前出现了设备截图无法显示的问题。目前还没有定位原因
```
imgUrl:
http://stf.demo.com/s/image/71775e2a-da0a-4683-86db-0704f8b89dbd/9YEDU19409001451.jpg

server:
2019-08-02T07:02:43.539Z INF/reaper 11673 [reaper001] Device "9YEDU19409001451" is present
2019-08-02T07:04:14.004Z INF/storage:temp 11709 [*] Uploaded "9YEDU19409001451.jpg" to "/tmp/upload_2dc2363adc71b10f6e3b79acf8c2c27d"

provider:
2019-08-02T07:04:14.031Z INF/device:support:storage 10175 [9YEDU19409001451] Uploaded to "/s/image/71775e2a-da0a-4683-86db-0704f8b89dbd/9YEDU19409001451.jpg"
```
Mac环境下本地启动的截图没有问题，看起来不像是代码问题，怀疑是目录权限之类的，今天（08.07）定位一下这个问题——问题已定位，[参考这里](https://www.jianshu.com/p/f6d334c822be)
