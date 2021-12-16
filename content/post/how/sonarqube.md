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



## 忘记密码 admin

默认密码为`admin/admin`，修改过，忘记了。[官方指导操作instance-administration/security](https://docs.sonarqube.org/8.9/instance-administration/security/)

```sql
-- 使用对应的用户登录postgresSql
psql
-- 链接到对应的数据库
\c dbname
-- 执行恢复密码操作
update users set crypted_password='100000$t2h8AtNs1AlCHuLobDjHQTn9XppwTIx88UjqUm4s8RsfTuXQHSd/fpFexAnewwPsO6jGFQUv/24DnO55hY6Xew==', salt='k9x9eN127/3e/hf38iNiKwVfaVk=', hash_method='PBKDF2', reset_password='true', user_local='true' where login='admin';
```

## 插件开发 plugin develop

### SonarQube Kotlin插件

SQ的社区版本也支持[kotlin语言的扫描](https://www.sonarsource.com/kotlin/)。目前包含[43条规则](https://rules.sonarsource.com/kotlin)，通过插件方式实现，同时也支持[第三方](https://docs.sonarqube.org/latest/analysis/languages/kotlin/)的执行结果，包括AndroidLint(`sonar.androidLint.reportPaths`)或Detekt(`sonar.kotlin.detekt.reportPaths`)的报告结果。

官方针对Kotlin的扫描插件库[slang](https://github.com/SonarSource/slang/)项目下的`sonar-kotlin-plugin`

>SLang (SonarSource Language) is a framework to quickly develop code analyzers for SonarQube. SLang defines language agnostic AST. Using this AST we can develop simple syntax based rules. Then we use parser for real language to create this AST. Currently Kotlin, Ruby and Scala analyzers use this approach.

Kotlin官方在更早之前[计划开发Sonar插件](https://discuss.kotlinlang.org/t/sonarqube-support/3657/28)，但似乎一直没有进行？目前Kotlin的静态代码扫描由[detekt](https://github.com/detekt)提供，同时也提供了对应的sanar插件[sonar-kotlin](https://github.com/detekt/sonar-kotlin) 

实际上这两个都做为插件提供，前者为官方版本，后者为第三方版本。类似于针对Java语言的`sonar-java` VS. `sonar-pmd`。

使用开发针对Kotlin的自定义插件，可以从这两个项目入手调研。

### 获取自定义变量

例如sonarscanner执行扫描时传递的参数`-Dsonar.projectKey`可以在插件中使用`context.getProject().key()`获取到。

[Is it possible to get some customized parameters in Custom rule?](https://community.sonarsource.com/t/is-it-possible-to-get-some-customized-parameters-in-custom-rule/36460)

可以利用`Sensor`或`ProjectSensor`传递自定义变量。前者在每一个module执行时都会被调用；后者针对每个project只调用一次

实现的接口`void execute(SensorContext context);`中，从对象API`org.sonar.api.scanner.sensor.SensorContext`中可获取`Configuration`对象，顾名思义，包含了针对project级别所有的信息。例如，传递`-Dsonar.mycontxt.name=abc`的自定义参数，可以通过`sensorContext.config().get("sonar.mycontxt.name")`获取自定义的变量。

注意：  
- Sensor的`execute`方法可以指定执行的时间点，默认在文件扫描之后执行，可以通过注解`@Phase(name = Phase.Name.PRE)`让sensor在执行具体的代码扫描之前运行。
- 自定义的Sensor需要添加到plugin的配置中，例如`MySensor`可以使用`context.addExtension(MySensor.class);`添加到插件定义中

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

[ASTs - What are they and how to use them](https://www.twilio.com/blog/abstract-syntax-trees) 

[astexplorer：在线查看ast](https://astexplorer.net/)，[站点源码](https://github.com/fkling/astexplorer)

- Lexical Analysis 词法分析，转变为Token流
- Syntax Analysis 句法分析，转变为AST
- Code Generation 基于AST进行操作：读，写，改，打印等

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

~~本地测试无法正常运行（环境改的比较多了）~~，但不影响打包操作。`mvn clean install`可以正常生成jar包。

放到对应的位置，重启SonarQube，可以看到这个自定义的plugin——

![](https://upload-images.jianshu.io/upload_images/3296949-6118488b181b12ff.png)

![](https://upload-images.jianshu.io/upload_images/3296949-de25702c52563d0f.png)

6.3版本与5.10版本最大的不同是对源码解析时使用的Parser类发生变化：

- 5.10版本使用独立的自研项目[sslr](https://github.com/SonarSource/sslr)(SonarSource Language Recognizer)
- 6.3版本使用Eclipse的[eclipse.jdt.core](https://github.com/eclipse/eclipse.jdt.core)项目


[官方示例工程](https://github.com/SonarSource/sonar-custom-rules-examples/tree/master/java-custom-rules)中的测试代码还会用到sslr项目，只在`MyJavaRulesDefinitionTest`这个测试类中使用

```xml
    <dependency>
			<groupId>org.sonarsource.sslr</groupId>
			<artifactId>sslr-testing-harness</artifactId>
			<version>${sslr.version}</version>
			<scope>test</scope>
		</dependency>
```

>~~本地测试无法正常运行（环境改的比较多了）~~

还是pom环境问题，对比了[5.10.1.16922](https://github.com/SonarSource/sonar-java/archive/5.10.1.16922.tar.gz)版本的Sonar-Java之后，已解决。

- 将ssrl升级到`1.23`版本
- 保留guava的版本为`26.0-jre`(降低到`19.0`时，会出现解析报错问题`java.lang.NoSuchMethodError: com.google.common.base.Preconditions.checkNotNull(Ljava/lang/Object;Ljava/lang/String;Ljava/lang/Object;)Ljava/lang/Object;`)


#### 扫描规则源码分析

- 依赖sonar-java项目最新版本(6.3)的示例工程可以进行运行单元测试。
- 目前使用的sonarqube对应的sonar-java的版本(5.10)
- 修改后兼容当前使用的sonarqube版本的示例工程分支无法进行单侧

只能人肉分析代码，学习规则是怎样运行起来的。

测试驱动——

- 一个unit测试类提供入库；
- 一个测试源码作为测试数据
- 一个规则类（最终自定义并且打包生效的就是这个文件，不同的规则写不同的规则类即可）

测试暴露的接口就是：测试源码+自定义的Check类。其他`注册`内容示例工程已经打好架子，暂时忽略。

调用入口如下： 
```java
// 5.x (在6.x版本上这个方法deprecated,等价于下面的6.x的方法)
JavaCheckVerifier.verify("src/test/files/MyFirstCustomCheck.java",
                             new MyFirstCustomCheck());
// 6.x
JavaCheckVerifier.newVerifier()
                .onFile("src/test/files/MyFirstCustomCheck.java")
                .withCheck(new MyFirstCustomCheck())
                .verifyIssues();
 
```

##### 5.10版本

过程式编程，一堆静态方法构造出来需要调用的组件。

进入如下方法——

- check就是自定义的规则类；
- verifier用来检查最后的结果。例如，如果出现issue时，如何描述。

其他类关系——

- MyFirstCustomCheck   
  - extends `IssuableSubscriptionVisitor`  
      - extends `SubscriptionVisitor`  
          - implements `JavaFileScanner`  Common interface for all checks analyzing a java file.
             - extends `JavaCheck` (empty interface)

- `ExpectedIssueCollector`
  - extends `SubscriptionVisitor`
     - implements `JavaFileScanner`

- `JavaCheckVerifier` extends `CheckVerifier`

```java
// line 261
private static void scanFile(String filename, JavaFileScanner check, JavaCheckVerifier javaCheckVerifier, Collection<File> classpath, boolean withSemantic) {
    //又添加了一个JavaScanner
    JavaFileScanner expectedIssueCollector = new ExpectedIssueCollector(javaCheckVerifier);
    VisitorsBridgeForTests visitorsBridge;
    File file = new File(filename);
    // 构造上下文关系context，所以放到了检查结果的CheckVerfiler中做静态方法
    SonarComponents sonarComponents = CheckVerifier.sonarComponents(file);
    if (withSemantic) {
        //进入这里初始化Bridge，
      visitorsBridge = new VisitorsBridgeForTests(Lists.newArrayList(check, expectedIssueCollector), Lists.newArrayList(classpath), sonarComponents);
    } else {
      visitorsBridge = new VisitorsBridgeForTests(Lists.newArrayList(check, expectedIssueCollector), sonarComponents);
    }
    //终于进入干活的地方，
    JavaAstScanner.scanSingleFileForTests(file, visitorsBridge, javaCheckVerifier.javaVersion);
    VisitorsBridgeForTests.TestJavaFileScannerContext testJavaFileScannerContext = visitorsBridge.lastCreatedTestContext();
    if (testJavaFileScannerContext == null) {
      Fail.fail("Semantic was required but it was not possible to create it. Please checks the logs to find out the reason.");
    }
    javaCheckVerifier.checkIssues(testJavaFileScannerContext.getIssues(), javaCheckVerifier.providedJavaVersion);
  }
```

新new一个scanner进行scan动作，对所有的文件进行扫描。其中的Parser用来将文件字符串内容读取为语法树结构AST？（`JavaParser`这块内容有点晕）
```java
  @VisibleForTesting
  public static void scanSingleFileForTests(File file, VisitorsBridge visitorsBridge, JavaVersion javaVersion) {
    if (!file.isFile()) {
      throw new IllegalArgumentException("File '" + file + "' not found.");
    }
    // 创建的Parser，用来将文件内容
    JavaAstScanner astScanner = new JavaAstScanner(JavaParser.createParser(), null);
    visitorsBridge.setJavaVersion(javaVersion);
    astScanner.setVisitorBridge(visitorsBridge);
    astScanner.scan(Collections.singleton(file));
  }
```

核心方法 
```java
private void simpleScan(File file) {
    //...
    try {
      String fileContent = getFileContent(file);
      Tree ast;
      if(fileContent.isEmpty()) {
        ast = parser.parse(file);
      } else {
        ast = parser.parse(fileContent);
      }
      visitor.visitFile(ast);
    } 
  }
```

```java
/**
 * Common interface for all checks analyzing a java file.
 */
@Beta
public interface JavaFileScanner extends JavaCheck {

  /**
   * Method called after parsing and semantic analysis has been done on file.
   * @param context Context of analysis containing the parsed tree.
   */
  void scanFile(JavaFileScannerContext context);

}
```

---

##### 6.3版本

6.3版本的`newVerifier()`对应的就是`InternalCheckVerifier`，builder模式，最终进入方法`verifyAll()`

- InternalCheckVerifier 
   - implements CheckVerifier

CheckVerifier从5.x的抽象类变为6.x的纯接口类

```java
private void verifyAll() {
    List<JavaFileScanner> visitors = new ArrayList<>(checks);
    if (withoutSemantic && expectations.expectNoIssues()) {
      visitors.add(expectations.noEffectParser());
    } else {
      visitors.add(expectations.parser());
    }
    SonarComponents sonarComponents = sonarComponents();
    VisitorsBridgeForTests visitorsBridge;
    if (withoutSemantic) {
      visitorsBridge = new VisitorsBridgeForTests(visitors, sonarComponents);
    } else {
      List<File> actualClasspath = classpath == null ? DEFAULT_CLASSPATH : classpath;
      visitorsBridge = new VisitorsBridgeForTests(visitors, actualClasspath, sonarComponents);
    }

    JavaAstScanner astScanner = new JavaAstScanner(sonarComponents);
    visitorsBridge.setJavaVersion(javaVersion == null ? DEFAULT_JAVA_VERSION : javaVersion);
    astScanner.setVisitorBridge(visitorsBridge);
    astScanner.scan(files);

    VisitorsBridgeForTests.TestJavaFileScannerContext testJavaFileScannerContext = visitorsBridge.lastCreatedTestContext();
    checkIssues(testJavaFileScannerContext.getIssues());
  }
```
核心方法也没有大的变化 

```java
private void simpleScan(InputFile inputFile) {
    ...
    try {
      String fileContent = inputFile.contents();
      final String version;
      if (visitor.getJavaVersion() == null || visitor.getJavaVersion().asInt() < 0) {
        version = /* default */ JParser.MAXIMUM_SUPPORTED_JAVA_VERSION;
      } else {
        version = Integer.toString(visitor.getJavaVersion().asInt());
      }
      //使用jdt的解析服务
      Tree ast = JParser.parse(
        version,
        inputFile.filename(),
        fileContent,
        visitor.getClasspath()
      );
      visitor.visitFile(ast);
    }
    ...
  }
```

核心流程都是将文件解析成`Tree` 

5.x的版本使用了 SonarSource自己的`sslr`工程——

- [解析表达式语法(Parsing Expression Grammars)](https://www.cnblogs.com/freeblues/p/8436523.html)  
- [An introduction to Parsing Expression Grammars with LPeg](https://leafo.net/guides/parsing-expression-grammars.html) 
- [wiki: Parsing Expression Grammars](https://en.wikipedia.org/wiki/Parsing_expression_grammar)
- [SonarSource Language Recognizer](https://github.com/SonarSource/sslr)   

>If you want to start working with SSLR, you must be familiar with the following standard concepts : `Lexical Analysis`, `Parsing Expression Grammar` and `AST`(Abstract Syntax Tree). 

```java
//5.x JavaParser.createParser().parse()
public static ActionParser<Tree> createParser() {
    return new JavaParser(JavaLexer.createGrammarBuilder(),
      JavaGrammar.class,
      new TreeFactory(),
      new JavaNodeBuilder(),
      JavaLexer.COMPILATION_UNIT);
}

// parser.parse()
private N parse(Input input) {
    //this.parseRunner = new ParseRunner(b.build().getRootRule());
    ParsingResult result = parseRunner.parse(input.input());
    ...
    N node = syntaxTreeCreator.create(result.getParseTreeRoot(), input);
    if (node instanceof AstNode) {
      astNodeSanitizer.sanitize((AstNode) node);
    }
    return node;
  }
```


6.x版本则使用了Eclipse的[`jdt`项目](https://git.eclipse.org/c/jdt/eclipse.jdt.core.git/) 

```java
public static CompilationUnitTree parse(
    String version,
    String unitName,
    String source,
    List<File> classpath
  ) {
    //   
    ASTParser astParser = ASTParser.newParser(AST.JLS13);
    ...
    CompilationUnit astNode;
    try {
      astNode = (CompilationUnit) astParser.createAST(null);
    } catch (Exception e) {
      LOG.error("ECJ: Unable to parse file", e);
      throw new RecognitionException(-1, "ECJ: Unable to parse file.", e);
    }
    ...
    // 做格式状态？
    JParser converter = new JParser();
    converter.sema = new JSema(astNode.getAST());
    converter.compilationUnit = astNode;
    converter.tokenManager = new TokenManager(lex(version, unitName, sourceChars), source, new DefaultCodeFormatterOptions(new HashMap<>()));

    JavaTree.CompilationUnitTreeImpl tree = converter.convertCompilationUnit(astNode);
    tree.sema = converter.sema;

    ASTUtils.mayTolerateMissingType(astNode.getAST());

    setParents(tree);
    return tree;
  }
```

- 5.x版本使用sslr将源码直接解析后识别为sonar-java识别的Tree对象；
- 6.x版本借用jdt将源码解析为AST，再转化为sonar-java识别的Tree对象；

- 最后大家都调用`visitor.visitFile(Tree parsedTree);`来进行具体的判断

#### visitFile规则源码分析

VisitorsBridge的构造：

##### 6.3版本

构造函数里初始化这样几个组件： 

- Scanners，可以理解为规则集合
- classPath，略
- sonarComponents：上下文关系（作用是什么？）
- SymbolicExecutionMode：包含三种模式：`DISABLED`,`ENABLED`, `ENABLED_WITHOUT_X_FILE`(影响的范围是？) ;

```java
public VisitorsBridge(Iterable<? extends JavaCheck> visitors, List<File> projectClasspath,
                        @Nullable SonarComponents sonarComponents, SymbolicExecutionMode symbolicExecutionMode,
                        @Nullable JavaFileScanner analysisIssueFilter) {
    this.allScanners = new ArrayList<>();
    for (Object visitor : visitors) {
      if (visitor instanceof JavaFileScanner) {
        allScanners.add((JavaFileScanner) visitor);
      }
    }
    this.analysisIssueFilter = analysisIssueFilter;
    this.classpath = projectClasspath;
    this.executableScanners = allScanners.stream().filter(IS_ISSUABLE_SUBSCRIPTION_VISITOR.negate()).collect(Collectors.toList());
    this.issuableSubscriptionVisitorsRunner = new IssuableSubsciptionVisitorsRunner(allScanners);
    this.sonarComponents = sonarComponents;
    this.classLoader = ClassLoaderBuilder.create(projectClasspath);
    this.symbolicExecutionEnabled = symbolicExecutionMode.isEnabled();
    this.behaviorCache = new BehaviorCache(classLoader, symbolicExecutionMode.isCrossFileEnabled());
  }
```

根据scanner过滤出来几类`runner`，在最终的`visitor.visitFile(Tree parsedTree);`里分别进行调用，每个runner会传递一个上下文对象`DefaultJavaFileScannerContext`

不同的scanner实现了不同的接口，runner中执行这个接口的实现，`JavaFileScanner`接口的`scanFile`方法

对于自定义的`MyFirstCustomCheck`checker来说，最终执行的是`issuableSubscriptionVisitorsRunner.run(javaFileScannerContext);`即——

```java
private void visit(Tree tree) throws CheckFailureException {
    Kind kind = tree.kind();
    List<SubscriptionVisitor> subscribed = checks.getOrDefault(kind, Collections.emptyList());
    Consumer<SubscriptionVisitor> callback;
    boolean isToken = (kind == Tree.Kind.TOKEN);
    if (isToken) {
    callback = s -> s.visitToken((SyntaxToken) tree);
    } else {
    callback = s -> s.visitNode(tree);
    }
    forEach(subscribed, callback);
    if (isToken) {
    forEach(checks.getOrDefault(Tree.Kind.TRIVIA, Collections.emptyList()), s -> ((SyntaxToken) tree).trivias().forEach(s::visitTrivia));
    } else {
    visitChildren(tree);
    }
    if(!isToken) {
    forEach(subscribed, s -> s.leaveNode(tree));
    }
}

// 
private final void forEach(Collection<SubscriptionVisitor> visitors, Consumer<SubscriptionVisitor> callback) throws CheckFailureException {
    for (SubscriptionVisitor visitor : visitors) {
    runScanner(() -> callback.accept(visitor), visitor);
    }
}
```

##### 5.10版本

构造函数——

```java
public VisitorsBridge(Iterable visitors, List<File> projectClasspath, @Nullable SonarComponents sonarComponents, SymbolicExecutionMode symbolicExecutionMode) {
    this.allScanners = new ArrayList<>();
    for (Object visitor : visitors) {
        if (visitor instanceof JavaFileScanner) {
        allScanners.add((JavaFileScanner) visitor);
        }
    }
    // private static Predicate<JavaFileScanner> isIssuableSubscriptionVisitor = s -> s instanceof IssuableSubscriptionVisitor;
    this.executableScanners = allScanners.stream().filter(isIssuableSubscriptionVisitor.negate()).collect(Collectors.toList());
    this.scannerRunner = new ScannerRunner(allScanners);
    this.sonarComponents = sonarComponents;
    this.classLoader = ClassLoaderBuilder.create(projectClasspath);
    this.symbolicExecutionEnabled = symbolicExecutionMode.isEnabled();
    this.behaviorCache = new BehaviorCache(classLoader, symbolicExecutionMode.isCrossFileEnabled());
}
```

实际的调用方法`visitor.visitFile(Tree parsedTree);`

```java
public void visitFile(@Nullable Tree parsedTree) {
    //...
    JavaFileScannerContext javaFileScannerContext = createScannerContext(tree, semanticModel, sonarComponents, fileParsed);
    // Symbolic execution checks
    if (symbolicExecutionEnabled && isNotJavaLangOrSerializable(PackageUtils.packageName(tree.packageDeclaration(), "/"))) {
      runScanner(javaFileScannerContext, new SymbolicExecutionVisitor(executableScanners, behaviorCache), AnalysisError.Kind.SE_ERROR);
      behaviorCache.cleanup();
    }
    executableScanners.forEach(scanner -> runScanner(javaFileScannerContext, scanner, AnalysisError.Kind.CHECK_ERROR));
    //
    scannerRunner.run(javaFileScannerContext);
    if (semanticModel != null) {
      classesNotFound.addAll(semanticModel.classesNotFound());
    }
  }
```

同6.3版本相似的逻辑。

#### 使用其他版本

官方版本提供了不同的[release版本](https://github.com/SonarSource/sonar-java/tags?after=5.13.1.18282)，可以下载对应的[5.10.1.16922](https://github.com/SonarSource/sonar-java/archive/5.10.1.16922.tar.gz)版本。

SonarQube强调TDD开发模式，测试代码很齐全，针对单一源码的测试代码可以直接在IDE中运行，例如`org.sonar.java.ast.JavaAstScannerTest`下的`comments`方法。

```java
@Test
  public void comments() {
    File file = new File("src/test/files/metrics/Comments.java");
    DefaultInputFile resource = new TestInputFileBuilder("", "src/test/files/metrics/Comments.java").build();
    fs.add(resource);
    NoSonarFilter noSonarFilter = mock(NoSonarFilter.class);
    JavaAstScanner.scanSingleFileForTests(file, new VisitorsBridge(new Measurer(fs, context, noSonarFilter)));
    verify(noSonarFilter).noSonarInFile(resource, ImmutableSet.of(15));
  }
```

事实上，对于测试包`org.sonar.java.checks`下的测试类都可以直接执行单元测试。

这样就可以针对使用的版本直接开发自定义规则并进行验证了

### 规则开发示例

[Java custom rule writing without exploring the Syntax Tree](https://community.sonarsource.com/t/java-custom-rule-writing-without-exploring-the-syntax-tree/550/2)

>When implementing a Java custom rule, nothing forces you to use a `BaseTreeVisitor` or `IssuableSubscriptionVisitor`, you can perfectly only implement the `JavaFileScanner` interface, which will give you access to the content of the file:

```java
package org.sonar.samples.java.checks;

import org.sonar.check.Rule;
import org.sonar.plugins.java.api.JavaFileScanner;
import org.sonar.plugins.java.api.JavaFileScannerContext;

@Rule(key = "MyCheck")
public class MyCheck implements JavaFileScanner {

  @Override
  public void scanFile(JavaFileScannerContext context) {
    context.getFileContent(); // to retrieve the full content of the file as a String
    context.getFileLines();   // to retrieve the content of each lines of the file, as a String
  }
}
```

`DefaultJavaFileScannerContext`传递出去后，上报issue时，

```java
  context.reportIssue(this, idf, String.format("Avoid using annotation @%s", name));

  @Override
  public void reportIssue(JavaCheck javaCheck, Tree tree, String message) {
    //这个方法如何调用到了——VisitorsBridgeForTests的对应方法？
    reportIssue(javaCheck, tree, message, ImmutableList.of(), null);
  }
```

- 编译自定义规则，将产出物放到`\extensions\plugins`目录下
- 本地启动SonarQube（生成登录的token）
- 准备好测试工程（包含源码+编译后的class文件内容）
- 执行SonarScanner。`path\to\sonar-scanner.bat  -Dsonar.host.url=http://127.0.0.1:9000  -Dsonar.login=generated_token_value -Dsonar.projectKey=local-loop-prj  -Dsonar.sourceEncoding=UTF-8 -Dsonar.sources=src -Dsonar.java.binaries=target/classes`
- 在SonarQube上查看执行结果 

** sonar-scanner执行的最小参数：url、login、projectKey、sources、binaries。

写了一个**最基础**版本的自定义规则：`不允许在循环中对容器进行赋值`。忽略算法实现- -||

```java
package org.sonar.samples.java.checks;

import org.sonar.check.Priority;
import org.sonar.check.Rule;
import org.sonar.plugins.java.api.JavaFileScanner;
import org.sonar.plugins.java.api.JavaFileScannerContext;
import org.sonar.plugins.java.api.semantic.Type;
import org.sonar.plugins.java.api.tree.BaseTreeVisitor;
import org.sonar.plugins.java.api.tree.BlockTree;
import org.sonar.plugins.java.api.tree.ExpressionStatementTree;
import org.sonar.plugins.java.api.tree.ForStatementTree;
import org.sonar.plugins.java.api.tree.MethodTree;
import org.sonar.plugins.java.api.tree.StatementTree;
import org.sonar.plugins.java.api.tree.Tree;
import org.sonar.plugins.java.api.tree.VariableTree;
import org.sonar.samples.java.RulesList;

import java.util.List;

@Rule(
        key = "ForLoopAssignCheck",
        name = "assign container value in for loop",
        description = "don't try to assign or update container value in any loop operation.",
        priority = Priority.MAJOR,
        tags = {"custom"})
public class ForLoopAssignCheck extends BaseTreeVisitor implements JavaFileScanner {


    private JavaFileScannerContext context;


    @Override
    public void scanFile(JavaFileScannerContext context) {
        this.context = context;
        scan(context.getTree());
    }

    @Override
    public void visitMethod(MethodTree tree) {
        BlockTree block = tree.block();
        if(block != null) {
            boolean containerFound = false;
            String containerVar = "";
            List<StatementTree> statements = block.body();
            for(StatementTree s : statements) {
                if(s.is(Tree.Kind.VARIABLE)) {
                    VariableTree vt = ((VariableTree)s);
                    Type type = vt.type().symbolType();
                    if(isSubTypeOfSetOrMap(type)) {
                        containerFound = true;
                        containerVar = vt.simpleName().name();
                    }
                }

                if(s.is(Tree.Kind.FOR_STATEMENT)) {
                    ForStatementTree ft = (ForStatementTree)s;
                    if(ft.statement().is(Tree.Kind.BLOCK)) {
                        List<StatementTree> body = ((BlockTree)ft.statement()).body();

                        for(StatementTree state : body) {
                            if(state.is(Tree.Kind.EXPRESSION_STATEMENT)) {
                                ExpressionStatementTree et = (ExpressionStatementTree)state;

                                String var = et.expression().firstToken().text();
                                if(var.length() > 0 && var.equals(containerVar)) {
                                    context.reportIssue(this, et, String.format("avoid assigning container %s in loop.", var));
                                }
                            }
                        }
                    }
                }
            }

        }
        super.visitMethod(tree);
    }

    private boolean isSubTypeOfSetOrMap(Type type) {
        return type.isSubtypeOf("java.util.Set") || type.isSubtypeOf("java.util.Map");
    }
}

```

![](https://upload-images.jianshu.io/upload_images/3296949-9bc1bd3092103c77.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## SonarQube debug

### SonarScanner是如何执行的


{{< fluid_imgs
  "pure-u-1-1|https://upload-images.jianshu.io/upload_images/3296949-c45ace8ed9378837.png|stack info|https://upload-images.jianshu.io/upload_images/3296949-c45ace8ed9378837.png"
>}}

本地复现  [Can not execute Checkstyle](https://community.sonarsource.com/t/all-files-are-still-parsed-by-check-style-parser-even-if-there-is-none-of-any-activated-checkstyle-rule/36801)

```
ERROR: Error during SonarQube Scanner execution
java.lang.IllegalStateException: Can not execute Checkstyle
        at org.sonar.plugins.checkstyle.CheckstyleExecutor.executeWithClassLoader(CheckstyleExecutor.java:110)
        at org.sonar.plugins.checkstyle.CheckstyleExecutor.execute(CheckstyleExecutor.java:78)
        at org.sonar.plugins.checkstyle.CheckstyleSensor.execute(CheckstyleSensor.java:42)
        at org.sonar.scanner.sensor.AbstractSensorWrapper.analyse(AbstractSensorWrapper.java:48)
        at org.sonar.scanner.sensor.ModuleSensorsExecutor.execute(ModuleSensorsExecutor.java:85)
        at org.sonar.scanner.sensor.ModuleSensorsExecutor.lambda$execute$1(ModuleSensorsExecutor.java:59)
        at org.sonar.scanner.sensor.ModuleSensorsExecutor.withModuleStrategy(ModuleSensorsExecutor.java:77)
        at org.sonar.scanner.sensor.ModuleSensorsExecutor.execute(ModuleSensorsExecutor.java:59)
        at org.sonar.scanner.scan.ModuleScanContainer.doAfterStart(ModuleScanContainer.java:82)
        at org.sonar.core.platform.ComponentContainer.startComponents(ComponentContainer.java:137)
        at org.sonar.core.platform.ComponentContainer.execute(ComponentContainer.java:123)
        at org.sonar.scanner.scan.ProjectScanContainer.scan(ProjectScanContainer.java:388)
        at org.sonar.scanner.scan.ProjectScanContainer.scanRecursively(ProjectScanContainer.java:384)
        at org.sonar.scanner.scan.ProjectScanContainer.doAfterStart(ProjectScanContainer.java:353)
        at org.sonar.core.platform.ComponentContainer.startComponents(ComponentContainer.java:137)
        at org.sonar.core.platform.ComponentContainer.execute(ComponentContainer.java:123)
        at org.sonar.scanner.bootstrap.GlobalContainer.doAfterStart(GlobalContainer.java:144)
        at org.sonar.core.platform.ComponentContainer.startComponents(ComponentContainer.java:137)
        at org.sonar.core.platform.ComponentContainer.execute(ComponentContainer.java:123)
        at org.sonar.batch.bootstrapper.Batch.doExecute(Batch.java:72)
        at org.sonar.batch.bootstrapper.Batch.execute(Batch.java:66)
        at org.sonarsource.scanner.api.internal.batch.BatchIsolatedLauncher.execute(BatchIsolatedLauncher.java:46)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
        at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
        at java.base/java.lang.reflect.Method.invoke(Unknown Source)
        at org.sonarsource.scanner.api.internal.IsolatedLauncherProxy.invoke(IsolatedLauncherProxy.java:60)
        at com.sun.proxy.$Proxy0.execute(Unknown Source)
        at org.sonarsource.scanner.api.EmbeddedScanner.doExecute(EmbeddedScanner.java:189)
        at org.sonarsource.scanner.api.EmbeddedScanner.execute(EmbeddedScanner.java:138)
        at org.sonarsource.scanner.cli.Main.execute(Main.java:112)
        at org.sonarsource.scanner.cli.Main.execute(Main.java:75)
        at org.sonarsource.scanner.cli.Main.main(Main.java:61)
Caused by: com.puppycrawl.tools.checkstyle.api.CheckstyleException: Exception was thrown while processing E:\download\1216\eipis-user\service\src\main\java\com\democom\finance\eipis\user\service\impl\UserServiceImpl.java
        at com.puppycrawl.tools.checkstyle.Checker.processFiles(Checker.java:311)
        at com.puppycrawl.tools.checkstyle.Checker.process(Checker.java:221)
        at org.sonar.plugins.checkstyle.CheckstyleExecutor.executeWithClassLoader(CheckstyleExecutor.java:103)
        ... 32 more
Caused by: com.puppycrawl.tools.checkstyle.api.CheckstyleException: IllegalStateException occurred while parsing file E:\download\1216\eipis-user\service\src\main\java\com\democom\finance\eipis\user\service\impl\UserServiceImpl.java.
        at com.puppycrawl.tools.checkstyle.JavaParser.parse(JavaParser.java:120)
        at com.puppycrawl.tools.checkstyle.TreeWalker.processFiltered(TreeWalker.java:149)
        at com.puppycrawl.tools.checkstyle.api.AbstractFileSetCheck.process(AbstractFileSetCheck.java:87)
        at com.puppycrawl.tools.checkstyle.Checker.processFile(Checker.java:337)
        at com.puppycrawl.tools.checkstyle.Checker.processFiles(Checker.java:298)
        ... 34 more
Caused by: java.lang.IllegalStateException: E:\download\1216\eipis-user\service\src\main\java\com\democom\finance\eipis\user\service\impl\UserServiceImpl.java:790:1: expecting RCURLY, found 'null'
        at com.puppycrawl.tools.checkstyle.JavaParser$1.reportError(JavaParser.java:108)
        at com.puppycrawl.tools.checkstyle.grammar.GeneratedJavaRecognizer.typeDefinition(GeneratedJavaRecognizer.java:424)
        at com.puppycrawl.tools.checkstyle.grammar.GeneratedJavaRecognizer.compilationUnit(GeneratedJavaRecognizer.java:212)
        at com.puppycrawl.tools.checkstyle.JavaParser.parse(JavaParser.java:114)
        ... 38 more
Caused by: E:\download\1216\eipis-user\service\src\main\java\com\democom\finance\eipis\user\service\impl\UserServiceImpl.java:790:1: expecting RCURLY, found 'null'
        at antlr.Parser.match(Parser.java:211)
        at com.puppycrawl.tools.checkstyle.grammar.GeneratedJavaRecognizer.compoundStatement(GeneratedJavaRecognizer.java:4555)
        at com.puppycrawl.tools.checkstyle.grammar.GeneratedJavaRecognizer.field(GeneratedJavaRecognizer.java:3158)
        at com.puppycrawl.tools.checkstyle.grammar.GeneratedJavaRecognizer.classBlock(GeneratedJavaRecognizer.java:3415)
        at com.puppycrawl.tools.checkstyle.grammar.GeneratedJavaRecognizer.classDefinition(GeneratedJavaRecognizer.java:646)
        at com.puppycrawl.tools.checkstyle.grammar.GeneratedJavaRecognizer.typeDefinitionInternal(GeneratedJavaRecognizer.java:561)
        at com.puppycrawl.tools.checkstyle.grammar.GeneratedJavaRecognizer.typeDefinition(GeneratedJavaRecognizer.java:402)
        ... 40 more
```

默认插件sonar-java中注册了` ExternalReportExtensions.define(context);`针对三大插件的Sensor——

```
context.addExtension(CheckstyleSensor.class);
context.addExtension(PmdSensor.class);
context.addExtension(SpotBugsSensor.class);
```

默认执行阶段是在对文件本地的扫描扫描之后调用执行——

```
INFO: 26/26 source files have been analyzed
INFO: Sensor SonarCSS Rules [cssfamily] (done) | time=4249ms
INFO: Sensor PmdSensor [pmd]
INFO: Sensor PmdSensor [pmd] (done) | time=0ms
INFO: Sensor EndSensor [javacustom]
INFO: Sensor EndSensor [javacustom] (done) | time=1ms
INFO: Sensor JaCoCo XML Report Importer [jacoco]
WARN: No coverage report can be found with sonar.coverage.jacoco.xmlReportPaths='ut/target/site/jacoco-aggregate/jacoco.xml'. Using default locations: target/site/jacoco/jacoco.xml,target/site/jacoco-it/jacoco.xml,build/reports/jacoco/test/jacocoTestReport.xml
INFO: No report imported, no coverage information will be imported by JaCoCo XML Report Importer
INFO: Sensor JaCoCo XML Report Importer [jacoco] (done) | time=5ms
INFO: Sensor SurefireSensor [java]
INFO: parsing [E:\download\1216\eipis-user\target\surefire-reports]
INFO: Sensor SurefireSensor [java] (done) | time=1ms
INFO: Sensor JavaXmlSensor [java]
INFO: Sensor JavaXmlSensor [java] (done) | time=3ms
INFO: Sensor HTML [web]
INFO: Sensor HTML [web] (done) | time=237ms
INFO: Sensor CheckstyleSensor [checkstyle]
INFO: Checkstyle output report: E:\download\1216\eipis-user\.scannerwork\checkstyle-result.xml
INFO: Checkstyle configuration: E:\download\1216\eipis-user\.scannerwork\checkstyle.xml
INFO: Checkstyle charset: UTF-8
INFO: ------------------------------------------------------------------------
INFO: EXECUTION FAILURE
```

~~这样还是不能倒推到报错的堆栈信息上啊？-_-|| 看起来还得加上 `https://github.com/SonarSource/sonarqube/`这里的代码才能梳理出来完整逻辑。先缓缓~~ 

看起来只要安装了对应的插件，即使没有启用对应的规则，也会触发一下检查操作。根据上面问题的官方答复建议，删除`checkStyle`插件后，问题消失。

但这是个好问题，可以借机调研一下整个sonarqube的插件的执行逻辑。**跟踪堆栈信息对应的源码**

重新读了一下执行逻辑，不需要sonarqube端的代码。`sonar-scanner-cli`只是简单做了一个壳，执行的逻辑依赖`sonar-scanner-api`。跟踪以下调用链路即可，包含了这两个工程的代码。

核心是通过proxy的方式，调用了`org.sonarsource.scanner.api.internal.batch.BatchIsolatedLauncher`类的execute方法。`IsolatedLauncherProxy`是代理执行的`InvocationHandler`

```
at org.sonarsource.scanner.api.internal.IsolatedLauncherProxy.invoke(IsolatedLauncherProxy.java:60)
at com.sun.proxy.$Proxy0.execute(Unknown Source)
at org.sonarsource.scanner.api.EmbeddedScanner.doExecute(EmbeddedScanner.java:189)
at org.sonarsource.scanner.api.EmbeddedScanner.execute(EmbeddedScanner.java:138)
at org.sonarsource.scanner.cli.Main.execute(Main.java:112)
at org.sonarsource.scanner.cli.Main.execute(Main.java:75)
at org.sonarsource.scanner.cli.Main.main(Main.java:61)
```

根据使用scanner执行一次正常的操作，结合log输出更方便理解。`sonar-scanner-cli`工程就是发布的sonar-scanner工具的源码，只是在发布时，根据不同的平台增加了一个`jre`的执行环境。

### How to get the value of the arguments of the sonar tree

获取某个节点的变量，`tree.arguments()`，返回Arguments， `public interface Arguments extends ListTree<ExpressionTree> ` ，所以每个变量对应一个`ExprssionTree`，根据不同的类型解析表达式即可。

sonar中将语法树重新解析为 `public interface Tree `，枚举出`enum Kind implements GrammarRuleKey `(目前)119中不同的`Kind`。

原则上，针对不同的类型(可以作为变量的Kind)，做不同的处理即可。

例如，基础类型可以直接转换为`LiteralTree `， 获取value方法的值即可。


[How to get the value of the arguments of the sonar tree](https://stackoverflow.com/questions/38862873/how-to-get-the-value-of-the-arguments-of-the-sonar-tree) relative

[Custom Rule for Sonar Java](https://community.sonarsource.com/t/custom-rule-for-sonar-java/521) similar 

```
public class YourRule extends IssuableSubscriptionVisitor {

  @Override
  public List<Tree.Kind> nodesToVisit() {
    // Register to the kind of nodes you want to be called upon visit.
    return ImmutableList.of(
        Tree.Kind.MEMBER_SELECT,
        Tree.Kind.METHOD_INVOCATION);
  }

  @Override
  public void visitNode(Tree tree) {
    if (tree.is(Tree.Kind.MEMBER_SELECT)) {
      MemberSelectExpressionTree mset = (MemberSelectExpressionTree) tree;
      System.out.println(mset);
    } else if (tree.is(Tree.Kind.METHOD_INVOCATION)) {
      MethodInvocationTree mit = (MethodInvocationTree) tree;
      System.out.println(mit);
    }
  }
}
```

[Get parent of method invocation - Java](https://community.sonarsource.com/t/get-parent-of-method-invocation-java/10073) similar scenario 

[How to get value of a variable in SonarQube (custom-rules )?](https://stackoverflow.com/questions/50039501/how-to-get-value-of-a-variable-in-sonarqube-custom-rules) not relative

[Writing a new custom Java Rule to find a method invocation](https://community.sonarsource.com/t/writing-a-new-custom-java-rule-to-find-a-method-invocation/1878) no result

### 编译SonarQube 

同样需要JDK11+环境；注意gradle的本地zip包指定；设置gradle的代理

[sonarqube](https://github.com/SonarSource/sonarqube)使用gradle进行编译

为导入ide进行代码查看，可执行`./gradlew ide`，耗时45分钟以上之后，终于提示build success。有些task提示使用过时的API

```
注: 某些输入文件使用或覆盖了已过时的 API。
注: 有关详细信息, 请使用 -Xlint:deprecation 重新编译。
注: 某些输入文件使用了未经检查或不安全的操作。
...
BUILD SUCCESSFUL in 45m 26s
87 actionable tasks: 87 executed
```

执行编译`./gradlew build`，编译成功后，压缩包在`sonar-application/build/distributions/`目录下

执行build则会先使用yarn编译站点`Task :server:sonar-web:yarn`，下载依赖时有时网络问题，代理稳定的情况下，多执行几次即可，本地执行时，缓存效果生效，相当于“断点续编译”。

windows环境下测试模块失败，重新编译，忽略测试`gradlew build -x test`，成功。

```
...
> Task :server:sonar-db-dao:dumpSchema
...
15:57:44.599 INFO  org.sonar.db.SQDatabase - #4201 'Move default quality gate to global properties': success | time=1ms
...
..
.
BUILD SUCCESSFUL in 26m 42s
209 actionable tasks: 15 executed, 194 up-to-date
```

window环境下启动时，提示链接失败，很久之前的hosts配置影响，关闭后OK。

新版本(8.7-snapshot版本)启动后强制要求修改默认的密码。

```
jvm 1    | 2021.01.12 16:20:49 WARN  app[][startup] ####################################################################################################################
jvm 1    | 2021.01.12 16:20:49 WARN  app[][startup] Default Administrator credentials are still being used. Make sure to change the password or deactivate the account.
jvm 1    | 2021.01.12 16:20:49 WARN  app[][startup] ####################################################################################################################
```

sonarqube包含的模块——

```
./sonar-shutdowner/build.gradle
./sonar-core/build.gradle
./server/sonar-process/build.gradle
./server/sonar-webserver-core/build.gradle
./server/sonar-webserver/build.gradle
./server/sonar-docs/build.gradle
./server/sonar-ce-task/build.gradle
./server/sonar-ce-common/build.gradle
./server/sonar-db-migration/build.gradle
./server/sonar-db-core/build.gradle
./server/sonar-auth-ldap/build.gradle
./server/sonar-webserver-es/build.gradle
./server/sonar-main/build.gradle
./server/sonar-webserver-ws/build.gradle
./server/sonar-ce/build.gradle
./server/sonar-auth-github/build.gradle
./server/sonar-server-common/build.gradle
./server/build.gradle
./server/sonar-ce-task-projectanalysis/build.gradle
./server/sonar-auth-saml/build.gradle
./server/sonar-webserver-webapi/build.gradle
./server/sonar-auth-common/build.gradle
./server/sonar-auth-gitlab/build.gradle
./server/sonar-webserver-api/build.gradle
./server/sonar-webserver-auth/build.gradle
./server/sonar-web/build.gradle
./server/sonar-db-dao/build.gradle
./sonar-scanner-engine/build.gradle
./sonar-scanner-engine-shaded/build.gradle
./sonar-plugin-api/build.gradle
./sonar-ws-generator/build.gradle
./sonar-plugin-api-impl/build.gradle
./sonar-application/build.gradle
./sonar-application/bundled_plugins.gradle
./sonar-ws/build.gradle
./sonar-testing-ldap/build.gradle
./sonar-markdown/build.gradle
./build.gradle
./.gradle
./sonar-testing-harness/build.gradle
./plugins/build.gradle
./plugins/sonar-xoo-plugin/build.gradle
./sonar-check-api/build.gradle
./sonar-scanner-protocol/build.gradle
./sonar-duplications/build.gradle
./settings.gradle
```

## SonarQube alike

[SonarQube History](https://www.sonarsource.com/company/history/) 

Freddy Mallet  
Simon Brandhof  
Olivier Gaudin  


最早在2007年，使用Sonar这个名称，后来由于商标问题，[更名](http://sonar.15.x6.nabble.com/SONAR-is-becoming-SONARQUBE-td5010134.html) 为SonarQube。到2008年，三人创建SonarSource公司进入ToB领域。

目前SQ更像是一个平台，通过支持自己开发的分析工具，以及支持的第三方工具结果。例如——

- Conventions (Checkstyle)
- Bad practices (PMD)
- Potential bugs (FindBugs)

SQ可以从多个维度衡量代码质量——

- bugs
- code smells
- security vulnerabilities 
- duplicated code
- coding standards 
- unit tests
- code coverage
- code complexity 



[Is SonarQube Replacement for Checkstyle, PMD, FindBugs?](https://stackoverflow.com/questions/5479019/is-sonarqube-replacement-for-checkstyle-pmd-findbugs)  

>[Yes and No.](https://stackoverflow.com/a/24597073/1087122) In addition to the other answers.
>
>SonarQube is currently on the way to deprecate PMD, Checkstyle and Findbugs and use their own technology to analyze Java code (called [SonarJava](https://redirect.sonarsource.com/plugins/java.html)). They do it, because they don't want to spend their time fixing, upgrading (or waiting on it) those libraries (e.g. for Java 8), which for example uses outdated libraries.
>
>They also got a new set of plugins for your personal IDE called [SonarLint](http://www.sonarlint.org/index.html).


- [SonarQube vs FindBugs, CheckStyle, PMD](https://groups.google.com/g/sonarqube/c/9M0iZ4OILVM)   
- [Sonarqube, PMD, findbugs and checkstyle](https://community.sonarsource.com/t/sonarqube-pmd-findbugs-and-checkstyle/2919)


从上面的讨论组以及目前SQ支持的插件来看，上述三个工具依然以插件形式支持。但对PMD和FindBugs的替换是成功的。

| Tool| Source | Sonar-Plugin|
| --- | --- | --- |
| [SpotBugs](https://spotbugs.github.io/)| [github](https://github.com/spotbugs/spotbugs) | [sonar-findbugs](https://github.com/spotbugs/sonar-findbugs) |
| [PMD](https://pmd.github.io/)| [github](https://github.com/pmd/pmd) | [sonar-pmd](https://github.com/jensgerdes/sonar-pmd)  |
| [Checkstyle](https://checkstyle.org/)|[github](https://github.com/checkstyle/checkstyle) | [sonar-checkstyle](https://github.com/checkstyle/sonar-checkstyle) |


[Importing Third-Party Issues](https://docs.sonarqube.org/latest/analysis/external-issues/)

| Language | Property | Remarks |
| --- | --- | --- |
| Java | `sonar.java.spotbugs.reportPaths` | Comma-delimited list of paths to reports from [SpotBugs](https://spotbugs.github.io/), FindSecBugs, or FindBugs |
| Java | `sonar.java.pmd.reportPaths` | Comma-delimited list of paths to reports from [PMD](http://maven.apache.org/plugins/maven-pmd-plugin/usage.html) |
| Java | `sonar.java.checkstyle.reportPaths` | Comma-delimited list of paths to reports from [Checkstyle](http://maven.apache.org/plugins/maven-checkstyle-plugin/checkstyle-mojo) |

[Tools to Improve Java Code Quality](https://www.romexsoft.com/blog/improve-java-code-quality/)   
>Sonar uses FindBugs, Checkstyle and PMD to collect and analyze source code for bugs, bad code, and possible violation of code style policies. 

[Alibaba P3C](https://github.com/alibaba/p3c) ——包含以下三个部分

*   [PMD implementations](https://github.com/alibaba/p3c/blob/master/p3c-pmd) 基于PMD规则，实现了54条Alibaba开发规约中的规则，[详见](https://github.com/alibaba/p3c/tree/master/p3c-pmd)

*   [IntelliJ IDEA plugin](https://github.com/alibaba/p3c/blob/master/idea-plugin)， 使用方法[说明](https://github.com/alibaba/p3c/blob/master/idea-plugin/README_cn.md)
>实现了开发手册中的的53条规则，大部分基于PMD实现，其中有4条规则基于IDEA实现，并且基于IDEA [Inspection](https://www.jetbrains.com/help/idea/code-inspection.html)实现了实时检测功能。


*   [Eclipse plugin](https://github.com/alibaba/p3c/blob/master/eclipse-plugin)，使用方法[说明](https://github.com/alibaba/p3c/blob/master/eclipse-plugin/README_cn.md)
>插件实现了开发手册中的53条规则，大部分基于PMD实现，其中有4条规则基于Eclipse实现，支持4条规则的QuickFix功能。

### 能否在8.4版本的SonarQube上使用P3C定义的规则？

**No and Yes**不能直接使用，需要做定制开发：将P3C-pmd集成或新创建一个类似`sonar-pmd`的插件使用。

### pmd、pmd的sonar插件问题

SQ对于PMD规则的采用插件的方式。根据[兼容矩阵Plugin Version Matrix](https://update.sonarsource.org/plugins/compatibility-matrix.html)，目前支持的PMD的插件版本为3.2.1

从[sonar-pmd](https://github.com/jensgerdes/sonar-pmd)插件的[对应关系](https://github.com/jensgerdes/sonar-pmd#description--features)可以看到，3.2.1版本的插件对应的pmd版本为6.10.0 (目前PMD的最新版本为[6.29](https://github.com/pmd/pmd/releases/tag/pmd_releases%2F6.29.0))

|name|version|version|version|version|version|
|---------|---------|---------|---------|---------|---------|
|PMD Plugin|2.5|2.6|3.0.0|3.1.x|3.2.x|
|PMD|5.4.0|5.4.2|5.4.2|6.9.0|6.10.0|
|Max. supported Java Version|1.7|1.8|1.8|11||
|Min. SonarQube Version|4.5.4|4.5.4|6.6|6.6||

###  p3c-pmd、p3c-pmd的插件问题

[p3c-pmd](https://github.com/alibaba/p3c/tree/master/p3c-pmd)是基于pmd 6.15版本 `net.sourceforge.pmd:pmd-java:jar:6.15.0:compile`，这意味着SQ支持的pmd插件版本可以集成这个规则。

目前没有现成的插件，目前开源的插件——

- [sonarqube中添加p3c-pmd整合阿里java开发规范](https://www.jianshu.com/p/a3a58ac368be)使用的是1.3.6版本
- [sonar-p3c-pmd](https://github.com/mrprince/sonar-p3c-pmd)基于5.4版本，对应的插件版本为`2.6`，应用方式[示例](http://duwanjiang.com/%E4%BB%A3%E7%A0%81%E8%B4%A8%E9%87%8F/2019/04/03/%E5%A6%82%E4%BD%95%E5%9C%A8sonarQube%E7%9A%84pmd%E6%8F%92%E4%BB%B6%E4%B8%AD%E6%95%B4%E5%90%88%E9%98%BF%E9%87%8C%E5%BC%80%E5%8F%91%E8%A7%84%E8%8C%83/) 
- [**sonar-p3c-pmd fork **](https://github.com/rhinoceros/sonar-p3c-pmd)，应用方式[示例](https://my.oschina.net/u/4327212/blog/3209285)
- [**sonar-p3c-pmd fork**](https://github.com/sqgzy/sonar-p3c-pmd)官方的sonar-pmd插件。 可以借鉴这个版本查看如何进行集成


结论：改造SonarQube支持的sonar-pmd插件，使用p3c-pmd的插件规则，生成插件，提供给SonarQube使用。

基于官方插件项目[sonar-pmd](https://github.com/jensgerdes/sonar-pmd)进行改造，[这篇文章](http://duwanjiang.com/%E4%BB%A3%E7%A0%81%E8%B4%A8%E9%87%8F/2019/04/03/%E5%A6%82%E4%BD%95%E5%9C%A8sonarQube%E7%9A%84pmd%E6%8F%92%E4%BB%B6%E4%B8%AD%E6%95%B4%E5%90%88%E9%98%BF%E9%87%8C%E5%BC%80%E5%8F%91%E8%A7%84%E8%8C%83/#21sonar-p3c-pmd%E5%B7%A5%E7%A8%8B%E4%BB%8B%E7%BB%8D)有详细说明，[这个工程](https://github.com/sqgzy/sonar-p3c-pmd)有低版本的实现。

每条规则对应的3个配置文件：

- `src\main\resources\org\sonar\l10n\pmd.properties`
- `src\main\resources\org\sonar\plugins\pmd\rules.xml`
- `src\main\resources\com\sonar\sqale\pmd-model.xml`

注意：

1. properties文件[编码问题](https://stackoverflow.com/a/30815935/1087122) 

>The problem is that the Java properties files are/must/should been encoded in ´ISO-8859-1´ (Latin-1) by default. Thats an Java requirement.

可以利用JDK自动的工具`native2ascii`进行转换，例如： `native2ascii -encoding utf8 zh.properties  .normal.properties` 

不进行转换的情况下：第一，页面展示将是乱码；第二，有可能导致数据库报错`ERROR: value too long for type character varying(200)`，类似——

```
### Error updating database.  Cause: org.postgresql.util.PSQLException: ERROR: value too long for type character varying(200)
### The error may exist in org.sonar.db.rule.RuleMapper
### The error may involve org.sonar.db.rule.RuleMapper.updateDefinition-Inline
### The error occurred while setting parameters
### Cause: org.postgresql.util.PSQLException: ERROR: value too long for type character varying(200)
        at org.apache.ibatis.exceptions.ExceptionFactory.wrapException(ExceptionFactory.java:30)
        at org.apache.ibatis.session.defaults.DefaultSqlSession.update(DefaultSqlSession.java:199)
        at org.apache.ibatis.binding.MapperMethod.execute(MapperMethod.java:67)
        at org.apache.ibatis.binding.MapperProxy$PlainMethodInvoker.invoke(MapperProxy.java:144)
        at org.apache.ibatis.binding.MapperProxy.invoke(MapperProxy.java:85)
        at com.sun.proxy.$Proxy53.updateDefinition(Unknown Source)
        at org.sonar.db.rule.RuleDao.update(RuleDao.java:181)
        at org.sonar.server.rule.RegisterRules.update(RegisterRules.java:782)
        at org.sonar.server.rule.RegisterRules.registerRule(RegisterRules.java:384)
        at org.sonar.server.rule.RegisterRules.start(RegisterRules.java:133)
        at org.sonar.core.platform.StartableCloseableSafeLifecyleStrategy.start(StartableCloseableSafeLifecyleStrategy.java:40)
        at org.picocontainer.injectors.AbstractInjectionFactory$LifecycleAdapter.start(AbstractInjectionFactory.java:84)
        at org.picocontainer.behaviors.AbstractBehavior.start(AbstractBehavior.java:169)
        at org.picocontainer.behaviors.Stored$RealComponentLifecycle.start(Stored.java:132)
        at org.picocontainer.behaviors.Stored.start(Stored.java:110)
        at org.picocontainer.DefaultPicoContainer.potentiallyStartAdapter(DefaultPicoContainer.java:1016)
        at org.picocontainer.DefaultPicoContainer.startAdapters(DefaultPicoContainer.java:1009)
        at org.picocontainer.DefaultPicoContainer.start(DefaultPicoContainer.java:767)
        at org.sonar.core.platform.ComponentContainer.startComponents(ComponentContainer.java:136)
        at org.sonar.server.platform.platformlevel.PlatformLevel.start(PlatformLevel.java:90)
        at org.sonar.server.platform.platformlevel.PlatformLevelStartup.access$001(PlatformLevelStartup.java:48)
        at org.sonar.server.platform.platformlevel.PlatformLevelStartup$1.doPrivileged(PlatformLevelStartup.java:85)
        at org.sonar.server.user.DoPrivileged.execute(DoPrivileged.java:46)
        at org.sonar.server.platform.platformlevel.PlatformLevelStartup.start(PlatformLevelStartup.java:82)
        at org.sonar.server.platform.PlatformImpl.executeStartupTasks(PlatformImpl.java:198)
        at org.sonar.server.platform.PlatformImpl.access$400(PlatformImpl.java:46)
        at org.sonar.server.platform.PlatformImpl$1.lambda$doRun$1(PlatformImpl.java:122)
        at org.sonar.server.platform.PlatformImpl$AutoStarterRunnable.runIfNotAborted(PlatformImpl.java:370)
        at org.sonar.server.platform.PlatformImpl$1.doRun(PlatformImpl.java:122)
        at org.sonar.server.platform.PlatformImpl$AutoStarterRunnable.run(PlatformImpl.java:354)
        at java.base/java.lang.Thread.run(Thread.java:834)
Caused by: org.postgresql.util.PSQLException: ERROR: value too long for type character varying(200)
        at org.postgresql.core.v3.QueryExecutorImpl.receiveErrorResponse(QueryExecutorImpl.java:2532)
        at org.postgresql.core.v3.QueryExecutorImpl.processResults(QueryExecutorImpl.java:2267)
        at org.postgresql.core.v3.QueryExecutorImpl.execute(QueryExecutorImpl.java:312)
        at org.postgresql.jdbc.PgStatement.executeInternal(PgStatement.java:448)
        at org.postgresql.jdbc.PgStatement.execute(PgStatement.java:369)
        at org.postgresql.jdbc.PgPreparedStatement.executeWithFlags(PgPreparedStatement.java:153)
        at org.postgresql.jdbc.PgPreparedStatement.execute(PgPreparedStatement.java:142)
        at org.apache.commons.dbcp2.DelegatingPreparedStatement.execute(DelegatingPreparedStatement.java:94)
        at org.apache.commons.dbcp2.DelegatingPreparedStatement.execute(DelegatingPreparedStatement.java:94)
        at org.apache.ibatis.executor.statement.PreparedStatementHandler.update(PreparedStatementHandler.java:47)
        at org.apache.ibatis.executor.statement.RoutingStatementHandler.update(RoutingStatementHandler.java:74)
        at org.apache.ibatis.executor.ReuseExecutor.doUpdate(ReuseExecutor.java:52)
        at org.apache.ibatis.executor.BaseExecutor.update(BaseExecutor.java:117)
        at org.apache.ibatis.executor.CachingExecutor.update(CachingExecutor.java:76)
        at org.apache.ibatis.session.defaults.DefaultSqlSession.update(DefaultSqlSession.java:197)
        ... 29 common frames omitted
```

2. 默认sonar-pmd插件的分类是**异味**

在rules.xml文件中对应的rule属性里添加`<tag>bug<tag>`， [官方说明](https://stackoverflow.com/a/44281775/1087122)

最终效果——

![](https://upload-images.jianshu.io/upload_images/3296949-92dbeb3da90bf452.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### sonar-pmd插件使用问题

在SonarQube中的Quality Profile中复制一份内置的`Sonar way`；增加新集成的插件p3c-pmd中的rules；然后将这个新的profile设置为默认。——完成激活并使用插件的前提

使用SonarScanner执行代码扫描——实测不需要另外添加任何依赖，会启用新集成的插件中的规则。使用中遇到以下三个问题。

第一，报错空指针问题。——从下面的调用栈可以看出插件起效的逻辑关系

**结论：** 这是因为集成时使用了最新的p3c-pmd代码，版本`2.1.1`；但pom依赖还是使用的低版本`1.3.6`。

```
ERROR: Error during SonarQube Scanner execution
java.lang.NullPointerException
        at java.base/java.util.concurrent.ConcurrentHashMap.get(Unknown Source)
        at java.base/java.util.Properties.getProperty(Unknown Source)
        at com.alibaba.p3c.pmd.I18nResources$XmlResourceBundle.handleGetObject(I18nResources.java:90)
        at java.base/java.util.ResourceBundle.getObject(Unknown Source)
        at java.base/java.util.ResourceBundle.getString(Unknown Source)
        at com.alibaba.p3c.pmd.I18nResources.getMessageWithExceptionHandled(I18nResources.java:74)
        at com.alibaba.p3c.pmd.lang.AbstractXpathRule.setDescription(AbstractXpathRule.java:30)
        at net.sourceforge.pmd.rules.RuleBuilder.build(RuleBuilder.java:194)
        at net.sourceforge.pmd.rules.RuleFactory.buildRule(RuleFactory.java:189)
        at net.sourceforge.pmd.RuleSetFactory.parseSingleRuleNode(RuleSetFactory.java:551)
        at net.sourceforge.pmd.RuleSetFactory.parseRuleNode(RuleSetFactory.java:450)
        at net.sourceforge.pmd.RuleSetFactory.parseRuleSetNode(RuleSetFactory.java:367)
        at net.sourceforge.pmd.RuleSetFactory.createRuleSet(RuleSetFactory.java:214)
        at net.sourceforge.pmd.RuleSetFactory.createRule(RuleSetFactory.java:313)
        at net.sourceforge.pmd.RuleSetFactory.parseRuleReferenceNode(RuleSetFactory.java:598)
        at net.sourceforge.pmd.RuleSetFactory.parseRuleNode(RuleSetFactory.java:452)
        at net.sourceforge.pmd.RuleSetFactory.parseRuleSetNode(RuleSetFactory.java:367)
        at net.sourceforge.pmd.RuleSetFactory.createRuleSet(RuleSetFactory.java:214)
        at net.sourceforge.pmd.RuleSetFactory.createRuleSet(RuleSetFactory.java:209)
        at net.sourceforge.pmd.RuleSetFactory.createRuleSet(RuleSetFactory.java:195)
        at org.sonar.plugins.pmd.PmdExecutor.createRuleSets(PmdExecutor.java:143)
        at org.sonar.plugins.pmd.PmdExecutor.executeRules(PmdExecutor.java:122)
        at org.sonar.plugins.pmd.PmdExecutor.executePmd(PmdExecutor.java:98)
        at org.sonar.plugins.pmd.PmdExecutor.execute(PmdExecutor.java:80)
        at org.sonar.plugins.pmd.PmdSensor.execute(PmdSensor.java:71)
        at org.sonar.scanner.sensor.AbstractSensorWrapper.analyse(AbstractSensorWrapper.java:48)
        at org.sonar.scanner.sensor.ModuleSensorsExecutor.execute(ModuleSensorsExecutor.java:85)
        at org.sonar.scanner.sensor.ModuleSensorsExecutor.lambda$execute$1(ModuleSensorsExecutor.java:59)
        at org.sonar.scanner.sensor.ModuleSensorsExecutor.withModuleStrategy(ModuleSensorsExecutor.java:77)
        at org.sonar.scanner.sensor.ModuleSensorsExecutor.execute(ModuleSensorsExecutor.java:59)
        at org.sonar.scanner.scan.ModuleScanContainer.doAfterStart(ModuleScanContainer.java:82)
        at org.sonar.core.platform.ComponentContainer.startComponents(ComponentContainer.java:137)
        at org.sonar.core.platform.ComponentContainer.execute(ComponentContainer.java:123)
        at org.sonar.scanner.scan.ProjectScanContainer.scan(ProjectScanContainer.java:388)
        at org.sonar.scanner.scan.ProjectScanContainer.scanRecursively(ProjectScanContainer.java:384)
        at org.sonar.scanner.scan.ProjectScanContainer.doAfterStart(ProjectScanContainer.java:353)
        at org.sonar.core.platform.ComponentContainer.startComponents(ComponentContainer.java:137)
        at org.sonar.core.platform.ComponentContainer.execute(ComponentContainer.java:123)
        at org.sonar.scanner.bootstrap.GlobalContainer.doAfterStart(GlobalContainer.java:144)
        at org.sonar.core.platform.ComponentContainer.startComponents(ComponentContainer.java:137)
        at org.sonar.core.platform.ComponentContainer.execute(ComponentContainer.java:123)
        at org.sonar.batch.bootstrapper.Batch.doExecute(Batch.java:72)
        at org.sonar.batch.bootstrapper.Batch.execute(Batch.java:66)
        at org.sonarsource.scanner.api.internal.batch.BatchIsolatedLauncher.execute(BatchIsolatedLauncher.java:46)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
        at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
        at java.base/java.lang.reflect.Method.invoke(Unknown Source)
        at org.sonarsource.scanner.api.internal.IsolatedLauncherProxy.invoke(IsolatedLauncherProxy.java:60)
        at com.sun.proxy.$Proxy0.execute(Unknown Source)
        at org.sonarsource.scanner.api.EmbeddedScanner.doExecute(EmbeddedScanner.java:189)
        at org.sonarsource.scanner.api.EmbeddedScanner.execute(EmbeddedScanner.java:138)
        at org.sonarsource.scanner.cli.Main.execute(Main.java:112)
        at org.sonarsource.scanner.cli.Main.execute(Main.java:75)
        at org.sonarsource.scanner.cli.Main.main(Main.java:61)
```

第二，提示找不到类`java.lang.NoClassDefFoundError: com/google/gson/Gson` 

**结论：** p3c-pmd项目的代码使用到gson包，需要在插件的依赖中添加。

第三， 被扫描的源码中的lambda表达式不支持导致对应源码被忽略。`Caused by: net.sourceforge.pmd.lang.java.ast.ParseException: Line 48, Column 74: Cannot use lambda expressions when running in JDK inferior to 1.8 mode! `

**结论：** PMD使用的默认JDK版本为1.6，[官方说明](https://github.com/jensgerdes/sonar-pmd#troubleshooting) `PMD uses the default Java version - which is 1.6` ，[论坛讨论](https://community.sonarsource.com/t/cannot-use-lambda-expressions-when-running-in-jdk-inferior-to-1-8-mode/14237) 。扫描时指定参数`-Dsonar.java.source=1.8`即可

## SonarQube Source

使用SQ代替SonarQube

### 指定profile

[sonar.profile](https://docs.sonarqube.org/7.6/analysis/analysis-parameters/)参数已经被废弃，[社区里的官方回应](https://community.sonarsource.com/t/quality-profiles-are-getting-changed-automatically-with-each-run-of-sonar-analysis/14650/3)——

>In fact, the `sonar.profile` has been removed in recent versions partly because of exactly what you’re experiencing. When you specify sonar.profile in your analysis, you override the profile that would normally be used. It is expected that you still see the other profile in the UI. Analysis parameters provide a temporary override; they do not re-assign.

目前的版本8.4只能从UI上进行创建和分配

### 编译sonar-java 

计划将SonarQube自带的一些code_smell级别的规则升级为bug级别。前端没法直接操作。社区的官方[答复](https://community.sonarsource.com/t/how-to-change-the-type-of-a-rule-e-g-from-bug-to-code-smell/1472/4?u=gebitang)——

**the type of a rule cannot be modified in SonarQube**

>there’s no easy way to change the rule type, and that’s on purpose. You could fork the plugin of interest, edit the code and re-package, but then you have to do that for each new plugin version, rather than being able to just benefit from the work of others. In all, this is really not a recommended solution.
>
>Far better, to tell your architect that rule types are immutable - because within the UI they are.

所以只能重新编译官方的[soanr-java](https://github.com/SonarSource/sonar-java)项目。

官方不推荐直接编辑 java-checks模块下`src\main\resources\org\sonar\l10n\java\rules\java`目录的内容，因为所有文件是根据[RSPEC](https://jira.sonarsource.com/browse/RSPEC)自动生成的

>The files from this folder are **generated** directly from our rules descriptions repository: [RSPEC](https://jira.sonarsource.com/browse/RSPEC).

修改对应rule的json描述文件即可，例如将type从`CODE_SMELL`更改为`BUG`。

执行`mvn package -Dmaven.test.skip=ture`即可重新打包。

- 项目打包需要Java11版本，否则报错类版本不兼容，类似` bad class file: xxx class file has wrong version 55.0, should be 52.0` 
我遇到的是针对`org.apache.wicket:wicket-core:jar:9.1.0`这个包中的类

安装jdk11之后，可以手动修改mvn对应的启动脚本中的JAVA_HOME对应的目录。 [set specific java version to Maven](https://stackoverflow.com/a/19654699/1087122), You could also go into your mvn(non-windows)/mvn.bat/mvn.cmd(windows) and set your java version explicitly there.

- 指定mvn使用代理 [set proxy for maven](https://medium.com/@petehouston/execute-maven-behind-a-corporate-proxy-network-5e08d075f744)

```
mvn [COMMAND] -Dhttp.proxyHost=[PROXY_SERVER] -Dhttp.proxyPort=[PROXY_PORT] -Dhttp.nonProxyHosts=[PROXY_BYPASS_IP] 
# example 
mvn install -Dhttp.proxyHost=10.10.0.100 -Dhttp.proxyPort=8080 -Dhttp.nonProxyHosts=localhost|127.0.0.1
mvn clean package -Dmaven.test.skip=true -Dhttp.proxyHost=127.0.0.1 -Dhttp.proxyPort=7890 -Dhttps.proxyHost=127.0.0.1 -Dhttps.proxyPort=7890 
```


需要下载指定的plugin

```
<plugin>
            <groupId>com.googlecode.maven-download-plugin</groupId>
            <artifactId>download-maven-plugin</artifactId>
            <executions>
              <execution>
                <id>unpack-jre-windows</id>
                <phase>generate-test-resources</phase>
                <goals>
                  <goal>wget</goal>
                </goals>
                <configuration>
                  <url>https://github.com/AdoptOpenJDK/openjdk8-binaries/releases/download/jdk8u265-b01/OpenJDK8U-jre_x86-32_windows_hotspot_8u265b01.zip</url>
                  <unpack>true</unpack>
                  <outputDirectory>${project.build.directory}/jre</outputDirectory>
                  <sha256>1d1624afe2aaa811e2bf09fb1c432f3f0ad4b8a7b6243db2245390f1b59a17cf</sha256>
                </configuration>
              </execution>
            </executions>
          </plugin>
```

编译打包通过

![](https://upload-images.jianshu.io/upload_images/3296949-02c92af04532ed36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 源码编译sonar-java问题

window环境下编译成功，但生成的sh脚本默认的回车格式为`Windows(CR LF)`，在linux环境下可以使用`cat -v sonar.sh`进行查看验证。

- window下可以利用`dos2unix`应用进行转换，[下载站点](http://dos2unix.sourceforge.net/)
- linux下使用脚本`sed -i 's/\r$//g' input`进行替换，参考[sed Delete / Remove ^M Carriage Return (Line Feed / CRLF) on Linux or Unix](https://www.cyberciti.biz/faq/sed-remove-m-and-line-feeds-under-unix-linux-bsd-appleosx/)

涉及到的脚本包括：
- `/bin/linux-x86-64`目录下的启动脚本`sonar.sh`
- `/elasticsearch/bin`目录下的两个脚本`elasticsearch-env`和`elasticsearch-env`

使用脚本启动时提示报错，修改后启动log提示`permission denied`，添加权限后提示`no such file`，对比了下载的官方版本，这两个文件`elasticsearch-env`和`elasticsearch-env`都是shell脚本。windows下编译时格式问题(每行结尾包含`^M `字符样式，实际是windows下的换行符在unix环境下的展示问题)


### 版本升级过程

官方指导[upgrade guide](https://redirect.sonarsource.com/doc/upgrading.html).

- 下载、解压
- 根据[兼容性列表](https://docs.sonarqube.org/display/PLUG/Plugin+Version+Matrix)复制对应的插件到插件目录`extensions/plugins` [兼容性列表详情](https://docs.sonarqube.org/latest/instance-administration/plugin-version-matrix/)
- 更新对应的配置文件`sonar.properties` 和 `wrapper.conf`
- 停止旧版本、启动新版本
- 访问`http://yourSonarQubeServerURL/setup`目录根据提示进行升级操作

使用SQ8.5版本时，启动后首页提示：

>### SonarQube is under maintenance
>
>While waiting, you might want to investigate [new plugins](https://redirect.sonarsource.com/doc/plugin-library.html) to extend the current functionality.
>
>If you are an administrator and have no idea why this message is being shown, you should read the [upgrade guide](https://redirect.sonarsource.com/doc/upgrading.html).


后台log提示需要升级数据库：

>2020.10.21 14:10:07 WARN  app[][startup] ################################################################################
>2020.10.21 14:10:07 WARN  app[][startup] The database must be manually upgraded. Please backup the database and browse /setup. For more information: https://docs.sonarqube.org/latest/setup/upgrading
>2020.10.21 14:10:07 WARN  app[][startup] ################################################################################


### webhook: Server Unreachable

现象：使用内网域名顶一顶webhook url，一直工作正常，但最近提示Server Unreachable。使用内网域名对应的服务ip:port的方式可以正常调用。

临时方案：尝试打开`sonar.log.level.app=DEBUG`然后重启SQ服务。域名对应的回调地址恢复正常。[How to check the details of webhook call?](https://community.sonarsource.com/t/how-to-check-the-details-of-webhook-call/)

webhook对应的log为Compute Engine，应该打开`sonar.log.level.ce=DEBUG`，对应的log类似——

>2020.10.21 10:30:15 DEBUG ce[AXVI_gynQqxgVa12m-LG][o.s.s.w.WebHooksImpl] Sent webhook 'wh-name' | url=http://su.mycallback.com/api/callback/url | time=655ms | status=200

这样可以跟踪到源码位置`WebHooksImpl`，属于`server:sonar-server-common`模块下的`org.sonar.server.webhook`包。回调请求使用的是okhttp的类库，还不清楚为什么会出现服务不可用的问题。可以以此为入口跟踪一下源码逻辑。

目前源码可以成功打包，修改对应代码增加log信息（慢慢使用自己编译的版本替换现有的发布版本？）

## SonarQube Usage

环境准备以及使用中的问题

### SonarQube in Docker 

[Installing SonarQube from the Docker Image](https://docs.sonarqube.org/latest/setup/install-server/#header-4)

[sonarqube on hub docker](https://hub.docker.com/_/sonarqube/)， [dockerfile for sonarqube](https://github.com/SonarSource/docker-sonarqube) 

[about docker volume](https://docs.docker.com/storage/volumes/)

- 拉取镜像 `docker pull sonarqube:8.2-community`
- 启动前创建volumes `docker volume create --name sonarqube_data`，包括`sonarqube_data`，`sonarqube_logs`，`sonarqube_extensions`
- 启动容器

```
# 使用docker run --help查看
docker run --stop-timeout 3600 -d --name sonarqube \
-p 9000:9000 \
-e SONAR_JDBC_URL=jdbc:postgresql://host:port/dbname \
-e SONAR_JDBC_USERNAME=sonar \
-e SONAR_JDBC_PASSWORD=sonarpassword \
-v sonarqube_data:/opt/sonarqube/data \
-v sonarqube_extensions:/opt/sonarqube/extensions \
-v sonarqube_logs:/opt/sonarqube/logs \
sonarqube:8.4.2-community
```

执行`docker exec -it sonarqube /bin/bash`登录到启动的容器中查看SQ状态

**注意事项：**  

- 环境要求 

因为SQ包含嵌入的ES，所以docker宿主机需要满足[Elasticsearch production mode requirements](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-cli-run-prod-mode)和[File Descriptors configuration](https://www.elastic.co/guide/en/elasticsearch/reference/current/file-descriptors.html)的要求

```
sysctl -w vm.max_map_count=262144
sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 4096

```

- 关闭参数

为保证SQ正常关闭，将关闭容器的默认时间10秒变为3600秒 `docker run --stop-timeout 3600 `

- Mac上查看volume的内容

[Host path of volume Docker Desktop for Mac](https://forums.docker.com/t/host-path-of-volume/12277)， [Screen Commands for Docker for Mac](https://gist.github.com/joacim-boive/569b66c3be673a3f3e802939fe8edd56)

`docker inspect sonarqube_logs`查看对应volume的挂载点  

执行`screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty` 登录到docker虚拟机  

到对应的目录，例如`cd /var/lib/docker/volumes/sonarqube_logs/_data`下可以看到对应的log。执行 `Ctrl + a k`退出虚拟机

### SonarQube Build Breaker

有需求希望在Sonar的静态代码扫描之后，直接判断是否终止打包流程。

按照现在的架构——

>好像有点麻烦。 因为 sonar分析完毕后将结果传递给sonarqube，SonarQube有个分析器分析结果（CE版本一次只能处理一个）。
>得到结果是异步的。

还是需要调研一下：

在5.2版本之前的SonarQube上是有这个功能的，作为Jenkins的插件使用，但后来SonarQube自身架构调整，分析器本身是不知道最终结果的，Server端聚合数据。

实际上，官方已经建议[不应该打破打包流程](https://blog.sonarsource.com/why-you-shouldnt-use-build-breaker/)，后来的实现(6.2版本开始)是[提供jenkins的webhook](https://blog.sonarsource.com/breaking-the-sonarqube-analysis-with-jenkins-pipelines/)。目前最新版本也是提供webhook方式。


但直接在pipeline中实现break的需求依然存在，于是[Breaking your build on SonarQube quality gate failure](http://qaware.blogspot.com/2020/02/breaking-your-build-on-sonarqube.html)出现。github对应代码[sonarqube-build-breaker](https://github.com/qaware/sonarqube-build-breaker)

插件实现逻辑——轮询SonarQube，查找结果，根据返回的结果判断是否应该终止pipeline（报错）。同时依赖SonarQube上的Quality Gate的设置。

```java
public void breakBuildIfNeeded(ProjectKey projectKey, BranchMode branchMode) throws IOException, SonarQubeException, InterruptedException, BreakBuildException {
        LOGGER.info("Fetching analysis tasks ...");
        AnalysisTasks analysisTasks = sonarQubeConnector.fetchAnalysisTasks(projectKey, branchMode);
        while (!analysisTasks.getQueue().isEmpty()) {
            LOGGER.info("Analysis task still running, checking again in {} second(s) ...", waitTime.toMillis() / 1000);
            Thread.sleep(waitTime.toMillis());
            analysisTasks = sonarQubeConnector.fetchAnalysisTasks(projectKey, branchMode);
        }

        if (analysisTasks.getLastFinished() == null) {
            LOGGER.error("Analysis queue is empty and there is no finished task. Make sure that you run SonarQube analysis before the build breaker!");
            throw new BreakBuildException("Analysis queue is empty and there is no finished task. Make sure that you run SonarQube analysis before the build breaker!");
        }

        if (analysisTasks.getLastFinished().getStatus() != AnalysisTasks.Status.SUCCESS) {
            LOGGER.error("Last analysis task failed, breaking build!");
            throw new BreakBuildException("Last analysis task failed, breaking build!");
        }

        LOGGER.info("Last analysis was successful, fetching quality gate status ...");
        QualityGateStatus qualityGateStatus = sonarQubeConnector.fetchQualityGateStatus(projectKey, branchMode);

        if (qualityGateStatus == QualityGateStatus.ERROR) {
            LOGGER.error("Quality gate failed, breaking build!");
            throw new BreakBuildException("Quality gate failed, breaking build!");
        }

        LOGGER.info("Great success, everything looks alright!");
    }
```

这看起来我们目前的做法更优雅乜。使用webhook，项目的质量阀采用自定义规则。


### SonarQube的测试覆盖和测试执行

[sonarqube coverage/](https://docs.sonarqube.org/latest/analysis/coverage/)

8.4版本包括两个概念：

- Test Coverage 测试覆盖 ——测试结果有多少覆盖率
- Test Execution 测试执行 ——具体包含多少个测试用例

低版本上，如7.6似乎没有“测试执行”的说法，从7.7版本有对应的文档：[https://docs.sonarqube.org/`7.7`/analysis/coverage/](https://docs.sonarqube.org/7.7/analysis/coverage/)后续的7.8, 7.9直到最新的8.4版本。

`mvn test`默认绑定的goal是`surefire:test`，用来执行单元测试——run tests using a suitable unit testing framework. These tests should not require the code be packaged or deployed.

所以默认的 `mvn test`只会生成`Test Execution`的结果；

添加了jacoco插件并与test阶段绑定后——
`mvn test`会同时产生（surefire插件生成）单测结果，和（jacoco插件生成）单测覆盖率结果。

此时再执行`sonar:sonar`后，单测结果就展示为 “Test Execution 测试执行”；单测覆盖率展示为“Test Coverage 测试覆盖”



完成的生命周期可以[参考官方说明lifecycle reference](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#lifecycle-reference)

最常用的打包`jar`阶段对应的plugin和goal如下表——

|Phase|plugin:goal|
|---------|---------|
|process-resources|resources:resources|
|compile|compiler:compile|
|process-test-resources|resources:testResources|
|test-compile|compiler:testCompile|
|test|surefire:test|
|package|jar:jar|
|install|install:install|
|deploy|deploy:deploy|

对应的具体plugin信息为[具体plugin信息](https://maven.apache.org/ref/3.6.3/maven-core/default-bindings.html) ——
```
<phases>
  <process-resources>
    org.apache.maven.plugins:maven-resources-plugin:2.6:resources
  </process-resources>
  <compile>
    org.apache.maven.plugins:maven-compiler-plugin:3.1:compile
  </compile>
  <process-test-resources>
    org.apache.maven.plugins:maven-resources-plugin:2.6:testResources
  </process-test-resources>
  <test-compile>
    org.apache.maven.plugins:maven-compiler-plugin:3.1:testCompile
  </test-compile>
  <test>
    org.apache.maven.plugins:maven-surefire-plugin:2.12.4:test
  </test>
  <package>
    org.apache.maven.plugins:maven-jar-plugin:2.4:jar
  </package>
  <install>
    org.apache.maven.plugins:maven-install-plugin:2.4:install
  </install>
  <deploy>
    org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy
  </deploy>
</phases>
```

从上述信息也可以看出来plugin的命名规范和使用中的快捷方式。


### 使用sonar-maven-plugin插件执行sonar扫描

理论上，只需要配置`sonar-maven-plugin`插件就可以执行sonar扫描过程，需要有SonarQube服务可供访问。

需要如果不从命令行传递参数，可以在pom文件中指定对应的参数，在`properties`字段提供`sonar.host.url`和`sonar.login`字段的值，登录可使用token方式。示例如下——


```
<properties>
	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
	<java.version>1.8</java.version>

	<sonar.host.url>http://127.0.0.1:9000</sonar.host.url>
	<sonar.login>eef68547ea46d48aaefc472170ec87b833db182f</sonar.login>

</properties>

<plugin>
	<groupId>org.sonarsource.scanner.maven</groupId>
	<artifactId>sonar-maven-plugin</artifactId>
	<version>3.7.0.1746</version>
</plugin>
```

[SonarScanner for Maven](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-maven/)最新版本已经是8.4版本的SonarQube和3.7.0.1746版本的maven插件。

执行`mvn sonar:sonar`默认使用最新版本的sonar插件，可以使用`mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar`指定配置的插件版本

~~实测，即使没有配置jacoco插件，插件可以自动分析单测的测试结果(mvn test?)~~，可以看到覆盖率。目前我还清楚依赖的最小环境是怎样的？

不包含Jacoco插件的场景——

- 1. `mvn clean`清理环境
- 2.  执行`mvn test` 确保完成编译并执行了测试`maven-surefire-plugin:2.22.2:test` 

```
#target 目录下的文件目录
classes
generated-sources
generated-test-sources
maven-status
surefire-reports
test-classes
```

- 3. 执行 `sonar:sonar`结果将不包含覆盖率信息


包含Jacoco插件的场景(Jacoco插件于`test`阶段绑定)——

```xml
<plugin>
	<groupId>org.jacoco</groupId>
	<artifactId>jacoco-maven-plugin</artifactId>
	<version>0.8.5</version>
	<executions>
		<execution>
			<goals>
				<goal>prepare-agent</goal>
			</goals>
		</execution>
		<!-- attached to Maven test phase -->
		<execution>
			<id>report</id>
			<phase>test</phase>
			<goals>
				<goal>report</goal>
			</goals>
		</execution>
	</executions>
</plugin>
```

执行上述相同的三个步骤，则包含了覆盖率。

第二步结束时可以看到target文件夹下多出`site` 文件夹和 `jacoco.exec`文件。

```
#target 目录下的文件目录
classes
generated-sources
generated-test-sources
maven-status
site
surefire-reports
test-classes
jacoco.exec
```

**结论：**

SonarQube上展示的覆盖率依赖Jacoco插件的配合。


附录：

- 使用 `mvn help:effective-pom`可以看到当前项目完整的pom相关配置
- 使用`mvn dependency:tree`可以看到当前项目的依赖关系
- `pluginManagement`用来配置管理所有的plugin信息，用来提供给其他集成当前项目的项目使用，当前项目使用的plugin依然需要配置到`plugins`节点中。[官方解释Plugin_Management](http://maven.apache.org/pom.html#Plugin_Management)，对应的[提问](https://stackoverflow.com/questions/26736876/difference-between-plugins-and-pluginmanagement-tag-in-maven-pom-xml)
- `dependencyManagement`于此类似。[dependency-management](http://maven.apache.org/pom.html#dependency-management)




### 搭建8.3版本SonarQube服务 on CentOS7

[install-java-on-centos](https://phoenixnap.com/kb/install-java-on-centos)

系统环境：

`Linux version 3.10.0-693.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC) ) #1 SMP Tue Aug 22 21:09:27 UTC 2017`

#### 安装Open JDK 11

`sudo yum install -y java-11-openjdk-devel`

```
# install Java
>sudo yum install -y java-11-openjdk-devel
Installed:
  java-11-openjdk-devel.x86_64 1:11.0.7.10-4.el7_8

Dependency Installed:
  copy-jdk-configs.noarch 0:3.3-10.el7_5      giflib.x86_64 0:4.1.6-9.el7        java-11-openjdk.x86_64 1:11.0.7.10-4.el7_8   java-11-openjdk-headless.x86_64 1:11.0.7.10-4.el7_8
  javapackages-tools.noarch 0:3.4.1-11.el7    libXtst.x86_64 0:1.2.3-1.el7       lksctp-tools.x86_64 0:1.0.17-2.el7           pcsc-lite-libs.x86_64 0:1.8.8-8.el7
  python-javapackages.noarch 0:3.4.1-11.el7   python-lxml.x86_64 0:3.2.1-4.el7   ttmkfdir.x86_64 0:3.0.9-42.el7               tzdata-java.noarch 0:2020a-1.el7
  xorg-x11-fonts-Type1.noarch 0:7.5-9.el7

Complete!

```
不需要进行默认配置，机器已安装Java8环境。Sonar服务在 `conf/wrapper.conf`中指定Java11的地址即可 `wrapper.java.command=/usr/lib/jvm/java-11-openjdk/bin/java`


#### 遇到的问题

- ES端口被占有 -->  修改sonar.conf的`sonar.search.port`的值，更换一个端口
- bootstrap checks failed。

```
ERROR: [2] bootstrap checks failed
[1]: max file descriptors [65500] for elasticsearch process is too low, increase to at least [65535]
[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

```

第二个问题可以直接执行命令`sudo sysctl -w vm.max_map_count=262144`生效。

第一个问题`max file descriptor`的限制稍微麻烦点。 按照网上的介绍，修改了 `/etc/security/limits.conf` 增大用户限制，并不生效。

- [increasing-file-descriptors-and-open-files-limit-in-centos-7](https://www.99ideas.in/blog-post/increasing-file-descriptors-and-open-files-limit-in-centos-7/)
- 社区也遇到了同样的问题：[Error max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535] installing SonarQube 8.1 on CentOS](https://community.sonarsource.com/t/error-max-file-descriptors-4096-for-elasticsearch-process-is-too-low-increase-to-at-least-65535-installing-sonarqube-8-1-on-centos/20029)

使用`ulimit -a`查看时，`open files `的值依然是65000，执行 `sudo ulimit -n 65536`会提示`sudo ulimit command not found`

[解决这个问题](https://stackoverflow.com/a/17483998/1087122) 执行 `sudo sh -c "ulimit -n 65535 && exec su $LOGNAME"` 

`ulimit` is a shell builtin like `cd`, not a separate program. `sudo` looks for a **binary** to run, but there is no ulimit binary, which is why you get the error message. You need to run it in a shell.

However, while you do need to be root to raise the limit to 65535, you probably don’t want to run your program as root. So after you raise the limit you should switch back to the current user.

To do this, run:

`sudo sh -c "ulimit -n 65535 && exec su $LOGNAME"`

and you will get a **new shell**, without root privileges, but with the raised limit. The exec causes the new shell to replace the process with sudo privileges, so after you exit that shell, you won’t accidentally end up as root again.

如果上面的操作还是不生效， 实测下列操作可生效—— 这里有针对 [ubuntu的环境的做法](https://superuser.com/questions/1200539/cannot-increase-open-file-limit-past-4096-ubuntu)

[centos/redhat: change open files ulimit without reboot?](https://serverfault.com/a/389121)
```shell
$ ulimit -n  
4096
$ ulimit -n 8192
bash: ulimit: open files: cannot modify limit: Operation not permitted
$ sudo bash                    
# ulimit -n                    
4096
# ulimit -n 8192                  
# su - normaluser                 
$ ulimit -n                       
8192

```

上面的设置依然没有生效，需要组合使用——

- 编辑`sudo vi /etc/security/limits.conf`，将对应的设置修改为需要的值65536

```
 * soft nproc 65536
 * hard nproc 65536
 * soft nofile 65536
 * hard nofile 65536
```

- 执行`sudo sysctl -p`使得设置生效
- 切换到root`sudo bash`
- 再切换回当前用户`su - username`

```
[vip@test ~]$ sudo vi /etc/security/limits.conf
[vip@test ~]$ sudo sysctl -p
net.core.rmem_max = 25165824
net.core.wmem_max = 25165824
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 87380 16777216
net.core.netdev_max_backlog = 2500
net.ipv4.ip_local_port_range = 12000 65535
net.ipv4.conf.all.promote_secondaries = 1
vm.min_free_kbytes = 524288
vm.swappiness = 1
net.ipv4.tcp_tw_reuse = 1
[vip@test ~]$
[vip@test ~]$
[vip@test ~]$ ulimit -n
65500
[vip@test ~]$ sudo bash
[root@test /home/vip]# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 126546
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 65536
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 65536
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
[root@test /home/vip]# su - vip
Last login: Thu Jul 23 10:49:22 CST 2020 on pts/0
[vip@test ~]$ ulimit -n
65536
[vip@test ~]$
```


#### 注意事项

[官方要求](https://docs.sonarqube.org/latest/requirements/requirements/#header-5)

If you're running on Linux, you must ensure that:

- `vm.max_map_count` is greater than or equal to `524288`. check with `sysctl vm.max_map_count`
- `fs.file-max` is greater than or equal to `131072`. check with `sysctl fs.file-max`
- the user running SonarQube can open at least `131072` file descriptors. check with `ulimit -u`
- the user running SonarQube can open at least `8192` threads. check with `ulimit -n`


To set these values more permanently, you must update either `/etc/sysctl.d/99-sonarqube.conf` (or `/etc/sysctl.conf` as you wish) to reflect these values.

If the user running SonarQube (sonarqube in this example) does not have the permission to have at least 131072 open descriptors, you must insert this line in `/etc/security/limits.d/99-sonarqube.conf` (or `/etc/security/limits.conf` as you wish):

```
sonarqube   -   nofile   131072
sonarqube   -   nproc    8192
```

If you are using systemd to start SonarQube, you must specify those limits inside your unit file in the section [service] :

```
[Service]
...
LimitNOFILE=131072
LimitNPROC=8192
...
```

### 搭建SonarQube CS on MacOS

轻车熟路了这次。 

Server端

- 安装JDK 11： [openJDK 11](http://jdk.java.net/archive/) for [MacOS](https://download.java.net/java/GA/jdk11/9/GPL/openjdk-11.0.2_osx-x64_bin.tar.gz)，下载解压即生效（可以不添加环境变量）
- 现在SonarQube完整的zip包，解压后，在conf目录下的wrapper.properties文件中指定上一步的JDK中java所在的目录，类似`~/jdk-11.0.2.jdk/Contents/Home/bin`
- 使用指定的数据库，或原生自动的H2数据库

Scanner端

- 下载[sonarscanner](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/)对应的[mac版本](https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.5.0.2216-macosx.zip)
- 解压并将bin目录添加到环境变量

可以进行完整的验证使用

### 搭建8.3版本SonarQube服务 on Ubuntu

现有服务使用Java8版本，需要新安装Java 11并保留Java8

#### 安装Open JDK 11

` sudo apt-get install -y openjdk-11-jdk`

安装成功后使用`update-java-alternatives -l`查看已经安装版本——

```
java-1.11.0-openjdk-amd64      1111       /usr/lib/jvm/java-1.11.0-openjdk-amd64
java-1.8.0-openjdk-amd64       1081       /usr/lib/jvm/java-1.8.0-openjdk-amd64
```

本地设置的环境变量会覆盖默认设置，例如 `/etc/profile`和 `~/.zshrc`文件中的`JAVA_HOME`值。直接使用 `update-java-alternatives -s java-1.11.0-openjdk-amd64`时甚至会报错——"update-alternatives error no alternatives for mozilla-javaplugin.so"

使用` sudo update-alternatives --config java`进行手动选择来设置

```
➜  ~ java -version
openjdk version "1.8.0_252"
OpenJDK Runtime Environment (build 1.8.0_252-8u252-b09-1~18.04-b09)
OpenJDK 64-Bit Server VM (build 25.252-b09, mixed mode)
➜  ~
➜  ~
➜  ~ sudo update-alternatives --config java
There are 2 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                            Priority   Status
------------------------------------------------------------
  0            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      auto mode
  1            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      manual mode
* 2            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      manual mode

Press <enter> to keep the current choice[*], or type selection number: 1
update-alternatives: using /usr/lib/jvm/java-11-openjdk-amd64/bin/java to provide /usr/bin/java (java) in manual mode
➜  ~
➜  ~ java -version
openjdk version "11.0.7" 2020-04-14
OpenJDK Runtime Environment (build 11.0.7+10-post-Ubuntu-2ubuntu218.04)
OpenJDK 64-Bit Server VM (build 11.0.7+10-post-Ubuntu-2ubuntu218.04, mixed mode, sharing)
➜  ~

```

**注：** 如果不想修改系统默认使用的Java版本，可以在安装SonarQube之后，在` $SONARQUBE-HOME/conf/wrapper.conf `配置选项里指定所用的JVM路径，例如——
```
#当前环境下
wrapper.java.command=/usr/lib/jvm/java-11-openjdk-amd64/bin/java
```

#### 安装8.3.1版本的SonarQube

[官方教程](https://docs.sonarqube.org/latest/setup/install-server/)

下载 [SonarQube downloads](https://www.sonarqube.org/downloads/)，解压后将bin目录添加到环境变量。

注意事项：  
- 不用解压在以数字开头的目录里
- 不要使用root用户运行sonar，可以创建一个用户只用来启动sonar服务

```
postgres:x:125:129:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
sonarqube:x:1001:1001::/home/sonarqube:/bin/sh
```

配置`/conf/sonar.properties `：  
```
# Example for PostgreSQL
sonar.jdbc.username=sonarqube
sonar.jdbc.password=mypassword
sonar.jdbc.url=jdbc:postgresql://localhost/sonardb

# 配置Elasticsearch 数据地址，默认在  $SONARQUBE-HOME/data 
sonar.path.data=/var/sonarqube/data
sonar.path.temp=/var/sonarqube/temp

# 配置web端信息
sonar.web.host=192.0.0.1
sonar.web.port=80
# 起始路径
sonar.web.context=/sonarqube

# 指定Java路径
wrapper.java.command=/usr/lib/jvm/java-11-openjdk-amd64/bin/java
```

最后在bin目录下启动sonar：`./sonar.sh start` 默认的用户名密码为`admin:admin`

最好将SonarQube放在当前用户(非root)用户有权限的目录下。

另外，可以根据[这个说明](https://docs.sonarqube.org/latest/setup/operate-server/)将Sonarqube设置为service、配置到Nginx后面等操作——暂未实操验证。目前这个环境仅作手动验证。

#### 可能遇到问题

- 启动失败，提示`ERROR: [1] bootstrap checks failed [1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]`

[issue fix：](https://github.com/docker-library/elasticsearch/issues/111#issuecomment-268511769) `sudo sysctl -w vm.max_map_count=262144`

- 提示` Failed to create table schema_migrations`，类似下面的内容

```
2020.07.01 16:21:53 ERROR web[][o.s.s.p.PlatformImpl] Web server startup failed
java.lang.IllegalStateException: Failed to create table schema_migrations

```

数据库连接指定了`?currentSchema=my_schema`，删除即可，使用默认的public schema。


### SonarQube 升级原因

现在的版本为7.6，使用中遇到类似`Objects.nonnull`方法无法支撑的问题， [Not validating Objects.nonNull](https://community.sonarsource.com/t/not-validating-objects-nonnull/12942)，官方建议升级到至少7.9版本。

升级的问题：
- 依赖Java 11
- 不再支撑MYSQL
- 数据库迁移必须确保在相同版本下的SonarQube进行 [https://github.com/SonarSource/mysql-migrator](https://github.com/SonarSource/mysql-migrator)


这意味着：先做低版本下的数据库迁移；再做升级动作。

[end-of-life-of-mysql-support](https://community.sonarsource.com/t/end-of-life-of-mysql-support/8667)

配合postgresql设置sonarqube
[https://dunterov.github.io/sq-psql/](https://dunterov.github.io/sq-psql/)


PostgreSQL

下载：
[https://www.enterprisedb.com/download-postgresql-binaries](https://www.enterprisedb.com/download-postgresql-binaries)

[https://www.enterprisedb.com/downloads/postgres-postgresql-downloads](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads)

简易教程：  
[https://www.runoob.com/postgresql/postgresql-create-database.html](https://www.runoob.com/postgresql/postgresql-create-database.html)



### 单module和多module下的Jacoco插件最小配置以及执行

[[Coverage & Test Data] Importing JaCoCo coverage report in XML format](https://community.sonarsource.com/t/coverage-test-data-importing-jacoco-coverage-report-in-xml-format/12151)

[Multi-module Apache Maven example](https://github.com/SonarSource/sonar-scanning-examples/tree/master/sonarqube-scanner-maven/maven-multimodule)

实践检验，亲测有效:) 

插播一下**Phase VS. goal**

Phase(阶段)包含了有一组goals(目标动作)。phase执行有依赖关系。比如执行 `mvn package`， 包含了`validate`, `compile`, `test`, and `package`动作。

goal就是maven用来管理项目的具体的task，可以属于某一个phase，也可以独立存在。比如`mvn dependency:tree`就不属于任何phase。

参考：[maven lifecycle-reference](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#lifecycle-reference)


goal在两种情况下触发：   

- 自动执行（如果plugin的goal默认绑定到了对应的phase，或手动定义了goal在哪个phase执行）
- 通过命令明确执行 `mvn <plugin name>:<goal>`

参考：[Maven plugin execution ID](https://stackoverflow.com/a/33279426/1087122)



插播结束。


#### 单module

比较简单，添加类似下面的配置即可。下面的配置将report重新绑定到了test阶段而已。

`report`默认绑定的阶段是在`verify`阶段。Binds by default to the [lifecycle phase](http://maven.apache.org/ref/current/maven-core/lifecycles.html): `verify`(run any checks to verify the package is valid and meets quality criteria.)

绑定的意思是在对应的phase下才会执行这一goal。

```
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.5</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
执行`mvn clean jacoco:prepare-agent install jacoco:report`执行成功则可以获取到单测文件`target/site/jacoco/jacoco.xml`  
（依次执行了多个动作，既包括maven自带的动作，也包括插件jacoco定义的goal）

#### 多module 

- 首先要在项目的pom文件下添加一个用来聚合的module，例如`utm`
- 项目pom文件依然需要添加jacoco插件（执行时需要在项目的根目录下，没有jacoco插件则无法执行对应的插件动作）依然使用 `report`动作
- 聚合module下需要：
  - a) 聚合所有的依赖模块，参考[JaCoCo-Multi-Module](../../jiansh/us/jacoco-multi-module/)； 
  - b) 定义jacoco的聚合动作，使用`report-aggregate`动作

```
<build>
  <plugins>
      <plugin>
        <groupId>org.jacoco</groupId>
        <artifactId>jacoco-maven-plugin</artifactId>
        <version>0.8.5</version>
        <executions>
          <execution>
            <id>self-define-report-aggregate</id>
            <phase>verify</phase>
            <goals>
                <goal>report-aggregate</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
  </plugins>
</build>
```

依然在项目根目录下执行`mvn clean jacoco:prepare-agent install jacoco:report` 执行成功则可以获取到聚合后的单测文件`utm/target/site/jacoco-aggregate/jacoco.xml`  

插件顺序：  

- 子module如果定义，使用子module的定义
- 子module没有定义的情况下，使用父module的定义，依次类推到根module
- 同一个module下的重复定义会被告警，但以后定义的作为最终生效的定义

### Jacoco Maven Plugin goals

[Jacoco Maven Plugin](https://www.jacoco.org/jacoco/trunk/doc/maven.html)

对应的github项目工程：[github jacoco-maven-plugin](https://github.com/jacoco/jacoco/tree/master/jacoco-maven-plugin)


The JaCoCo Maven plug-in provides the JaCoCo runtime agent to your tests and allows basic report creation.

与surefire插件配合使用时需要注意，surefire插件的`forkCount`参数不能为0。 

>When using the <tt>maven-surefire-plugin</tt> or <tt>maven-failsafe-plugin</tt> you **must not** use a [<tt>forkCount</tt>](http://maven.apache.org/surefire/maven-surefire-plugin/test-mojo.html#forkCount) of <tt>0</tt> or set the [<tt>forkMode</tt>](http://maven.apache.org/surefire/maven-surefire-plugin/test-mojo.html#forkMode) to <tt>never</tt> as this would prevent the execution of the tests with the <tt>javaagent</tt> set and no coverage would be recorded.

**forkCount**

>Option to specify the number of VMs to fork in parallel in order to execute the tests. When terminated with "C", the number part is multiplied with the number of CPU cores. Floating point value are only accepted together with "C". If set to "0", no VM is forked and all tests are executed within the main process.

---

一共定义了11个goals：可以通过配置了`jacoco-maven-plugin`的项目下执行 `mvn help:describe -Dplugin=org.jacoco:jacoco-maven-plugin -Ddetail`获取到详细的说明。或者参考下面的链接说明——

The JaCoCo Maven plug-in defines the following goals:

*   [help](https://www.jacoco.org/jacoco/trunk/doc/help-mojo.html)
*   [prepare-agent](https://www.jacoco.org/jacoco/trunk/doc/prepare-agent-mojo.html)
*   [prepare-agent-integration](https://www.jacoco.org/jacoco/trunk/doc/prepare-agent-integration-mojo.html)
*   [merge](https://www.jacoco.org/jacoco/trunk/doc/merge-mojo.html)
*   [report](https://www.jacoco.org/jacoco/trunk/doc/report-mojo.html)
*   [report-integration](https://www.jacoco.org/jacoco/trunk/doc/report-integration-mojo.html)
*   [report-aggregate](https://www.jacoco.org/jacoco/trunk/doc/report-aggregate-mojo.html)
*   [check](https://www.jacoco.org/jacoco/trunk/doc/check-mojo.html)
*   [dump](https://www.jacoco.org/jacoco/trunk/doc/dump-mojo.html)
*   [instrument](https://www.jacoco.org/jacoco/trunk/doc/instrument-mojo.html)
*   [restore-instrumented-classes](https://www.jacoco.org/jacoco/trunk/doc/restore-instrumented-classes-mojo.html)



### 多模块项目包含war类型依赖

[JaCoCo-Multi-Module](../../jiansh/us/jacoco-multi-module/)强调了如果有子模块为pom类型时，需要将type进行明确说明。

但如果子模块为war类型时，如果不明示，会按照默认的jar类型去查找——不符合实际情况。

这个[问题](https://stackoverflow.com/questions/56071617/getting-coverage-from-war-dependency-in-multi-module-project)描述的是这一现象；

这个[官方jacoco issue #991](https://github.com/jacoco/jacoco/issues/991)提示的解决方案也是增加`<type>war</type>`。

增加了`<type>war</type>`之后，在执行`spring-boot:repackage`任务时，提示“找不到主类”——

>Execution default of goal org.springframework.boot:spring-boot-maven-plugin:1.5.4.RELEASE:repackage failed: Unable to find main class

**解决方案**：用来聚合单侧结果的模块打包方式制定为`<packaging>pom</packaging>`。[有效的打包类型包括](https://maven.apache.org/pom.html#packaging) `pom`, `jar`, `maven-plugin`, `ejb`, `war`, `ear`, `rar`

这样在执行`mvn clean install`时聚合模块不做打包处理。pom类型意味着只有`install`和`deploy`两个[阶段动作](https://maven.apache.org/ref/3.6.3/maven-core/default-bindings.html#Plugin_bindings_for_pom_packaging)

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

###  测试代码被误认为是源码

原因：`sonar.source`参数值未正确设置。

[Coverage info displayed on Test code](https://community.sonarsource.com/t/coverage-info-displayed-on-test-code/30452)

使用sonar-scanner执行扫描动作，Java项目工程指定项目目录作为`sonar.source`参数的值。`sonar.source=.`一直执行的挺好。

最近有项目组反馈“测试目录里*INT的代码”被识别为了源码——导致覆盖率降低。类似在`src/test/java`目录下创建了类似`ServiceXXXINT.java`类型的文件。

看起来是如果测试目录下的文件如果以Test结尾，即使使用之前的设定，也会被识别为测试代码；否则会被认为是源码文件。

官方人员建议：使用 `mvn sonar:sonar`执行，会默认识别出项目结构。
[文档](https://docs.sonarqube.org/latest/analysis/analysis-parameters/)中也做了说明——


`sonar.source` Comma-separated paths to directories containing main source files.  
>Read from build system for Maven, Gradle, MSBuild projects. Defaults to project base directory when neither sonar.sources nor sonar.tests is provided.

将此参数指定到`src/main/java`目录即可解决此问题。 

`mvn sonar:sonar`会自动识别项目结构：[Introduction to the Standard Directory Layout](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)

![](https://upload-images.jianshu.io/upload_images/3296949-265e8fe314e1edf1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 验证环境 

从[7.7版本](https://docs.sonarqube.org/7.7/analysis/coverage/)开始，覆盖(Coverage)包括两个概念： 

1.  Test Coverage 测试覆盖 ——测试结果有多少覆盖率  
2.  Test Execution 测试执行 ——具体包含多少个测试用例

此演示用来说明Java Maven项目

1). 如何执行单元测试；  

2). 如何利用sonar插件完成静态代码扫描。

### 项目内容

项目地址： [https://gitee.com/gebitang/mojo](https://gitee.com/gebitang/mojo) 

只包含三个文件：

*   一个class， https://gitee.com/gebitang/mojo/blob/master/src/main/java/very/basic/demo/mutation/Palindrome.java 
*   一个对应的测试class， https://gitee.com/gebitang/mojo/blob/master/src/test/java/very/basic/demo/mutation/PalindromeUnitTest.java 
*   pom文件(只有一个junit依赖)  https://gitee.com/gebitang/mojo/blob/master/pom.xml 

### 测试环境

本地搭建的8.3版本的SonarQube服务

使用maven自带的maven-surefire-plugin执行单测

使用jacoco插件jacoco-maven-plugin收集单测覆盖结果

使用Sonar-maven-plugin插件执行扫描

## 执行验证

`mvn test`默认绑定的goal是`surefire:test`，使用默认自带的"maven-surefire-plugin"插件执行单元测试(run tests using a suitable unit testing framework. These tests should not require the code be packaged or deployed.)

项目所有依赖的插件列表可以通过执行"mvn help:effective-pom"命令获取到，可以看到默认的插件包括——

1.  maven-clean-plugin   
2. maven-resources-plugin
3.  maven-jar-plugin
4.  maven-compiler-plugin
5.  maven-surefire-plugin
6.  maven-install-plugin
7.  maven-deploy-plugin
8.  maven-site-plugin

### 无jacoco插件执行结果

#### mvn test执行信息

```
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------< very.basic.demo:mojo >------------------------
[INFO] Building mojo 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ mojo ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 0 resource
[INFO]
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ mojo ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to D:\gprojects\us\mojo\target\classes
[INFO]
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ mojo ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory D:\gprojects\us\mojo\src\test\resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ mojo ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to D:\gprojects\us\mojo\target\test-classes
[INFO]
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ mojo ---
[INFO] Surefire report directory: D:\gprojects\us\mojo\target\surefire-reports

-------------------------------------------------------
T E S T S
-------------------------------------------------------
Running very.basic.demo.mutation.PalindromeUnitTest
Tests run: 4, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.045 sec

Results :

Tests run: 4, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1.524 s
[INFO] Finished at: 2020-07-16T11:44:50+08:00
[INFO] ------------------------------------------------------------------------
```

####  SonarQube结果 

在SonarQube的展示上，此时只有 Test Execution的结果，没有覆盖率的结果

![](https://upload-images.jianshu.io/upload_images/3296949-7f5259b4d0ca6860.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###  有jacoco插件执行结果

mvn test执行结果

```
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------< very.basic.demo:mojo >------------------------
[INFO] Building mojo 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ mojo ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 0 resource
[INFO]
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ mojo ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to D:\gprojects\us\mojo\target\classes
[INFO]
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ mojo ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory D:\gprojects\us\mojo\src\test\resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ mojo ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to D:\gprojects\us\mojo\target\test-classes
[INFO]
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ mojo ---
[INFO] Surefire report directory: D:\gprojects\us\mojo\target\surefire-reports

-------------------------------------------------------
T E S T S
-------------------------------------------------------
Running very.basic.demo.mutation.PalindromeUnitTest
Tests run: 4, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.045 sec

Results :

Tests run: 4, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1.524 s
[INFO] Finished at: 2020-07-16T11:44:50+08:00
[INFO] ------------------------------------------------------------------------
```

#### SonarQube结果


添加了jacoco插件并与test阶段绑定后，`mvn test`会同时产生

- （surefire插件生成的）单测结果(Test Execution)
- （jacoco插件生成）单测覆盖率结果(Test Coverage)


此时再执行`sonar:sonar`后，单测结果包括了——

-  “Test Execution 测试执行”；
-  “Test Coverage 测试覆盖”

Sonar插件执行log `sonar:sonar`

```
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------< very.basic.demo:mojo >------------------------
[INFO] Building mojo 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- sonar-maven-plugin:3.3.0.603:sonar (default-cli) @ mojo ---
[INFO] User cache: C:\Users\joechin\.sonar\cache
[INFO] SonarQube version: 8.3.1
[INFO] Default locale: "zh_CN", source code encoding: "UTF-8"
[WARNING] SonarScanner will require Java 11 to run starting in SonarQube 8.x
[INFO] Load global settings
[INFO] Load global settings (done) | time=51ms
[INFO] Server id: 86E1FA4D-AXLG8ocC0I8Fk3NXo4jb
[INFO] User cache: C:\Users\joechin\.sonar\cache
[INFO] Load/download plugins
[INFO] Load plugins index
[INFO] Load plugins index (done) | time=31ms
[INFO] Load/download plugins (done) | time=54ms
[INFO] Process project properties
[INFO] Process project properties (done) | time=7ms
[INFO] Execute project builders
[INFO] Execute project builders (done) | time=2ms
[INFO] Project key: very.basic.demo:mojo
[INFO] Base dir: D:\gprojects\us\mojo
[INFO] Working dir: D:\gprojects\us\mojo\target\sonar
[INFO] Load project settings for component key: 'very.basic.demo:mojo'
[INFO] Load quality profiles
[INFO] Load quality profiles (done) | time=38ms
[INFO] Load active rules
[INFO] Load active rules (done) | time=314ms
[INFO] Indexing files...
[INFO] Project configuration:
[INFO] 3 files indexed
[INFO] 0 files ignored because of scm ignore settings
[INFO] Quality profile for java: Sonar way
[INFO] Quality profile for xml: Sonar way
[INFO] ------------- Run sensors on module mojo
[INFO] Load metrics repository
[INFO] Load metrics repository (done) | time=17ms
[INFO] Sensor JavaSquidSensor [java]
[INFO] Configured Java source version (sonar.java.source): 8
[INFO] JavaClasspath initialization
[WARNING] Bytecode of dependencies was not provided for analysis of source files, you might end up with less precise results. Bytecode can be provided using sonar.java.libraries property.
[INFO] JavaClasspath initialization (done) | time=8ms
[INFO] JavaTestClasspath initialization
[INFO] JavaTestClasspath initialization (done) | time=1ms
[INFO] Java Main Files AST scan
[INFO] 1 source files to be analyzed
[INFO] Load project repositories
[INFO] Load project repositories (done) | time=3ms
[INFO] 1/1 source files have been analyzed
[INFO] Java Main Files AST scan (done) | time=731ms
[INFO] Java Test Files AST scan
[INFO] 1 source files to be analyzed
[INFO] Java Test Files AST scan (done) | time=46ms
[INFO] 1/1 source files have been analyzed
[INFO] Java Generated Files AST scan
[INFO] 0 source files to be analyzed
[INFO] Java Generated Files AST scan (done) | time=1ms
[INFO] 0/0 source files have been analyzed
[INFO] Sensor JavaSquidSensor [java] (done) | time=917ms
[INFO] Sensor SonarCSS Rules [cssfamily]
[INFO] No CSS, PHP, HTML or VueJS files are found in the project. CSS analysis is skipped.
[INFO] Sensor SonarCSS Rules [cssfamily] (done) | time=1ms
[INFO] Sensor JaCoCo XML Report Importer [jacoco]
[INFO] 'sonar.coverage.jacoco.xmlReportPaths' is not defined. Using default locations: target/site/jacoco/jacoco.xml,target/site/jacoco-it/jacoco.xml,build/reports/jacoco/test/jacocoTestReport.xml
[INFO] Importing 1 report(s). Turn your logs in debug mode in order to see the exhaustive list.
[INFO] Sensor JaCoCo XML Report Importer [jacoco] (done) | time=13ms
[INFO] Sensor SurefireSensor [java]
[INFO] parsing [D:\gprojects\us\mojo\target\surefire-reports]
[INFO] Sensor SurefireSensor [java] (done) | time=18ms
[INFO] Sensor JavaXmlSensor [java]
[INFO] 1 source files to be analyzed
[INFO] Sensor JavaXmlSensor [java] (done) | time=93ms
[INFO] 1/1 source files have been analyzed
[INFO] Sensor HTML [web]
[INFO] Sensor HTML [web] (done) | time=2ms
[INFO] Sensor XML Sensor [xml]
[INFO] 1 source files to be analyzed
[INFO] Sensor XML Sensor [xml] (done) | time=78ms
[INFO] 1/1 source files have been analyzed
[INFO] ------------- Run sensors on project
[INFO] Sensor Zero Coverage Sensor
[INFO] Sensor Zero Coverage Sensor (done) | time=0ms
[INFO] Sensor Java CPD Block Indexer
[INFO] Sensor Java CPD Block Indexer (done) | time=8ms
[INFO] SCM Publisher SCM provider for this project is: git
[INFO] SCM Publisher 3 source files to be analyzed
[INFO] SCM Publisher 2/3 source files have been analyzed (done) | time=72ms
[WARNING] Missing blame information for the following files:
[WARNING] * pom.xml
[WARNING] This may lead to missing/broken features in SonarQube
[INFO] CPD Executor 1 file had no CPD blocks
[INFO] CPD Executor Calculating CPD for 0 files
[INFO] CPD Executor CPD calculation finished (done) | time=0ms
[INFO] Analysis report generated in 612ms, dir size=85 KB
[INFO] Analysis report compressed in 90ms, zip size=15 KB
[WARNING] locking FileBasedConfig[C:\Users\joechin\.config\jgit\config] failed after 5 retries
[INFO] Analysis report uploaded in 794ms
[INFO] ANALYSIS SUCCESSFUL, you can browse [http://127.0.0.1:9000/dashboard?id=very.basic.demo%3Amojo](http://127.0.0.1:9000/dashboard?id=very.basic.demo%3Amojo)
[INFO] Note that you will be able to access the updated dashboard once the server has processed the submitted analysis report
[INFO] More about the report processing at [http://127.0.0.1:9000/api/ce/task?id=AXNWNqQPI0wKADe8C9RC](http://127.0.0.1:9000/api/ce/task?id=AXNWNqQPI0wKADe8C9RC)
[INFO] Analysis total time: 9.434 s
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 10.902 s
[INFO] Finished at: 2020-07-16T14:01:28+08:00
[INFO] ------------------------------------------------------------------------

```

####  SonarQube web展示

![](https://upload-images.jianshu.io/upload_images/3296949-6b4df882f5ae2b7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
