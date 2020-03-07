+++
title = "Sonar实践问题记录(二)"
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



[link on JianShu](https://www.jianshu.com/p/34fb445ce96d)

[查询字段的时间戳格式：](https://community.sonarsource.com/t/createdafter-parameter-in-api-issues-search-not-working-for-the-datetime/11930/2)
>Hi, the correct format is yyyy-MM-dd'T'HH:mm:ssZ, so in your example createdAfter=2017-10-19T13:00:00+0200 is correct, but you need to url-encode the + character to %2b : createdAfter=2017-10-19T13:00:00%2b0200

--- 
Issue包括：Bug、Vulnerability、Code_Smell

如：查询包含了Blocker级别的所有Issue：
http://sonar.guazi-corp.com/api/issues/search?componentKeys=se%3Asearch-frontend-springboot&resolved=false&ps=500&createdAfter=2019-10-22&severities=BLOCKER&types=BUG,VULNERABILITY,CODE_SMELL

返回的内容将包括具体的内容。

---
[SonarQube 指标定义](https://blog.csdn.net/justyman/article/details/88761590)  几个主要的查询API

**api/issues**  
Read and update issues.

**api/measures**
Get components or children with specified measures.

**api/metrics**  
Get information on automatic metrics, and manage custom metrics. See also api/custom_measures.
