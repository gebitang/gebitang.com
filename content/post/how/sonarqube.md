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
