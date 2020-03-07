+++
title = "Sonar实践问题记录(三)-SonarQube"
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



[link on JianShu](https://www.jianshu.com/p/551f6aae53dd)

8.0版本需要Java 11，java8环境启动时就会提示。可以[下载7.6](https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.6.zip)版本

###架构
[Sonarqube Architecture](https://docs.sonarqube.org/7.6/architecture/architecture-integration/)

1. One SonarQube Server starting 3 main processes:

  - Web Server for developers, managers to browse quality snapshots and configure the SonarQube instance
  - Search Server based on Elasticsearch to back searches from the UI
  - Compute Engine Server in charge of processing code analysis reports and saving them in the SonarQube Database

2. One SonarQube Database to store:
  - the configuration of the SonarQube instance (security, plugins settings, etc.)
  - the quality snapshots of projects, views, etc.
3. Multiple SonarQube Plugins installed on the server, possibly including language, SCM, integration, authentication, and governance plugins
4. One or more SonarScanners running on your Build / Continuous Integration Servers to analyze projects

简单理解：

1. SonarQube包括三个主要进程：Web Server（UI 展示管理）、本地的ES进程（支持UI上的搜索功能）、分析结果的引擎（Compute Engine Server）

2. 数据库，默认使用自带的H2数据库，可连接外部的数据库，[install server](https://docs.sonarqube.org/7.6/setup/install-server/)支持Microsoft SQL Server、Oracle、PostSQL、Mysql（最新的版本似乎不支持了，7.6版本还可以）

3. 插件支持，如中文包支持。  
4. SonarScanner，执行扫描动作的组件。  


[优化 Compute Engine Server](https://docs.sonarqube.org/latest/instance-administration/compute-engine-performance/) CE版本，只有一个Worker执行任务分析的动作 `Administration > Projects > Background Tasks > Number of Workers`

###限制
- 只有一个SonarQube Server和一个对应的DB（意味着不能交叉使用）
- The SonarQube Platform cannot have more than one SonarQube Server and one SonarQube Database.
- 各个组件独立部署（建议）
- For optimal performance, each component (server, database, scanners) should be installed on a separate machine, and the server machine should be dedicated.
- Scanner随意扩展
- SonarScanners scale by adding machines.
- 所有的机器上做好时间同步
- All machines must be time synchronized.
- SonarQube和DB在同一个网络
- The SonarQube Server and the SonarQube Database must be located in the same network
- Scanner和SonarQube Server不需要在同一个网络（通过api调用传递结果）
- SonarScanners don't need to be on the same network as the SonarQube Server.
- Scanner不会直接与DB打交道
- There is no communication between SonarScanners and the SonarQube Database.


--- 

查看系统类型 `cat /proc/version` 确定为Centos，安装 unzip软件 `sudo yum install unzip`， 解压 sonarqube压缩文件 `unzip sonarqube-7.6.zip`

[Running SonarQube as a Service on Linux with SystemD](https://docs.sonarqube.org/7.6/setup/operate-server/) 

配置conf文件夹下的`sonar.properties`文件——

- 配置数据库连接：连接Mysql需要使用 5.6 < version < 8.0
- 配置web服务端口号 `sonar.web.port`
- 配置search端口号 `sonar.search.port`

###问题

1. 在linux环境下使用，不要使用root用户启动 `sonar.sh console`，会提示 错误类似——`SonarQube is stopped: [o.s.a.p.AbstractManagedProcess] Process exited with exit value [es]: 1`
可以在logs文件夹下查看对应的log信息。其中的`es.log`会显示无法使用root用户启动Elasticsearch  

切换用户后，可能导致一下文件的权限会受限，确保删除了使用root用户创建的文件。可以删除 temp文件夹的所有内容

2. 提示错误`Sonarqube Process exited with exit value [es]: 143`ws服务启动失败。可以查看log文件夹下的 `web.log`   
我遇到的情况是数据库连接问题：无法连接数据库

连接示例：`sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false`  
指定数据库即可。会自动创建表

3. 语言显示，中文语言包需要使用插件进行安装。Administration --> Marketplace中搜索 `Chinese Pack`进行安装。默认会根据浏览器的语言设置显示。

### 社区
可以到这里搜索相关问题——[https://community.sonarsource.com/](https://community.sonarsource.com/)
