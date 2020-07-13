+++
title = "Maven"
description = "Everything about maven"
tags = [
]
date = "2020-07-13"
topics = [
    "maven",
    "tech"
]
toc = true
+++

[Maven](https://maven.apache.org/what-is-maven.html)最初是为了更好地为[Jakarta Turbine项目](http://jakarta.apache.org/)(已于2011年12月11日退役)打包而开发的工具。

>Maven, a Yiddish word meaning *accumulator of knowledge*, began as an attempt to simplify the build processes in the Jakarta Turbine project. There were several projects, each with their own Ant build files, that were all slightly different. JARs were checked into CVS. We wanted a standard way to build the projects, a clear definition of what the project consisted of, an easy way to publish project information, and a way to share JARs across several projects.

>"Maven" is really just a core framework for a collection of Maven Plugins.

[Maven in five minutes](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)


<!--more-->

## Maven Plugin Dev

[官方文档](https://maven.apache.org/guides/plugin/guide-java-plugin-development.html)，[baeldung demo](https://www.baeldung.com/maven-plugin)

官方最简单的一个Plugin: [Maven clean plugin](https://github.com/apache/maven-clean-plugin)， 核心代码[CleanMojo](https://github.com/apache/maven-clean-plugin/blob/master/src/main/java/org/apache/maven/plugins/clean/CleanMojo.java)——用来“删除各种文件夹”。

- 护身符**Mojo**? A mojo is a **M**aven **O**ld **J**ava **O**bject. Each mojo is an executable **goal** in Maven, and a plugin is a distribution of one or more related mojos.
- 借用POJO(Plain Old Java Object)的概念，用Maven替代了Plain。
- [POJO](https://www.martinfowler.com/bliki/POJO.html)是由ThoughsWork的[Martin Fowler](https://www.martinfowler.com/)，Rebecca Parsons和Josh MacKenzie为2000年9月的一场演讲“杜撰”的新词，用来取代[Entity Bean](https://en.wikipedia.org/wiki/Entity_Bean)的概念。然后就流行起来了。——有个好名字很重要啊


demo代码如下就算一个完整的plugin了。

```java
package sample.plugin;
 
import org.apache.maven.plugin.AbstractMojo;
import org.apache.maven.plugin.MojoExecutionException;
import org.apache.maven.plugins.annotations.Mojo;
 
/**
 * Says "Hi" to the user.
 * 
 */
@Mojo( name = "sayhi")
public class GreetingMojo extends AbstractMojo {

    /**
     * The greeting to display.
     * 传递参数
     */
    @Parameter( property = "sayhi.greeting", defaultValue = "Hello World!" )
    private String greeting;

    public void execute() throws MojoExecutionException {
        getLog().info(greeting);
    }
}

```

注意事项： 

- 需要添加必要的依赖

```xml
<dependencies>
    <!-- core framwork api -->
    <dependency>
        <groupId>org.apache.maven</groupId>
        <artifactId>maven-plugin-api</artifactId>
        <version>3.6.3</version>
    </dependency>
    <!-- 注解 -->
    <dependency>
        <groupId>org.apache.maven.plugin-tools</groupId>
        <artifactId>maven-plugin-annotations</artifactId>
        <version>3.6.0</version>
        <scope>provided</scope>
    </dependency>
    <!-- 获取maven项目本身的信息 -->
    <dependency>
        <groupId>org.apache.maven</groupId>
        <artifactId>maven-project</artifactId>
        <version>2.2.1</version>
    </dependency>
</dependencies>
```

- 执行plugin时的3种默认规则

一、可以使用完整的`groupID:projectID:goalName`的方式，默认使用最新版本。指定版本使用[`mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.8-SNAPSHOT:sonar`](https://github.com/SonarSource/sonar-scanner-maven/blob/master/pom.xml#L11)指定配置的插件版本。  

二、如果符合规范的命名，例如 `${prefix}-maven-plugin`，则可以使用默认的 `mvn prefix:goalName`方式执行。可以在插件中自定义`mvn somePrefix:goalName`，类似——

```xml
<project>
  ...
  <build>
    ...
    <plugins>
      ...
      <plugin>
        <artifactId>maven-plugin-plugin</artifactId>
        <version>2.3</version>
        <configuration>
          ...
          <goalPrefix>somePrefix</goalPrefix>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

三、使用配置文件`settings.xml`进行[预定义插件](https://maven.apache.org/guides/introduction/introduction-to-plugin-prefix-mapping.html)——

```xml
<pluginGroups>
  <pluginGroup>org.codehaus.modello</pluginGroup>
</pluginGroups>
```


## 单测中使用的几个插件

- [`maven-site-plugin`](https://maven.apache.org/plugins/maven-site-plugin/) 

用来生成静态web站点   

>The Site Plugin is used to generate a site for the project. The generated site also includes the project's reports that were configured in the POM.


- [`maven-project-info-reports-plugin`]()

生成项目信息

>The Maven Project Info Reports plugin is used to generate reports information about the project.


- [maven-surefire-plugin](https://maven.apache.org/surefire/maven-surefire-plugin/)

> The Surefire Plugin has only one goal: `surefire:test` runs the unit tests of an application.

- [`maven-surefire-report-plugin`](https://maven.apache.org/surefire/maven-surefire-report-plugin/)

将测试结果生成HTML格式的报告

>Generate the Report as Part of Project Reports

*   [surefire-report:report](https://maven.apache.org/surefire/maven-surefire-report-plugin/report-mojo.html) Generates the test results report into HTML format.
*   [surefire-report:report-only](https://maven.apache.org/surefire/maven-surefire-report-plugin/report-only-mojo.html) This goal does not run the tests, it only builds the reports. It is provided as a work around for [SUREFIRE-257](https://issues.apache.org/jira/browse/SUREFIRE-257)
*   [surefire-report:failsafe-report-only](https://maven.apache.org/surefire/maven-surefire-report-plugin/failsafe-report-only-mojo.html) This goal does not run the tests, it only builds the IT reports. See [SUREFIRE-257](https://issues.apache.org/jira/browse/SUREFIRE-257)

## mvn clean install 

[a short guide to Maven](https://www.marcobehler.com/guides/mvn-clean-install-a-short-guide-to-maven)  

1.  You are calling the `mvn` executable, which means you need Maven installed on your machine. 

2.  You are using the `clean` command, which will delete all previously compiled Java sources and resources (like .properties) in your project. Your build will start from a clean slate.

3.  `Install` will then compile, test & package your Java project and even `install`/`copy` your built .jar/.war file into your local Maven repository.

核心作用就是：通常说的“编译、打包”。clean是为了先清除原有的产出物，最终的阶段是：

- 打包(package) 
- 验证(verify) 通常没有[集成测试](https://www.baeldung.com/maven-integration-test)
- 安装(install)到本地
- 部署(deploy)到配置的远程仓库

参理解生命周期。[Maven-build-lifesycle](../jiansh/spring/maven-build-lifesycle)

![](https://www.marcobehler.com/images/mavenphasesv3.png)

## 单测失败

[Failed to execute goal org.apache.maven.plugins:maven-surefire-plugin:2.10:test](https://stackoverflow.com/questions/13170860/failed-to-execute-goal-org-apache-maven-pluginsmaven-surefire-plugin2-10test)


Locate the dependencies you're missing with `mvn dependency::tree`

## maven clean package

[执行逻辑](https://www.cnblogs.com/frankyou/p/6062179.html)  

1、使用清理插件：maven-clean-plugin:2.5执行清理删除已有target目录（版本2.5）；

2、使用资源插件：maven-resources-plugin:2.6执行资源文件的处理（版本2.6）；

3、使用编译插件：maven-compiler-plugin:3.1编译所有源文件生成class文件至target\classes目录下（版本3.1）；

4、使用资源插件：maven-resources-plugin:2.6执行测试资源文件的处理（版本2.6）；

5、使用编译插件：maven-compiler-plugin:3.1编译测试目录下的所有源代码（版本3.1）；

6、使用插件：maven-surefire-plugin:2.12运行测试用例（版本2.12）；

7、使用插件：maven-jar-plugin:2.4对编译后生成的文件进行打包，包名称默认为：artifactId-version，比如本例生成的jar文件：rtp-front-1.0-SNAPSHOT，包文件保存在target目录下（这个生成的包不能在命令行中直接执行，因为我们还没有入口类配置到Manifest资源配置文件中去，后续会阐述）。

备注：

不管是compile、package还是install等前三个步骤都是必不可少的。


## editor不报错，但执行main方法时提示找不到class

可能的情况是：工程配置中的`sources`的路径配置错误或没有指定对应的文件夹。

## 无法获取到dependency的jar包

新系统下一阵慌乱：本地仓库无法修改？dependency无法下载？（恢复默认，重新加载……都不好使）

仔细检查一下：最终结果是——1. 内网搭建的仓库下没有对应的jar包；2. aliyun的仓库居然罕见的正遇上无法访问。

没有黑魔法，知道原理，明白到底发生了什么，就没什么好慌乱的了。

## maven项目默认Java编译版本
Intellij Idea 创建Maven项目时，默认的Java Language是1.5。[修改](https://www.cnblogs.com/liu-s/p/5371289.html)
```
<properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>

# 或
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.1</version>
            <configuration>
                <!-- http://maven.apache.org/plugins/maven-compiler-plugin/ -->
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```


## cannot resolve symbol

dependency中有对应的条目，到提示找不到类。


不同的工程打开后使用的Maven配置不同，打开相同工程时，对应的jar包条目存在，但实际的类找不到--（没有下载到）
注意使用不同的Maven设置即可

也可能是idea中的index报错，`file--> invalidate caches`

## 依赖多个模块
依赖模块的Maven需要重新导入 `reimport all maven projects` 之后，依赖模块的maven工程才会利用自己的pom文件生成依赖。

- 下载apache-maven-3.5.0-bin.tar.gz到本地
- 解压：`tar -xvf apache-maven-3.5.0-bin.tar.gz`
- 更新配置文件：

## Maven 本地使用

```
M2_HOME=/Users/gebitang/apache-maven-3.5.0
PATH=$PATH:$JAVA_HOME/bin:/Users/gebitang/libs/:/usr/local/smlnj/bin:$M2_HOME/bin
```

## 安装依赖

mvn install 

- 如需要，修改conf文件夹下的settings文件中的localRepository

## 安装jar包到本地
```
mvn install:install-file -Dfile=itestin.jar -DgroupId=cn.itestin.cv -DartifactId=itestin -Dversion=170921 -Dpackaging=jar

[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building Maven Stub Project (No POM) 1
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-install-plugin:2.4:install-file (default-cli) @ standalone-pom ---
[INFO] Installing /Users/gebitang/Downloads/itestin.jar to /Users/gebitang/.m2/repository/cn/itestin/cv/itestin/170921/itestin-170921.jar
[INFO] Installing /var/folders/tw/pwg0lcfx4j79s8txc3z10dqm0000gn/T/mvninstall8572123810588485936.pom to /Users/gebitang/.m2/repository/cn/itestin/cv/itestin/170921/itestin-170921.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 0.828 s
[INFO] Finished at: 2017-09-22T10:14:08+08:00
[INFO] Final Memory: 5M/123M
[INFO] ------------------------------------------------------------------------
```

## 红色波浪线 Maven Failed to read artifact descriptor 导致dependency文件无法识别

[参考](https://stackoverflow.com/questions/6642146/maven-failed-to-read-artifact-descriptor)

在对应的根目录下执行 
```
# -U Forces a check for updated releases and snapshots on remote repositories
# run clean and install
mvn -U clean install
``` 
重新下载识别

为完成git merge，执行了`git clean -d -fx`清理后，整个工程中的配置都会消失——因会恢复到git仓库中的内容，当前目前下其他的内容都会被清理。

打开maven窗口，选择重新导入，可能出现pom文件中的dependency文件本地都有，但maven认为dependency都无法识别——显示红色波浪线。

此时执行该命令会强制重新安装。然后根据错误提示——如果本地有的jar包url仓库中没有会报错——进行对应操作即可。

## Failed to read artifact descriptor for xxx

对应的jar包无法获取到。1. 更改repository； 2. 或者不跟踪更高版本的jar包

## Dependencies version

[Reference](http://maven.apache.org/pom.html)
```
1.0: "Soft" requirement on 1.0 (just a recommendation, if it matches all other ranges for the dependency)
[1.0]: "Hard" requirement on 1.0
(,1.0]: x <= 1.0
[1.2,1.3]: 1.2 <= x <= 1.3
[1.0,2.0): 1.0 <= x < 2.0
[1.5,): x >= 1.5
(,1.0],[1.2,): x <= 1.0 or x >= 1.2; multiple sets are comma-separated
(,1.1),(1.1,): this excludes 1.1 (for example if it is known not to work in combination with this library)
```

## pom文件报错

dependency注意空格格式
