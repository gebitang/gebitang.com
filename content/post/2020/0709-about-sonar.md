+++
title = "Sonar info"
description = "SonarQube 信息整理"
tags = [
    "sonar",
    "tech"
]
date = "2020-07-07"
topics = [
    "tech"
]
toc = true
+++

[7月7，SonarQube发布8.4版本](https://www.sonarqube.org/sonarqube-8-4/) ，历史、团队、社区 [中文插件已支持](https://github.com/SonarQubeCommunity/sonar-l10n-zh/commit/ebd423699f63d250ff0589fb40923639bdc48c06)

[code quality](https://www.sonarsource.com/why-us/code-quality/)质量维度：

- Maintainability 可维护性  
- Reliability 可靠性
- Security 安全性

对应： 

- Code smell 异味
- Bug bug
- Vulnerability  漏洞 These issues are well documented in lists maintained by [CWE](https://cwe.mitre.org/) and [CERT](https://www.cert.org/)

CWE：Common Weakness Enumeration. 

> CWE  is a community-developed list of software and hardware weakness types. It serves as a common language, a measuring stick for security tools, and as a baseline for weakness identification, mitigation, and prevention efforts.

CERT：[Computer Emergency Response Team](https://en.wikipedia.org/wiki/Computer_emergency_response_team) 卡梅隆大学 CERT Division，注册商标。 

[CNCERT/CC](http://www.cert.org.cn/)国家互联网应急中心

[Blog：My Consulting Journey at SonarSource](https://blog.sonarsource.com/my-consulting-journey-at-sonarsource) 提到——

[OWASP top ten](https://owasp.org/www-project-top-ten) Open Web Application Security Project  
[top ten 源码信息](https://github.com/OWASP/www-project-top-ten/blob/master/index.md)通常特指：“10项最严重的 Web 应用程序安全风险”，这里可以看到[中文版本](https://wiki.owasp.org/images/d/dc/OWASP_Top_10_2017_%E4%B8%AD%E6%96%87%E7%89%88v1.3.pdf)


--- 

Community EditionVersion 7.6 (build 21501) 

```
# Java Total 906
Bug	127
Vulnerability	123
Code Smell	624
Security Hotspot	32

## Type
SonarAnalyzerJava	529
PMDJava	268
XanitizerJava	76
PMD Unit TestsJava	17
MyCompany Custom RepositoryJava	10
Common JavaJava	6
```

```
# Go Total 46
Bug	11
Vulnerability	2
Code Smell	33

## Type 
SonarAnalyzerGo	40
Common GoGo	6
```

Community EditionVersion 8.3.1 (build 34397) 

```
# Java Total 555 
Bug	125
Vulnerability	47
Code Smell	349
Security Hotspot	34

## Type
SonarAnalyzerJava	549
Common JavaJava	6
```

```
#Go total 44 
Bug	8
Vulnerability	2
Code Smell	34

## Type 
SonarAnalyzerGo	38
Common GoGo	6
```

