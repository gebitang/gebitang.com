+++
title = "Sonar实践问题记录(五)-插件Plugins"
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



[link on JianShu](https://www.jianshu.com/p/36cee9b46c16)

插件也需要与主版本兼容。 例如7.6版本的中文包需要使用`sonar-l10n-zh-plugin-1.26.jar` , 8.0版本的sonarqube使用的为`sonar-l10n-zh-plugin-8.0.jar`

最新版本的plugin兼容性列表可参考[官方说明 Plugin Version Matrix](https://docs.sonarqube.org/latest/instance-administration/plugin-version-matrix/)

SonarQube 7.6 默认的插件包括**19**个：
>sonar-csharp-plugin-7.10.0.7896.jar  
 sonar-css-plugin-1.0.3.724.jar  
 sonar-flex-plugin-2.4.0.1222.jar  
 sonar-go-plugin-1.1.0.1612.jar  
 sonar-html-plugin-3.1.0.1615.jar  
 sonar-jacoco-plugin-1.0.1.143.jar  
 sonar-java-plugin-5.10.1.16922.jar  
 sonar-javascript-plugin-5.0.0.6962.jar  
 sonar-kotlin-plugin-1.4.0.155.jar  
 sonar-ldap-plugin-2.2.0.608.jar  
 sonar-php-plugin-2.16.0.4355.jar  
 sonar-python-plugin-1.11.0.2473.jar  
 sonar-ruby-plugin-1.4.0.155.jar  
 sonar-scala-plugin-1.4.0.155.jar  
 sonar-scm-git-plugin-1.7.0.1491.jar  
 sonar-scm-svn-plugin-1.9.0.1295.jar  
 sonar-typescript-plugin-1.9.0.3766.jar  
 sonar-vbnet-plugin-7.10.0.7896.jar  
 sonar-xml-plugin-2.0.1.2020.jar  

目前需要额外安装的包括：
- gitlab 插件`sonar-gitlab-plugin-4.1.0-SNAPSHOT.jar` 4.0版本与7.6版本的sonarqube不兼容
- 中文包 `sonar-l10n-zh-plugin-1.26.jar`
- 代码常用的扫描规则 `sonar-pmd-plugin-3.2.1.jar` 见[详细说明](https://github.com/jensgerdes/sonar-pmd/blob/master/docs/RULES.md)
- Xanitizer公司提供的安全扫描分析插件 `sonar-xanitizer-plugin-2.0.0.jar` 
