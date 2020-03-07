+++
title = "Basic-Mybatis(一)"
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

[link to JianShu](https://www.jianshu.com/p/78242834b78e)

[工作原理](http://www.mybatis.cn/archives/706.html)   
[中文官方文档：入门](https://mybatis.org/mybatis-3/zh/getting-started.html)  
[中文官方文档mybatis-spring：入门](http://mybatis.org/spring/zh/)

[最后实际用的应该是 mybatis-spring-boot-starter](http://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/)  
[example for spring-boot-starter](https://github.com/mybatis/spring-boot-starter)  
[Mybatis generator](https://mybatis.org/generator/index.html)


---

上面三篇读完基本就可以动手了。

JDBC有四个核心对象：
（1）DriverManager，用于注册数据库连接
（2）Connection，与数据库连接对象
（3）Statement/PrepareStatement，操作数据库SQL语句的对象
（4）ResultSet，结果集或一张虚拟表

MyBatis四大核心对象：
（1）SqlSession对象，该对象中包含了执行SQL语句的所有方法。类似于JDBC里面的Connection。
（2）Executor接口，它将根据SqlSession传递的参数动态地生成需要执行的SQL语句，同时负责查询缓存的维护。类似于JDBC里面的Statement/PrepareStatement。
（3）MappedStatement对象，该对象是对映射SQL的封装，用于存储要映射的SQL语句的id、参数等信息。
（4）ResultHandler对象，用于对返回的结果进行处理，最终得到自己想要的数据格式或类型。可以自定义返回类型。

![MyBatis的工作原理](https://upload-images.jianshu.io/upload_images/3296949-1e957fc8d8970241.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


 
