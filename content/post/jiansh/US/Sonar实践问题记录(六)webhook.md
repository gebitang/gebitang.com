+++
title = "Sonar实践问题记录(六)webhook"
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



[link on JianShu](https://www.jianshu.com/p/b2d13c638b20)

[sonarqube7.6 webhook](https://docs.sonarqube.org/7.6/project-administration/webhooks/)

使用SonarScanner扫描结束后，会将结果提交给SonarQube，其中的Computer Engine会负责分析数据——这会消耗一定的时间。尤其是免费版本只支持一个Worker工作，不可避免会有排队现象。

项目之前的实现，会使用API`/api/ce`轮询查找提交任务的结果。
- 实践发现，这个请求出现超时现象，即使时长已经设置到1小时；
- 占用资源，线程池执行任务的话，扫描已经完成，但一直没有释放资源

显然webhook是正确的方式。

- 回调列表保留30天
- 回调接口10秒未返回结果会被认为失败

支持定制化参数，在scanner的参数里增加`sonar.analysis.*`的格式即可。下面是一个payload样例，定制化内容会在properties字段里记录。

```
{
  "serverUrl": "http://localhost:8181",
  "taskId": "AW8iCUsHmhhaRoNHDPY8",
  "status": "SUCCESS",
  "analysedAt": "2019-12-20T14:40:08+0800",
  "changedAt": "2019-12-20T14:40:08+0800",
  "project": {
    "key": "aftermarket:quote-service",
    "name": "aftermarket-quote-service",
    "url": "http://localhost:8181/dashboard?id=groupName%3AprojectName"
  },
  "branch": {
    "name": "master",
    "type": "LONG",
    "isMain": true,
    "url": "http://localhost:8181/dashboard?id=groupName%3AprojectName"
  },
  "qualityGate": {
    "name": "remove coverage",
    "status": "OK",
    "conditions": [
      {
        "metric": "new_reliability_rating",
        "operator": "GREATER_THAN",
        "value": "3",
        "status": "OK",
        "errorThreshold": "4"
      },
      {
        "metric": "new_security_rating",
        "operator": "GREATER_THAN",
        "value": "1",
        "status": "OK",
        "errorThreshold": "4"
      }
    ]
  },
  "properties": {
    "sonar.analysis.git.path": "http://git.fxxktest.com/groupName/projectName.git",
    "sonar.analysis.suResultId": "45510",
    "sonar.analysis.host": "sonar.test.site.com",
    "sonar.analysis.git.commit": "bd5603509a4cddcbb1c74820769c939de0a7dea7",
    "sonar.analysis.suScanTime": "2",
    "sonar.analysis.git.branch": "online",
    "sonar.analysis.git.noahPrj": "false",
    "sonar.analysis.suResultStartTime": "1576824002783",
    "sonar.analysis.suType": "true",
    "sonar.analysis.startSonarTime": "1576824006360",
    "sonar.analysis.git.projectId": "5182"
  }
}
```
