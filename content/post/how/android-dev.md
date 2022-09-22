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

```
