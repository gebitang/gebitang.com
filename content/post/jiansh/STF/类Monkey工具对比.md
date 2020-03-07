+++
title = "类Monkey工具对比"
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



[link on JianShu](https://www.jianshu.com/p/13d09692a410)

原生[Monkey](https://developer.android.google.cn/studio/test/monkey)：

[源码参考](http://androidxref.com/5.0.0_r2/xref/development/cmds/monkey/src/com/android/commands/monkey/)

实现原理：Java反射获取系统接口，注入事件 

[http://androidxref.com/5.0.0_r2/xref/development/cmds/monkey/src/com/android/commands/monkey/Monkey.java](http://androidxref.com/5.0.0_r2/xref/development/cmds/monkey/src/com/android/commands/monkey/Monkey.java)

控制业务逻辑；

[http://androidxref.com/5.0.0_r2/xref/development/cmds/monkey/src/com/android/commands/monkey/MonkeyMotionEvent.java](http://androidxref.com/5.0.0_r2/xref/development/cmds/monkey/src/com/android/commands/monkey/MonkeyMotionEvent.java)

事件注入实现

功能——

系统自带支持；随机点击、滑动、系统事件如音量、亮度等；

设置内容：事件数量、范围（指定在package）、比例分布（各个事件的比例）、Debugg选项（ANR、Crash是否退出）

执行log输出

---

[Maxin](https://testerhome.com/topics/11719)：

最低版本要求：android 5 

兼容原生Monkey 参数定义；

基于Android Monkey开发：

实现原理使用AccesssibilityNodeInfo生成事件
```
adb shell CLASSPATH=/sdcard/monkey.[jar:/sdcard/framework.jar](http://jar/sdcard/framework.jar) exec app_process /system/bin tv.panda.test.monkey.Monkey -p com.panda.videoliveplatform --uiautomatormix --running-minutes 60
```
特色功能——

速度足够快：每秒达到10-15个事件；避免点击重复位置

防止跳出以及点击到状态栏：选择区域时已经过滤掉状态栏（防止误操作）

判断：唤醒屏幕

支持白名单Activity；

参数设置输入内容；

特殊事件序列：可利用此功能实现登录效果。如：判断在当前Activity下：执行输入、点击动作

黑控件区域配置：

截图以及回溯式截图：（发送崩溃式，利用回溯式截图提供截图；复现是使用直接截图功能）

不同执行策略：参数说明部分

--uiautomatormix 混合模式（70%控件解析随机点击，其余30%按原Monkey事件概率分布） 
--pct-uiautomatormix n 可自定义混合模式中控件解析事件概率
--uiautomatordfs DFS深度遍历算法（优化版）（注 Android5不支持dfs）
--uiautomatortroy Troy模式 指定选择哪些类型的控件进行操作

---
[app Crawler](https://github.com/seveniruby/AppCrawler)：

待调用细则实现——

> 自动爬取加上规则引导(完成)
> 
> 支持定制化, 可以自己设定遍历深度(完成)
> 
> 支持插件化, 允许别人改造和增强(完成)
> 
> 支持滑动等更多动作(完成)
> 
> 支持自动截获接口请求(完成)
> 
> 支持新老版本的界面对比(Doing)

---

[google AppCrawler](https://developer.android.google.cn/training/testing/crawler)：

beta状态，可用性不高；依赖sdk环境；加固应用不支持，待跟踪观察效果(2019.11)

|  | 原生Monkey | Maxim | app Crawler | google appCrawler |
| --- | --- | --- | --- | --- |
| 兼容 | android全系列 | > android 5 | android, iOS | all android  |
| 语言 | java | kotlin | scala | java? |
| 功能 | 单一，随机 | 定制功能 | 遍历 | 遍历 |
| 依赖 | 无 | 无 | appium | android sdk |
| 结果 | log text | log text、img | web展示，文件较多 | log |
| 开源 | 否 | 部分开源，活跃 | 开源，2017年 | 否 |
