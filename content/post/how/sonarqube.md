+++
title = "SonarQube"
description = "Everything about Sonar"
tags = [
]
date = "2020-04-23"
topics = [
    "sonar",
    "tech"
]
toc = true
+++

### SonarQube添加自定义规则

[官方文档之CUSTOM_RULES_101](https://github.com/SonarSource/sonar-java/blob/master/docs/CUSTOM_RULES_101.md)

[官方示例工程](https://github.com/SonarSource/sonar-custom-rules-examples/tree/master/java-custom-rules)

按照官方文档在官方示例工程上进行操作，可以编译通过，生成最终的jar包文件。

在`extentions/plugins`添加上自定义jar包后，启动时总是提示类似下面的错误——

```
jvm 1    | 2020.05.06 18:58:43 WARN  app[][o.e.t.n.Netty4Transport] exception caught on transport layer [[id: 0xa148553e, L:/127.0.0.1:3333 - R:/127.0.0.1:9001]], closing connection
jvm 1    | java.io.IOException: 远程主机强迫关闭了一个现有的连接。
jvm 1    |      at sun.nio.ch.SocketDispatcher.read0(Native Method)
jvm 1    |      at sun.nio.ch.SocketDispatcher.read(Unknown Source)
jvm 1    |      at sun.nio.ch.IOUtil.readIntoNativeBuffer(Unknown Source)
jvm 1    |      at sun.nio.ch.IOUtil.read(Unknown Source)
jvm 1    |      at sun.nio.ch.SocketChannelImpl.read(Unknown Source)
jvm 1    |      at io.netty.buffer.UnpooledUnsafeDirectByteBuf.setBytes(UnpooledUnsafeDirectByteBuf.java:433)
jvm 1    |      at io.netty.buffer.AbstractByteBuf.writeBytes(AbstractByteBuf.java:1100)
jvm 1    |      at io.netty.channel.socket.nio.NioSocketChannel.doReadBytes(NioSocketChannel.java:372)
jvm 1    |      at io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:123)
jvm 1    |      at io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:644)
jvm 1    |      at io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized
```

第一次定位发现log中有 `java.lang.IllegalStateException: Name of rule [repository=mycompany-java, key=MyFirstCustomRule] is empty`的提示，意识到Rule的定义写错了位置，注解写到Test文件上去了。重新打包后，这个错误提示不再出现。

第二次，依然有网络连接关闭的提示，但删除自定义的jar包之后就可以正常运行。

本地使用的是sonarqube7.6版本，猜测有可能是不兼容导致的。但这个错误提示不知道是怎样的逻辑出现的。

---

SonarQube提供了独立的log服务，命令行启动时输出的是sonar.log的内容。上述`Netty4Transport exception caught on transport layer`远程连接关闭的问题log属于es.log(elastic search)的部分。

而由于自定义规则的plugin导致的问题会在ce.log(Compute Engine)里有具体的提示（但这个内容居然不在命令行输出，导致一直对上面的远程连接失败摸不着头脑）。基本会有类似——

>org.sonar.updatecenter.common.exception.IncompatiblePluginVersionException: The plugin 'java' is in version 5.10.1.16922 whereas the plugin 'javacustom' requires a least a version 6.3.0.21585.

猜测没错，属于兼容性问题。关键的两个版本值——

- `sonar.version`的值为plugin支持的SonarQube最低版本（目前8.0以上对应的LTS版本为7.9；目前看7.6根7.7还是兼容的）。最新的[兼容矩阵](https://docs.sonarqube.org/latest/instance-administration/plugin-version-matrix/)在7.9以下版本里没有对应的说明。
- `sonarjava.version`对应plugin支持的版本，需要跟使用的SonarQube中安装的版本一致，管理员登录后，可以在`admin/marketplace?filter=installed`位置查看`SonarJava`的版本。

根据pom.xml的git log对其他的依赖进行对应的修改。

```
# current 
<sonar.version>8.2.0.32929</sonar.version>
<sonarjava.version>6.3.0.21585</sonarjava.version>

# modify
<sonar.version>7.7</sonar.version>
<sonarjava.version>5.10.1.16922</sonarjava.version>
```

回滚到上述版本后，目前的测试方法都失效了。——这是因为`org.sonar.java.checks.verifier.JavaCheckVerifier`的 `public static CheckVerifier newVerifier()`方法在2020/3/17加入的。SONARJAVA-3315 Unify CheckVerifiers accross the plugin (#2871)`e7f1493fa8572fa88ed528be37fc8debabc5e671`

此时本地再执行单元测试时，会报错——

>The rule 'f.newTuple20(Object, Object)' has already been defined somewhere in the grammar.

这两个地方都讨论了这个问题：   
- [项目issue #38](https://github.com/SonarSource/sonar-custom-rules-examples/issues/38)  
- [官方社区 create-java-custom-rule](https://community.sonarsource.com/t/create-java-custom-rule/10320/5)

本地测试无法正常运行（环境改的比较多了），但不影响打包操作。`mvn clean install`可以正常生成jar包。

放到对应的位置，重启SonarQube，可以看到这个自定义的plugin——

![](https://upload-images.jianshu.io/upload_images/3296949-6118488b181b12ff.png)

![](https://upload-images.jianshu.io/upload_images/3296949-de25702c52563d0f.png)

### 定制规则

[How to deactivate a rule in SonarQube?](https://sqa.stackexchange.com/questions/24734/how-to-deactivate-a-rule-in-sonarqube)


Follow below steps to disable any rule in SonarQube:

1.  Login by admin

2.  Go to quality profile & Select java/php profile [whichever is appropriate to you]

3.  Enter the rule as key and Search

4.  Uncheck the box which will inactive the rule

5.  Run Sonar runner command once again to verify the modifications are working properly

I have borrowed my answer from [here](http://javadeveloperchoiceno1.blogspot.com/2014/12/how-to-disable-rules-in-sonar.html)

---

Profile（质量配置）--> 选择语言 --> copy（复制出新的profile）--> 点击新的profile进行规则设置 （可通过搜索rule）

- SonarQube不允许修改默认的profile配置
- 需要先复制一份出来
- 在复制的profile里修改rule的规则（激活、未激活）
- 然后将复制的profile设置为默认选项

--- 

### SonarQube设置排除目录

Administration（配置）--> Configuration（配置）--> Analysis Scope (排除）

可对Code Coverage（代码覆盖率）、File（源码文件）、Duplication Excusions（重复）分别进行单独的设置

---

### 覆盖率统计不准确

本地使用ide执行的测试覆盖率**明显高于**线上使用jacoco动态统计的结果。

本地模拟线上环境执行也得出了相同的结果。

> 对应的测试用例的确执行了，但没有对应的统计。

分析来看，所有使用了 `MockitoJUnitRunner ` (来自[mockito-core/2.8.9](https://mvnrepository.com/artifact/org.mockito/mockito-core/2.8.9)，[github mockito](https://github.com/mockito/mockito))运行junit测试的用例都被忽略了

看起来跟[PowerMock](https://github.com/powermock/powermock/wiki/Code-coverage-with-JaCoCo)是相同的问题？
>We are going to replace Javassist with ByteBuddy (#727) and it should help to resolve this old issue. But right now there is NO WAY TO USE PowerMock with JaCoCo On-the-fly instrumentation. And no workaround to get code coverage in IDE.

>简单来说，大家都是动态搞字节码的，所以我（Jacoco）是没办法知道你（mockito）是怎么搞（bytecode）的。

**PowerMock** is an open source mocking library for the Java world. It extends the existing mocking frameworks, such as **[EasyMock](http://www.easymock.org/ "easymock")** and **[Mockito](https://code.google.com/p/mockito/ "mockito")**, to add even more powerful features to them.


[java-code-coverage-jacoco-and-mockitojunitrunner](https://jacoco.narkive.com/Sh86kg5E/java-code-coverage-jacoco-and-mockitojunitrunner)——
>JaCoCo does not play well with other tools that modify bytecode on-the-fly. JaCoCo needs to see the same classes at runtime than at reporting time (as the warning text says).
>
>A possible (but not really recommended) workaround is to use JaCoCo offline instrumentation.


[mockito issues 969](https://github.com/mockito/mockito/issues/969)问题里认为可以兼容，应该都是只offline模式，直接读取class文件，而不是on-the-fly模式。

[Stack Overflow jacoco-mockito-android-tests-zero-coverage-reported](https://stackoverflow.com/questions/46517471/jacoco-mockito-android-tests-zero-coverage-reported/46614216#46614216)

前两个问题链接看起来是有希望兼容的？第三个告诉你目前还不行哈 :(   
[https://github.com/jacoco/jacoco/issues/193](https://github.com/jacoco/jacoco/issues/193)  
[https://github.com/mockito/mockito/issues/757](https://github.com/mockito/mockito/issues/757)  
[https://github.com/jacoco/jacoco/issues/51](https://github.com/jacoco/jacoco/issues/51)  
