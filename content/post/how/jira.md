+++
title = "JIRA API"
description = "Everything about Jira api operation"
tags = [
    "jira"
]
date = "2021-08-05"
topics = [
    "jira"
]
toc = true
+++

- [golang jira client api](https://github.com/andygrunwald/go-jira)
- [Simple command line client for Atlassian's Jira service written in Go.](https://github.com/go-jira/jira)
## 搜索 

### JIRA 获取查询语句
[https://community.atlassian.com/t5/Jira-questions/How-do-I-retrieve-issues-of-specific-status-with-JQL/qaq-p/540468](https://community.atlassian.com/t5/Jira-questions/How-do-I-retrieve-issues-of-specific-status-with-JQL/qaq-p/540468)

An easy way to get the proper encoding is to

1.  build your search in JIRA first
2.  When you have the results you want, click the Views button in Issue Navigator
3.  Right-click the XML link and copy the link address. It has all the proper formatting already.
4.  Paste everything that comes after 'jqlQuery=' into your code and add your variables and such.


[https://developer.atlassian.com/server/jira/platform/jira-rest-api-examples/](https://developer.atlassian.com/server/jira/platform/jira-rest-api-examples/)

### 排序逻辑

[按照问题id大小排序](https://community.atlassian.com/t5/Jira-questions/Sort-by-request-number/qaq-p/704645)， 类似`project = projectKey AND statusCategory = Done AND  issuetype in (线上缺陷, Bug)  ORDER BY issuekey DESC`。其中的 `issuekey`为多个可排序的自定义字段，可以从`/rest/api/2/field`查看到哪些字段可参与排序，例如`resolutiondate`解决时间排序

[jql高级搜索api](https://developer.atlassian.com/server/confluence/advanced-searching-using-cql/#AdvancedSearchingusingCQL-ORDERBY)，[jql: Jira Query Language](https://www.idalko.com/jira-jql/)——待实践

## 其他操作

### 创建issue

必选项：summary、description、issuetype、reporter
可选项：assignee、labels


[https://developer.atlassian.com/cloud/jira/platform/rest/v2/#api-rest-api-2-issue-post](https://developer.atlassian.com/cloud/jira/platform/rest/v2/#api-rest-api-2-issue-post)

项目实例：
[https://bitbucket.org/atlassian/atlassian-connect-spring-boot/src/master/README.md](https://bitbucket.org/atlassian/atlassian-connect-spring-boot/src/master/README.md)

Basic 验证方式：
[https://developer.atlassian.com/server/jira/platform/basic-authentication](https://developer.atlassian.com/server/jira/platform/basic-authentication)
需要对用户名和密码做Base64转码， 例如 `fred:fred`转码为`ZnJlZDpmcmVk`

```
curl -H "Authorization: Basic ZnJlZDpmcmVk" -X GET -H "Content-Type: application/json" http://localhost:8080/rest/api/2/issue/createmeta
```

Basic 验证下的API调研
[https://developer.atlassian.com/cloud/jira/platform/basic-auth-for-rest-apis/](https://developer.atlassian.com/cloud/jira/platform/basic-auth-for-rest-apis/)


### 修改issue状态

先查询JIRA下对应的transition的定义
[https://developer.atlassian.com/cloud/jira/platform/rest/v3/?#api-rest-api-3-issue-issueIdOrKey-transitions-get](https://developer.atlassian.com/cloud/jira/platform/rest/v3/?#api-rest-api-3-issue-issueIdOrKey-transitions-get)

`GET /rest/api/3/issue/{issueIdOrKey}/transitions`

根据查询到的transitions进行操作
[https://developer.atlassian.com/cloud/jira/platform/rest/v3/?#api-rest-api-3-issue-issueIdOrKey-transitions-post](https://developer.atlassian.com/cloud/jira/platform/rest/v3/?#api-rest-api-3-issue-issueIdOrKey-transitions-post)


```
ObjectNode payload = jnf.objectNode();
ObjectNode transition = payload.putObject("transition");
 transition.put("id", "401"); 
HttpResponse<JsonNode> response = Unirest.post(String.format("http://project.jira-corp.com/rest/api/2/issue/%s/transitions", id))
                    .basicAuth("user", "password")
                    .header("Accept", "application/json")
                    .header("Content-Type", "application/json")
                    .body(payload)
                    .asJson();
```

