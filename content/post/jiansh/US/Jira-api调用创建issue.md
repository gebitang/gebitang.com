+++
title = "Jira-api调用创建issue"
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



[link on JianShu](https://www.jianshu.com/p/ad0561577bb8)

- gitlab api获取触发当前pipeline任务的commit信息的人员
[http://git.guazi-corp.com/help/api/commits.md#get-a-single-commit](http://git.guazi-corp.com/help/api/commits.md#get-a-single-commit)

- Jira api调用
创建issue
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

