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

### 多模块项目包含war类型依赖

[JaCoCo-Multi-Module](../../jiansh/us/jacoco-multi-module/)强调了如果有子模块为pom类型时，需要将type进行明确说明。

但如果子模块为war类型时，如果不明示，会按照默认的jar类型去查找——不符合实际情况。

这个[问题](https://stackoverflow.com/questions/56071617/getting-coverage-from-war-dependency-in-multi-module-project)描述的是这一现象；

这个[官方jacoco issue #991](https://github.com/jacoco/jacoco/issues/991)提示的解决方案也是增加`<type>war</type>`。

增加了`<type>war</type>`之后，在执行`spring-boot:repackage`任务时，提示“找不到主类”——

>Execution default of goal org.springframework.boot:spring-boot-maven-plugin:1.5.4.RELEASE:repackage failed: Unable to find main class

**解决方案**：用来聚合单侧结果的模块打包方式制定为`<packaging>pom</packaging>`。[有效的打包类型包括](https://maven.apache.org/pom.html#packaging) `pom`, `jar`, `maven-plugin`, `ejb`, `war`, `ear`, `rar`

这样在执行`mvn clean install`时聚合模块不做打包处理。pom类型意味着只有`install`和`deploy`两个[阶段动作](https://maven.apache.org/ref/3.6.3/maven-core/default-bindings.html#Plugin_bindings_for_pom_packaging)

### Abstract Syntax Tree

[官方文档之CUSTOM_RULES_101](https://github.com/SonarSource/sonar-java/blob/master/docs/CUSTOM_RULES_101.md)之[Using syntax trees and API basics](https://github.com/SonarSource/sonar-java/blob/master/docs/CUSTOM_RULES_101.md#first-version-using-syntax-trees-and-api-basics)提到——

>Prior to running any rule, the SonarQube Java Analyzer parses a given Java code file and produces an equivalent data structure: the **Syntax Tree**. 

项目依赖的[JDT](https://github.com/SonarSource/sonar-java/blob/master/jdt/pom.xml)模块就来自Eclipse项目[eclipse.jdt.core](https://github.com/eclipse/eclipse.jdt.core)

```
# JDT module in sonar-java
<dependency>
        <groupId>org.eclipse.jdt</groupId>
        <artifactId>org.eclipse.jdt.core</artifactId>
        <version>3.20.0</version>
        ...
</dependency>
```

AST(Abstract Syntax Tree)在Eclipse的[入门文章](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/) 

![](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/images/workflow.png)

- 1. Java源码：可以是源码文件，也可以是一组字符串；
- 2. 使用 `org.eclipse.jdt.core.dom.ASTParser`类解析Java源码
- 3. 得到抽象语法树AST
- 4. 操作AST
- 5. 保存修改后的AST到源码

---

1.  *Java source*: To start off, you provide some source code to parse. This source code can be supplied as a Java file in your project or directly as a `char[]` that contains Java source
2.  *Parse*: The source code described at [1](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#workflow-legend-1) is parsed. All you need for this step is provided by the class `org.eclipse.jdt.core.dom.ASTParser`. See [the section called “Parsing source code”](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#sec-parsing-a-source-file "Parsing source code").
3.  The *Abstract Syntax Tree* is the result of step [2](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#workflow-legend-2). It is a tree model that entirely represents the source you provided in step [1](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#workflow-legend-1). If requested, the parser also computes and includes additional symbol resolved information called "[bindings](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#sec-bindings "Bindings")".
4.  *Manipulating the AST*: If the AST of point [3](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#workflow-legend-3) needs to be changed, this can be done in two ways:

    1.  By directly modifying the AST.
    2.  By noting the modifications in a separate protocol. This protocol is handled by an instance of `ASTRewrite`.

    See more in [the section called “How to Apply Changes”](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#sec-how-to-apply-changes "How to Apply Changes").
5.  *Writing changes back*: If changes have been made, they need to be applied to the source code that was provided by [1](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#workflow-legend-1). This is described in detail in [the section called “Write it down”](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#sec-write-it-down "Write it down").
6.  *`IDocument`*: Is a wrapper for the source code of step [1](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#workflow-legend-1) and is needed at point [5](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#workflow-legend-5)

AST的背景知识包括了`Java Model`，最新的内容参考[Java Model](https://help.eclipse.org/2020-03/topic/org.eclipse.jdt.doc.isv/guide/jdt_int_model.htm)

![](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/images/java-model-overview.png)

![](https://help.eclipse.org/2020-03/topic/org.eclipse.jdt.doc.isv/guide/images/openables.png)

![](https://help.eclipse.org/2020-03/topic/org.eclipse.jdt.doc.isv/guide/images/sourceelements.png)


关于JDT最新的[官网内容](https://help.eclipse.org/2020-03/topic/org.eclipse.jdt.doc.isv/guide/jdt_int_core.htm)，包括了

- [Java model](https://help.eclipse.org/2020-03/topic/org.eclipse.jdt.doc.isv/guide/jdt_int_model.htm)
- [Manipulating Java code](https://help.eclipse.org/2020-03/topic/org.eclipse.jdt.doc.isv/guide/jdt_api_manip.htm)
- [Eclipse JDT - Abstract Syntax Tree (AST) and the Java Model by Vogella](https://www.vogella.com/tutorials/EclipseJDT/article.html) 

[Creating CompilationUnit (AST) from file on disk](https://www.vogella.com/tutorials/EclipseJDT/article.html#creating-compilationunit-ast-from-file-on-disk)
```
private static void createFromDist() throws Exception {
    ASTParser parser = ASTParser.newParser(AST.JLS8);
    parser.setResolveBindings(true);
    parser.setStatementsRecovery(true);
    parser.setBindingsRecovery(true);
    parser.setKind(ASTParser.K_COMPILATION_UNIT);
    File resource = new File("./tests/resources/Snippet.java");
    Path sourcePath = Paths.get(resource.toURI());
    String sourceString = new String(Files.readAllBytes(sourcePath));
    char[] source = sourceString.toCharArray();
    parser.setSource(source);
    parser.setUnitName(sourcePath.toAbsolutePath().toString());
    CompilationUnit astRoot = (CompilationUnit) parser.createAST(null);
}
```

配合Eclipse版本的[ASTView](https://www.eclipse.org/jdt/ui/astview/index.php)，效果类似——

![](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/images/md-astview.png)

Idea上也有对应的[插件](https://plugins.jetbrains.com/plugin/9345-jdt-astview)，展示效果当然不如Eclipse官方的酷炫，也足够理解对应的概念。


中文资料参考——  

- [Java代码分析器(一): JDT入门](https://segmentfault.com/a/1190000000609246)
- [Java代码分析器(二): 使用DOM API操作抽象语法树](https://segmentfault.com/a/1190000000638838)


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
