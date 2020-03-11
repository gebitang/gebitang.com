+++
title = "Basic-Mybatis(二)-生成数据库操作类以及mvn调用"
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



[link on JianShu](https://www.jianshu.com/p/9bc58d486726)

### Mybatis 

[工作原理](http://www.mybatis.cn/archives/706.html)   
[中文官方文档：入门](https://mybatis.org/mybatis-3/zh/getting-started.html)  
[中文官方文档mybatis-spring：入门](http://mybatis.org/spring/zh/)

[最后实际用的应该是 mybatis-spring-boot-starter](http://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/)
[example for spring-boot-starter](https://github.com/mybatis/spring-boot-starter)

![MyBatis的工作原理](https://upload-images.jianshu.io/upload_images/3296949-1e957fc8d8970241.png)

#### Mybatis generator practice

[quick start](https://mybatis.org/generator/quickstart.html)默认使用`generatorConfig.xml`文件定义如何generate。 
[With Maven](https://mybatis.org/generator/running/runningWithMaven.html):  
- 添加plugin
```
<plugin>
  <groupId>org.mybatis.generator</groupId>
  <artifactId>mybatis-generator-maven-plugin</artifactId>
  <version>1.4.0</version>
</plugin>
```
- 执行mvn 命令 `mvn -Dmybatis.generator.overwrite=true mybatis-generator:generate`

- 配置文件内容属性详情[xml config](https://mybatis.org/generator/configreference/xmlconfig.html)

使用低版本的MySQL Connector(5.x, 6.x版本)，可能出现 `mybatis generator “Column name pattern can not be NULL or empty”`[错误](https://stackoverflow.com/a/39270234/1087122)：  
- 使用高版本（8.0.x）即可，
- 或在连接属性里增加 `?nullNamePatternMatchesAll=true`内容

#### Mybatis spring boot starter practice

[official guides](https://spring.io/guides)  
[guide: accessing mysql data](https://spring.io/guides/gs/accessing-data-mysql/)  
[mybatis spring boot config](http://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/)


overwrite existed xml files: [参考](https://segmentfault.com/a/1190000013038272)
`mvn -Dmybatis.generator.overwrite=true mybatis-generator:generate` not work.

```
The overwrite property is only used for generated Java files. It should not affect XML files at all. XML files should always be merged.

Have you configured a comment generator with suppressAllComment=true? If so, that would be the cause of this behavior. The XML merge won't delete old elements if the comments are removed.

```
overwrite 配置只是为了覆盖 java 类, xml 文件是不受这个控制的

mvn deploy 提示401： 
确保mvn settings中的 servers下的server字段的id名称与distributionManagement下repository字段下的id名称相同（大小写敏感）

```
<servers>
    ...
    <server>
        <id>snapshots</id>
        <username>username</username>
        <password>password</password>
    </server>
</servers>


<distributionManagement>
    <snapshotRepository>
        <id>snapshots</id>
        <url>http://nexus.mynexus.service.com:8081/nexus/content/repositories/snapshots/</url>
    </snapshotRepository>
</distributionManagement>

```
