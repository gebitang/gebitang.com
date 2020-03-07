+++
title = "gitlab更新project配置、JIRA查询及更新ISSUE状态"
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



[link on JianShu](https://www.jianshu.com/p/069d93a6cfbf)

#### gitlab 更新project的配置：
[https://docs.gitlab.com/ce/api/projects.html#edit-project](https://docs.gitlab.com/ce/api/projects.html#edit-project)

更新项目的设置
```
PUT /projects/:id
```
获取项目的设置：
```
GET /projects/:id
```


#### JIRA 获取查询语句
[https://community.atlassian.com/t5/Jira-questions/How-do-I-retrieve-issues-of-specific-status-with-JQL/qaq-p/540468](https://community.atlassian.com/t5/Jira-questions/How-do-I-retrieve-issues-of-specific-status-with-JQL/qaq-p/540468)

An easy way to get the proper encoding is to

1.  build your search in JIRA first
2.  When you have the results you want, click the Views button in Issue Navigator
3.  Right-click the XML link and copy the link address. It has all the proper formatting already.
4.  Paste everything that comes after 'jqlQuery=' into your code and add your variables and such.

--- 

[https://developer.atlassian.com/server/jira/platform/jira-rest-api-examples/](https://developer.atlassian.com/server/jira/platform/jira-rest-api-examples/)


#### 修改issue状态

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
