+++
title = "Spring-boot服务(一)"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "Spring"
]
toc = true
+++

[link to JianShu](https://www.jianshu.com/p/0b596eab81ea)

### 阿里云
1. 目前阿里云新用户有优惠，89年一年。购买的是劵，然后用劵再正式开通服务，设置好用户名密码，默认22端口开通。可以远程访问。
2. 阿里云实例列表--更多--网络和安全组--安全组配置--配置规则--增加入网的端口。这样才可以访问对应的端口
3. 增加[日常用户](http://gebitang.com/post/how/linux-and-shell/#用户管理)
4. [安装mysql服务](https://blog.csdn.net/LunaEditor/article/details/83051809) 
5. 安装maven `sudo apt install maven`，配置阿里云镜像。创建 `.m2/settings.xml`文件，内容如下——

```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                              https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <localRepository />
    <interactiveMode />
    <usePluginRegistry />
    <offline />
    <pluginGroups />
    <servers />
    <mirrors>
        <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>*</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
    </mirrors> 
    <proxies />
    <profiles />
    <activeProfiles />
</settings>
```
6. git服务默认已经安装。

---
基本上这样就可以在服务器上构建自己的简单服务了。

### Spring boot 
默认打包会将依赖包都打到一个包里，如果每次都上传jar包到服务器上，有点麻烦。所以直接在服务器上从源打包运行。——先RUN起来再说。

类似这样[hello](http://geb.im:8081/demo/welcome)一下

[unable-to-run-spring-boot-on-80-port](https://reviewdb.io/questions/1503131900849/unable-to-run-spring-boot-on-80-port)

linux下占用1024以下的端口都需要root用户权限。替代方案参考上面的链接。

阿里云上的80端口会要求备案。FML
