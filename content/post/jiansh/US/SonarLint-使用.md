+++
title = "SonarLint-使用"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "US"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/4d34224a56ab)

基本上开箱即用。

#### 安装
手动下载：
[不同版本](https://plugins.jetbrains.com/plugin/7973-sonarlint/versions)
下载到本地，然后从Idea的`settings->Plugins->settings install plugin from disk`，需要保留zip包的形式。
下载针对idea的zip包：[https://www.sonarlint.org/intellij/](https://www.sonarlint.org/intellij/)

也可以直接从idea的plugin的market上选择在线安装。

或者直接从源码安装？ [sonarlint-intellij](https://github.com/SonarSource/sonarlint-intellij)

#### 使用

Connect to SonarCloud or to a SonarQube server
To have rules, issues and exclusions synched. In IntelliJ preferences, first connect to a server via the **SonarLint General Settings**, then bind the project under **SonarLint Project Settings**.

插件的设置会在IDEA的 `settings--> other settings`可找到。

配置到对应的SonarServer，搭建Server可参考 [Basic SonarQube SonarScanner](https://www.jianshu.com/p/0b94bc843357)

按照提示使用token（可以在users目录里手动生成）连接后，会自动判断连接状态。

![](https://upload-images.jianshu.io/upload_images/3296949-39755f73c0760d33.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
