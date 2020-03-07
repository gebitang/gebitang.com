+++
title = "JaCoCo-Multi-Module"
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



[link on JianShu](https://www.jianshu.com/p/a65f9b99105e)

[官方提供的说明](https://github.com/jacoco/jacoco/wiki/MavenMultiModule)

>Create a dedicated module in your project for generation of the report. This module should depend on all or some other modules in the project. JaCoCo itself does this in the org.jacoco.doc module (see the [pom.xml](https://github.com/jacoco/jacoco/blob/98a2a41e30f2044b151e4a986cac032961a5abb8/org.jacoco.doc/pom.xml)).

JaCoCo 0.7.7版本开始提供这个功能。使用独立的模块，专门用来聚合测试结果。

基本思路——

- 在该模块中将其它所有的moudle都添加为依赖
- pom中添加jacoco插件 

参考上面官方的pom文件，精简后如下，只需要动态的将依赖的模块添加进来即可。当前模块不能将packaging指定为pom格式。

>do not set packaging to pom, because otherwise we will receive "Not executing Javadoc as the project is not a Java classpath-capable package

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.you.aggregate.module</groupId>
    <artifactId>jacoco.aggregate</artifactId>
    <version>1.0-SNAPSHOT</version>
    <dependencies>

    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.19.1</version>
                <configuration>
                    <forkCount>1</forkCount>
                    <runOrder>random</runOrder>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.7.9</version>
                <executions>
                    <execution>
                        <id>report-aggregate</id>
                        <phase>verify</phase>
                        <goals>
                            <goal>report-aggregate</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>

```

### 依赖问题

但依赖的子模块是可以将packaging指定为`pom`格式的，默认为`jar`格式。如果是`pom`格式时，需要对依赖进行明确的声明，增加`<type>pom</type>`内容，例如——

```
<dependencies> 
<dependency>
  <groupId>com.basic.pro</groupId>
  <artifactId>business-api</artifactId>
  <version>1.0.1-SNAPSHOT</version>
</dependency>
<dependency>
  <groupId>com.basic.pro</groupId>
  <artifactId>business-common</artifactId>
  <version>1.0.1-SNAPSHOT</version>
</dependency>
<dependency>
  <groupId>com.basic.pro</groupId>
  <artifactId>business-util</artifactId>
  <version>1.0.1-SNAPSHOT</version>
  <type>pom</type>
</dependency>
</dependencies>  
```

最终执行的命令为：`mvn clean jacoco:prepare-agent install jacoco:report`

--- 

JaCoCo的` jacoco:prepare-agent`的含义为对jaCoCo的agent指定一些参数值。参考这里的[说明](https://stackoverflow.com/a/45973128)


>`Prepares` a property pointing to the JaCoCo runtime agent that can be
passed as a VM argument to the application under test.

---

### sonar scanner multi module

区别于sonar scanner对multi module的处理，参考这里的官方说明[sonarscanner-for-maven/](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-maven/)以及对应的示例[scanning-examples](https://github.com/SonarSource/sonar-scanning-examples/tree/master/sonarqube-scanner-maven)

包括[basic](https://github.com/SonarSource/sonar-scanning-examples/tree/master/sonarqube-scanner-maven/maven-basic)和[multi module](https://github.com/SonarSource/sonar-scanning-examples/tree/master/sonarqube-scanner-maven/maven-multimodule)

JaCoCo本身支持定义规则：不满足单测标准时，build失败。[参考](https://stackoverflow.com/questions/41891739/how-to-fail-a-maven-build-if-junit-coverage-falls-below-certain-threshold)



























