+++
title = "Mybatis"
description = "Leaning about Mybatis"
tags = [
    "mybatis"
]
date = "2021-01-14"
topics = [
    "mybatis",
    "Spring"
]
toc = true
+++


## Basic info

[工作原理](http://www.mybatis.cn/archives/706.html)   
[中文官方文档：入门](https://mybatis.org/mybatis-3/zh/getting-started.html)  
[中文官方文档mybatis-spring：入门](http://mybatis.org/spring/zh/)

[最后实际用的应该是 mybatis-spring-boot-starter](http://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/)  
[example for spring-boot-starter](https://github.com/mybatis/spring-boot-starter)  
[Mybatis generator](https://mybatis.org/generator/index.html)

上面三篇读完基本就可以动手了。

JDBC有四个核心对象：
- （1）DriverManager，用于注册数据库连接  
- （2）Connection，与数据库连接对象  
- （3）Statement/PrepareStatement，操作数据库SQL语句的对象  
- （4）ResultSet，结果集或一张虚拟表  

MyBatis四大核心对象：
- （1）SqlSession对象，该对象中包含了执行SQL语句的所有方法。类似于JDBC里面的Connection。
- （2）Executor接口，它将根据SqlSession传递的参数动态地生成需要执行的SQL语句，同时负责查询缓存的维护。类似于JDBC里面的Statement/PrepareStatement。
- （3）MappedStatement对象，该对象是对映射SQL的封装，用于存储要映射的SQL语句的id、参数等信息。
- （4）ResultHandler对象，用于对返回的结果进行处理，最终得到自己想要的数据格式或类型。可以自定义返回类型。

{{< fluid_imgs
  "pure-u-1-1|https://upload-images.jianshu.io/upload_images/3296949-1e957fc8d8970241.png|MyBatis的工作原理|https://upload-images.jianshu.io/upload_images/3296949-1e957fc8d8970241.png"
>}}

## myybatis generator practice

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

## Mybatis spring boot starter practice

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

### insert return primary key

MyBatis insert操作返回主键—— 

[手动封装到bean中，不推荐](https://www.cnblogs.com/sxdcgaq8080/p/10869336.html)  
[官方使用：sqlmap-xml](https://mybatis.org/mybatis-3/sqlmap-xml.html)  
[深入浅出mybatis之返回主键ID](https://www.cnblogs.com/nuccch/p/9067305.html)  

在mapper接口楼中对insert操作进行声明，类似 `@Options(useGeneratedKeys = true, keyProperty = "id")` 

或者使用下面的方式——
```
  <insert id="create" useGeneratedKeys="true" keyProperty="id">
        <selectKey resultType="java.lang.Integer" order="AFTER" keyProperty="id">
            SELECT LAST_INSERT_ID()
        </selectKey>
        INSERT INTO SIN_Manager (Account,Password,Name,Phone,Deleted)
        VALUES (#{account}, #{password},#{name},#{phone},0)
    </insert>
```

- updateByPrimaryKey 将为空的字段在数据库中置为NULL
- updateByPrimaryKeySelective 更新新的model中不为空的字段。

后者可以在对应的mapper.xml文件中看到，类似——
```
<update id="updateByPrimaryKeySelective" parameterType="com.xxx.yyy.test.platform.entity.SUResult" >
    update su_result
    <set >
      <if test="groupName != null" >
        group_name = #{groupName,jdbcType=VARCHAR},
      </if>
      <if test="projectName != null" >
        project_name = #{projectName,jdbcType=VARCHAR},
      </if>
    </set>
    where id = #{id,jdbcType=INTEGER}
  </update>
```
