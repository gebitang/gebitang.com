+++
title = "Spring-boot-project-trial"
tags = [
    "jiansh"
]
date = "2021-03-07"
topics = [
    "jiansh",
    "Spring"
]
toc = true
+++


涉及到的问题：

1. Spring boot工程基础
使用bootcase工程完成练习。

2. yaml文件类型的配置使用，可参考[spring yaml](https://www.baeldung.com/spring-yaml) 
默认位置为`/myApplication/src/main/resources/application.yml` 支持多环境配置

3. 使用druid管理数据库连接
参考[官方文档](https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter)
使用依赖工程里的pom依赖。配置优化为了yml文件格式
```
<dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>druid-spring-boot-starter</artifactId>
   <version>1.1.17</version>
</dependency>
```

4. kafka配置以及基本使用
consumer & producer在两个不同的工程下。
使用yml文件指定对应的 property文件名称，
```
kafka:
  topic: q_test_topic
  consumer:
    properties-file: kafka-consumer-test.properties
```

读取property文件构造对象
```
ClassPathResource resource = new ClassPathResource(kafkaPropFile);
// utils来自spring boot --> spring core
Properties onlineConsumerProperties = PropertiesLoaderUtils.loadProperties(resource);
KafkaConsumer kafkaConsumer = new KafkaConsumer(onlineConsumerProperties);
```

5. redis配置使用
Redis官方提供的Redis client部分Java端的包括Jedis和Lettuce。springboot 2.x版本中默认客户端是用lettuce实现。

Jedis
>使用阻塞的I/O，且其方法调用都是同步的，程序流需要等到sockets处理完I/O才能执行，不支持异步。Jedis客户端实例不是线程安全的，所以需要通过连接池来使用Jedis。

Lettuce	
>高级Redis客户端，用于线程安全同步，异步和响应使用，支持集群，Sentinel，管道和编码器。基于Netty框架的事件驱动的通信层，其方法调用是异步的。Lettuce的API是线程安全的，所以可以操作单个Lettuce连接来完成各种操作

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
默认情况下的模板只能支持 RedisTemplate<String,String>，只能存入字符串。目前工程使用的`StringRedisTemplate`正式默认支持的模板，所以只需要提供默认配置即可——
```
spring:
  redis:
    # Redis服务器地址
    host: redis.dns.fxxk.com
    # 端口号   
    port: 10037
    # Redis数据库索引，默认为0
    database: 0
    # Redis服务器连接密码（默认为空）
    password:
    # 连接超时时间（毫秒）
    timeout: 1000
    poll:
      # 连接池最大连接数（使用负值表示没有限制）
      max-active: 8
      # 连接池最大阻塞等待时间（使用负值表示没有限制）
      max-wait: -1
      # 连接池最大空闲连接
      max-idle: 8
      # 连接池最小空闲连接
      min-idle: 0
```
指定了cluster的方式——
```
spring:
  redis:
    timeout: 6000ms
    database: 0
    cluster:
      nodes:
        - 10.16.9.191:4567
        - 10.16.9.192:4567
        - 10.16.9.193:4567
      max-redirects: 3 # 获取失败 最大重定向次数
    lettuce:
      pool:
        max-active: 1000  #连接池最大连接数（使用负值表示没有限制）
        max-idle: 10 # 连接池中的最大空闲连接
        min-idle: 5 # 连接池中的最小空闲连接
        max-wait: -1 # 连接池最大阻塞等待时间（使用负值表示没有限制）
```

---

*update date one year later*

命令行更新依赖：`mvn dependency:resolve`，或使用`mvn clean -U`。`-U`参数表示强制更新。`Forces a check for missing releases and updated snapshots on remote repositories`。

默认的任务可以使用帮忙函数进行查看，例如 `mvn --help`查看基础参数；`mvn dependency:help`查看依赖相关任务；`mvn clean:help`查看清理函数


[指定配置文件](https://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/html/boot-features-external-config.html#boot-features-external-config-application-property-files)，例如——

```
java -jar start-1.0.0-SNAPSHOT.jar --spring.config.location=file:/E:/gdoc/serviceMap/application.yml
```

[打包为可执行文件](https://www.baeldung.com/spring-boot-run-maven-vs-executable-jar)，需要添加mvn插件`spring-boot-maven-plugin`并指定启动主程序

```
<build>
  <plugins>
      <plugin>
          <!--https://www.baeldung.com/spring-boot-run-maven-vs-executable-jar -->
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-maven-plugin</artifactId>
          <executions>
              <execution>
                  <configuration>
                      <mainClass>com.gebitang.infra.service.Application</mainClass>
                  </configuration>
              </execution>
          </executions>
      </plugin>
  </plugins>
</build>
```


- ~~单独使用`base-mybatis`中的`mybatis-plus-boot-starter`代替`mybatis-spring-boot-starter`时，会提示报错类似下面的内容——(初始化数据库Bean对象时)~~
- ~~即使不引入`base-mybatis`，只引入`base-swagger`时，由于共享了`base-dependencies`中的依赖声明，还是会默认引入`mybatis-plus-boot-starter`~~  

上述两种情况可能是工程中其他依赖的日志与base相关的依赖产生了冲突。

导致`单一引入`的成本过高，只适用于全部都使用。

```
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'sqlSessionFactory' defined in class path resource [com/baomidou/mybatisplus/autoconfigure/MybatisPlusAutoConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.apache.ibatis.session.SqlSessionFactory]: Factory method 'sqlSessionFactory' threw exception; nested exception is java.lang.NoClassDefFoundError: org/mybatis/logging/LoggerFactory
	at org.springframework.beans.factory.support.ConstructorResolver.instantiate(ConstructorResolver.java:658)
	at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:638)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1336)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1179)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:571)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:531)
	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:335)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:234)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:333)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:208)
	at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:276)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1367)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1287)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.autowireByType(AbstractAutowireCapableBeanFactory.java:1503)
	... 75 common frames omitted
Caused by: org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.apache.ibatis.session.SqlSessionFactory]: Factory method 'sqlSessionFactory' threw exception; nested exception is java.lang.NoClassDefFoundError: org/mybatis/logging/LoggerFactory
	at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:185)
	at org.springframework.beans.factory.support.ConstructorResolver.instantiate(ConstructorResolver.java:653)
	... 88 common frames omitted
Caused by: java.lang.NoClassDefFoundError: org/mybatis/logging/LoggerFactory
	at com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean.<clinit>(MybatisSqlSessionFactoryBean.java:94)
	at com.baomidou.mybatisplus.autoconfigure.MybatisPlusAutoConfiguration.sqlSessionFactory(MybatisPlusAutoConfiguration.java:159)
	at com.baomidou.mybatisplus.autoconfigure.MybatisPlusAutoConfiguration$$EnhancerBySpringCGLIB$$f879f3fe.CGLIB$sqlSessionFactory$1(<generated>)
	at com.baomidou.mybatisplus.autoconfigure.MybatisPlusAutoConfiguration$$EnhancerBySpringCGLIB$$f879f3fe$$FastClassBySpringCGLIB$$b09bd244.invoke(<generated>)
	at org.springframework.cglib.proxy.MethodProxy.invokeSuper(MethodProxy.java:244)
	at org.springframework.context.annotation.ConfigurationClassEnhancer$BeanMethodInterceptor.intercept(ConfigurationClassEnhancer.java:331)
	at com.baomidou.mybatisplus.autoconfigure.MybatisPlusAutoConfiguration$$EnhancerBySpringCGLIB$$f879f3fe.sqlSessionFactory(<generated>)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:154)
	... 89 common frames omitted
Caused by: java.lang.ClassNotFoundException: org.mybatis.logging.LoggerFactory
	at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	... 101 common frames omitted
```

[Introduction to the Dependency Mechanism](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)中，`import`范围只针对`<dependencyManagement>`中依赖定义为`pom`的类型有效，表示只是提供依赖列表信息，不参与实际的编译过程。

```
<dependency>
    <groupId>com.gebitang.base</groupId>
    <artifactId>base-dependencies</artifactId>
    <version>1.0-SNAPSHOT</version>
    <scope>import</scope>
    <type>pom</type>
</dependency>

```

依赖机制：A引入B，则B锁需要的所有依赖都自动引入，依赖的依赖也会被自动引入。没有依赖深度限制。

可能发生循环依赖，或引入了相同包的不同版本，此时，maven引入以下机制——

- 最近原则。依赖深度最近的包被使用，依赖深度更深的包被忽略。这样，当有包版本冲突时，直接在pom文件总声明想要的版本就可以被优先使用
- 依赖管理。通过在`<dependencyManagement>`中集中声明和管理依赖的版本。匹配了management中的依赖包，优先使用其对应的版本
- 依赖范围。默认为`compile`，其他的包括`provided`(有JDK提供或启动的容器提供)、`runtime`(编译期不需要，运行时才使用)、`test`、`system`(类似于provide，但需要显示声明所在的路径，例如`<systemPath>${java.home}/lib/rt.jar</systemPath>`)、`import`

