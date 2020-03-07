+++
title = "Maven-pom-packaging"
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



[link on JianShu](https://www.jianshu.com/p/dbac10201154)

[What is “pom” packaging in maven?](https://stackoverflow.com/questions/7692161/what-is-pom-packaging-in-maven)
>`pom` packaging is simply a specification that states the primary artifact is not a war or jar, but the pom.xml itself.

只是一直打包形式，打包为pom文件本身。加入当前目录是做完文档模块存在，显然无法打包为`jar`形式。

那么这种类型的模块如果作为依赖使用呢？
[Use pom-packaging maven project as dependency](https://stackoverflow.com/questions/40032721/use-pom-packaging-maven-project-as-dependency)

明确知道依赖的type类型为pom即可，类似——
```
<dependency>
    <groupId>the.pom.project</groupId>
    <artifactId>aggregate-pom</artifactId>
    <version>1.0</version>
    <type>pom</type>
</dependency>
```

官方文档有更详细的说明，[dependency mechanism](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)

官方文档的 用户中心（User Centre）部分读完，基本上万事可解。核心部分就是[getting started](https://maven.apache.org/guides/getting-started/index.html)里的内容了。

![](https://upload-images.jianshu.io/upload_images/3296949-c95e1698ded5a32b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
