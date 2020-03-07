+++
title = "Basic-SonarQube-SonarScanner"
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



[link on JianShu](https://www.jianshu.com/p/0b94bc843357)

[SonarQube](https://www.sonarsource.com/products/sonarqube/) Continuous code quality made easy.

### Start In Two Minute 

1. 下载 [SonarQube downloads](https://www.sonarqube.org/downloads/)，使用Sonar qube 7.6版本，Java8支持；最新版本8.0，需要Java 11+  
2. 解压后将bin目录添加到环境变量  
3. 启动SonarQube服务 (默认会使用内置的数据库，可单独[配置数据库](https://docs.sonarqube.org/latest/setup/install-server/))
```
# On Windows, execute:
C:\sonarqube\bin\windows-x86-xx\StartSonar.bat

# On other operating systems, as a non-root user execute:
/opt/sonarqube/bin/[OS]/sonar.sh console

# server log
...
jvm 1    | 2019.11.20 10:38:32 INFO  app[][o.s.a.SchedulerImpl] Process[ce] is up
jvm 1    | 2019.11.20 10:38:32 INFO  app[][o.s.a.SchedulerImpl] SonarQube is up
```
启动后，在`localhost:9000`可以访问web页面，默认(admin/admin)登录。

4. 下载 [SonarScanner](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/) 

**Download SonarScanner 4.2** - Compatible with SonarQube 6.7+ (LTS) By [SonarSource](https://www.sonarsource.com/) – GNU LGPL 3 – [Issue Tracker](https://jira.sonarsource.com/browse/SQSCANNER) – [Source](https://github.com/Sonarsource/sonar-scanner-cli)
[Linux 64-bit](https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.2.0.1873-linux.zip) | [Windowx 64-bit](https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.2.0.1873-windows.zip) | [Mac OS X 64-bit](https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.2.0.1873-macosx.zip) | [Any*](https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.2.0.1873.zip) *Requires a pre-installed JVM - with the same requirements as the SonarQube server.

5. 下载示例工程
To help you get started, simple project samples are available for most languages on github. They can be [browsed](https://github.com/SonarSource/sonar-scanning-examples) or [downloaded](https://github.com/SonarSource/sonar-scanning-examples/archive/master.zip). You'll find them filed under sonarqube-scanner/src.

6. 执行扫描（需要确保SnoarQube已经运行起来）

在对应的工程下配置好`sonar-project.properties`文件，然后在当前目录下执行 `sonar-scanner.bat -X`，执行成功后会自己将结果上报到SonarQube

```
10:45:11.372 INFO: Analysis total time: 11.075 s
10:45:11.495 INFO: ------------------------------------------------------------------------
10:45:11.496 INFO: EXECUTION SUCCESS
10:45:11.496 INFO: ------------------------------------------------------------------------
10:45:11.497 INFO: Total time: 13.632s
10:45:11.532 INFO: Final Memory: 15M/80M
10:45:11.532 INFO: ------------------------------------------------------------------------
```

![Result Example](https://upload-images.jianshu.io/upload_images/3296949-6c4678840ffd2fa2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Result Example](https://upload-images.jianshu.io/upload_images/3296949-99901c244818f79e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
