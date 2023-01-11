+++
title = "Scrum for Android"
description = "YKYL"
tags = [
    "android"
]
date = "2022-09-15"
topics = [
    "android"
]
draft = true
toc=true
+++

TODO:

- Meson编译scrcpy，[Meson构建系统](https://blog.csdn.net/u010074726/article/details/108695256) 
- gradle配置优化



Demo级别：

- 简易apk：主页面，按钮展示自描述信息；支持响应广播启动activity
- app_process方式启动 server：socker服务；
- 如何打包为dex文件
- 联调验证：adb转发之后，PC端调用api请求

1. socker服务检测心跳断开之后，发起广播给自己，接收到广播之后，打开红屏activity
2. 还是直接启动activity即可？

[How to start an activity from BroadcastReceiver's onReceive() method when app is in background?](https://stackoverflow.com/a/60183640/1087122) 高版本中，如果应用在后台运行时，对启动activity做了对应的限制，推荐的做法是[Display time-sensitive notifications](https://developer.android.com/training/notify-user/time-sensitive)或者展示一个 [full-screen intent](https://developer.android.com/reference/android/app/Notification.Builder#setFullScreenIntent(android.app.PendingIntent,%20boolean))

```
adb shell am broadcast 后面的参数有：

[-a <ACTION>]
[-d <DATA_URI>]
[-t <MIME_TYPE>] 
[-c <CATEGORY> [-c <CATEGORY>] ...] 
[-e|--es <EXTRA_KEY> <EXTRA_STRING_VALUE> ...] 
[--ez <EXTRA_KEY> <EXTRA_BOOLEAN_VALUE> ...] 
[-e|--ei <EXTRA_KEY> <EXTRA_INT_VALUE> ...] 
[-n <COMPONENT>]
[-f <FLAGS>] [<URI>]

adb shell am broadcast -a com.android.test --es test_string "this is test string" --ei test_int 100 --ez test_boolean true

adb shell pm list packages
```

## 导入的依赖无法import

实际上依赖并没有被下载下来，提示 `Failed to resolve dependency xxx`，对应的依赖比较古老，需要在settings.gradle里的依赖中添加 `jcenter()` [参考](https://stackoverflow.com/a/71799874/1087122)


## 使用 android.jar 中的隐藏api 

原生monkey依赖源码中的隐藏api，如果想修改编译需要得到包含隐藏api的jar包做依赖，包括至少以下imports.这个问题[How do I build the Android SDK with hidden and internal APIs available?](https://stackoverflow.com/questions/7888191/how-do-i-build-the-android-sdk-with-hidden-and-internal-apis-available)虽然有点年头，但可以看到项目[android-hidden-api](https://github.com/anggrayudi/android-hidden-api.git)作者的答案。可以从google driver下载对应的jar包。第一步完成。

```java
import android.app.IActivityController;
import android.app.IActivityManager;
import android.content.IIntentReceiver;
import android.content.pm.IPackageManager;
import android.os.ServiceManager;
import android.view.IWindowManager;
```

- The internal APIs are located in package `com.android.internal` and available in the `framework.jar`,
- while the hidden APIs are located in the `android.jar` file with `@hide` javadoc attribute.

[手动实现步骤](https://hardiannicko.medium.com/create-your-own-android-hidden-apis-fa3cca02d345)——将

- 获取两个jar包
  - 从android sdk的platform下得到对应版本的`android.jar`文件
  - 从虚拟机或真实设备上获取`framwork.jar`文件 `adb pull /stystem/framework/framework.jar`
- 解压jar包得到class文件
  - 重命名framework.jar为zip格式，解压出其中的多个`classes.dex`，`classes2.dex`，`classes3.dex` ，`classes4.dex`  
  - 使用[dex2jar](https://github.com/pxb1988/dex2jar)工具将dex文件转换为jar包，再从中解压出class文件
  - 解压android.jar获取claess文件
- 合并class文件并生成jar包
  - 新建目录，将所有的class文件复制到一起，有重复时Mac下选择`merge`，Windows下选择`copy and replace`
  - 将当前目录所有内容打包为jar包 `jar cvf android.jar *`

(实操在处理出lasses.dex文件时报错)
