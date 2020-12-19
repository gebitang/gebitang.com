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


[open source on github: sonarlint-intellij](https://github.com/SonarSource/sonarlint-intellij)  
[sonarlint-intellij on IDEA market](https://plugins.jetbrains.com/plugin/7973-sonarlint) 

如果对JavaScript或TypeScript源码进行扫描，需要安装Node环境(最多版本要求：8）

**配合SonarQube使用**

假设工程没有在SonarQube上。

- 在SonarQube上手动创建项目——

![](https://upload-images.jianshu.io/upload_images/3296949-732ca937746ed96d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 在插件的“设置”中进行SonarQube Server的配置

![](https://upload-images.jianshu.io/upload_images/3296949-1c8b7cc4ac7fa6e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选择 “Bind project to SonarQube/SonarCloud”，然后配置SQ对应的URL地址，下一步中添加对应的认证 token（在SQ上创建项目时会生成，也可以从用户中心生成）

![](https://upload-images.jianshu.io/upload_images/3296949-a987bb3ee9d6ecad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

配置完成后，选择对应的SQ Server和 project Key（创建时定义的项目名称）

![](https://upload-images.jianshu.io/upload_images/3296949-c7a58cec8397e387.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以选择自动触发，也可以点击执行按钮，收到执行。（此时使用的为SonarQube上指定的默认Profile所包含的规则）

执行效果——
![](https://upload-images.jianshu.io/upload_images/3296949-4f4029398f92b2c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
