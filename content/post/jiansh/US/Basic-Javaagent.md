+++
title = "Basic-Javaagent"
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



[link on JianShu](https://www.jianshu.com/p/05cad98b91dd)

man java 

>      -javaagent:jarpath[=options]
>  Loads the specified Java programming language agent. For more information about instrumenting Java applications, see the java.lang.instrument package description in the Java API documentation at  http://docs.oracle.com/javase/8/docs/api/java/lang/instrument/package-summary.html  
文档说明部分介绍了基本概念

> Provides services that allow Java programming language agents to instrument programs running on the JVM.

[四步快速入门](https://stackoverflow.com/a/11898653/1087122)：
1. Implement a static premain (as an analogy to main) method, like this:
```
import java.lang.instrument.Instrumentation;
class Example {
    public static void premain(String args, Instrumentation inst) {
        ...
    }
}
```
2. Create a manifest file (say, manifest.txt) marking this class for pre-main execution. Its contents are:
```
Premain-Class: Example
```
3. Compile the class and package this class into a JAR archive:
```
javac Example.java
jar cmf manifest.txt yourAwesomeAgent.jar *.class
```
4. Execute your JVM with -javaagent parameter, like this:
```
java -javaagent:yourAwesomeAgent.jar -jar yourApp.jar
```

[Guide to Java Instrumentation](https://www.baeldung.com/java-instrumentation) 这个baeldung.com站点的google搜索权重很高，一般的Java相关的各种Guide都可以在[这里](https://www.baeldung.com/full_archive)看到。

编译完执行`java -javaagent:core-java-jvm.jar -jar core-java-jvm.jar StartMyAtmApplication 15000 2 3`出来下面的log表示上面的文章实现了Java Instrumentation的效果。

```
E:\>java -javaagent:core-java-jvm.jar -jar core-java-jvm.jar StartMyAtmApplication 15000 2 3
17:48:09.109 [main] INFO com.baeldung.instrumentation.agent.MyInstrumentationAgent - [Agent] In premain method
17:48:09.113 [main] INFO com.baeldung.instrumentation.agent.AtmTransformer - [Agent] Transforming class MyAtm
StartMyAtmApplication
17:48:09.212 [main] INFO com.baeldung.instrumentation.application.MyAtmApplication - [Application] Starting ATM application
17:48:11.213 [main] INFO com.baeldung.instrumentation.application.MyAtm - [Application] Successful Withdrawal of [2] units!
17:48:11.219 [main] INFO com.baeldung.instrumentation.application.MyAtm - [Application] Withdrawal operation completed in:2 seconds!
17:48:28.221 [main] INFO com.baeldung.instrumentation.application.MyAtm - [Application] Successful Withdrawal of [3] units!
17:48:28.221 [main] INFO com.baeldung.instrumentation.application.MyAtm - [Application] Withdrawal operation completed in:2 seconds!

```

[https://www.jacoco.org/jacoco/](https://www.jacoco.org/jacoco/)
[https://github.com/jacoco/jacoco](https://github.com/jacoco/jacoco)  
[测acoco 插桩的不同形式总结和踩坑记录](https://testerhome.com/topics/20632# "Jacoco 插桩的不同形式总结和踩坑记录")  

