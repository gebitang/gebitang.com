+++
title = "Sonar实践问题记录(九)：sonar-scanner分析参数"
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



[link on JianShu](https://www.jianshu.com/p/ba30b4a927cf)

>ERROR: Validation of project reactor failed:
  "groupName:projectName:BranchName-22695-合规视频" is not a valid project or module key. Allowed characters are alphanumeric, '-', '_', '.' and ':', with at least one non-digit.
ERROR: 
ERROR: Re-run SonarQube Scanner using the -X switch to enable full debug logging.

一直提示类似这样的问题，因为最后一部分来自gitlab工程的branch名称——第一反应是“不应该使用中文作为branch名称啊”。

- 之前调研api时印象里有看到 key、name都是一致的结果。
- web端展示不允许中文也说不过去

查里一下参数，[analysis parameters](https://docs.sonarqube.org/7.6/analysis/analysis-parameters/)就明白了。

核心参数包括三个：
- server端一个：指定server端的url地址
- 项目配置两个：projectKey和source

### Server

| Key | Description | Default |
| --- | --- | --- |
| `sonar.host.url` | the server URL | [http://localhost:9000](http://localhost:9000/) |

###  Project Configuration
| Key | Description | Default |
| --- | --- | --- |
| `sonar.projectKey` | The project's unique key. Allowed characters are: letters, numbers, - , _ , . and : , with at least one non-digit.|For Maven projects, this is automatically set to <groupId>:<artifactId>|
| `sonar.sources` | Comma-separated paths to directories containing source files.|Read from build system for Maven, Gradle, MSBuild projects|

Web端展示的来自与可选参数`sonar.projectName`，如果没有设置，命令行执行时默认使用`sonar.projectKey` 的值。

所以只需要给这两个值分别赋值即可。

根据业务场景，需要确保兼容性。


