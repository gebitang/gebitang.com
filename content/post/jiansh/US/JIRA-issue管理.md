+++
title = "JIRA-issue管理"
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



[link on JianShu](https://www.jianshu.com/p/213fbef7a9cc)

#### 创建ISSUE
[调用API创建issue](https://www.jianshu.com/p/ad0561577bb8)

[https://developer.atlassian.com/cloud/jira/platform/rest/v2/#api-rest-api-2-issue-post](https://developer.atlassian.com/cloud/jira/platform/rest/v2/#api-rest-api-2-issue-post)


必选项：summary、description、issuetype、reporter
可选项：assignee、labels

#### 搜索ISSUE
[https://developer.atlassian.com/cloud/jira/platform/rest/v2/#api-rest-api-2-search-get](https://developer.atlassian.com/cloud/jira/platform/rest/v2/#api-rest-api-2-search-get)

查看单个issue：
http://project.jira.com/rest/api/2/issue/531214

搜索：
http://project.jira.com/rest/api/2/search?jql=project=SONARUT&fields=summary,description,labels


#### 更新ISSUE(只能更新issue内容)
[https://developer.atlassian.com/cloud/jira/platform/rest/v2/#api-rest-api-2-issue-issueIdOrKey-put](https://developer.atlassian.com/cloud/jira/platform/rest/v2/#api-rest-api-2-issue-issueIdOrKey-put)

```
// This code sample uses the  'Unirest' library:
// http://unirest.io/java.html
HttpResponse<JsonNode> response = Unirest.put("/rest/api/2/issue/{issueIdOrKey}")
  .basicAuth("email@example.com", "<api_token>")
  .header("Accept", "application/json")
  .header("Content-Type", "application/json")
  .body(payload)
  .asJson();

System.out.println(response.getBody());
```
