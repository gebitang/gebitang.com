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

