+++
title = "Sonar-Web-api(一)"
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



[link on JianShu](https://www.jianshu.com/p/b15e364bc8f8)

[数据库定义](https://github.com/SonarSource/sonarqube/blob/4.5.5/sonar-core/src/main/resources/org/sonar/core/persistence/schema-h2.ddl)，但 [不建议](https://stackoverflow.com/a/9202474/1087122)直接查询DB，使用Sonar提供的Web_Api，默认在SonarQube服务的/web_api下可以查看到提供的所有API信息，如`http://localhost:9000/web_api/api/project_analyses`

[查询示例1](https://stackoverflow.com/a/53526094/1087122)
[查询示例2](https://stackoverflow.com/questions/51526635/sonarqube-rest-apis-read-metrics-for-individual-projects)

---

 对应的属性含义参考： https://docs.sonarqube.org/7.6/user-guide/metric-definitions/ 

数据来源方法：
0. 获取项目总数：
 
http://example.sonar.com/api/components/search_projects?pageIndex=2&ps=500


1. 获取最近一次的分析结果时间：

API:  /api/components/search_projects
Para: 
    ps? #大小，最大500
    f? #
example:    
http://example.sonar.com/api/components/search_projects?ps=50&f=analysisDate
result:  
``` 
{
    "paging":{
        "pageIndex":1,
        "pageSize":50,
        "total":2337
    },
    "organizations":[

    ],
    "components":Array[50],
    "facets":[
    ]
}

# 其中components中包含项目name 和时间戳，如
{
    "organization":"default-organization",
    "id":"AW0j4CH4yaNpfFfxGPl2",
    "key":"projectGroup1:projectNameExample",
    "name":"projectGroup1:projectNameExample",
    "isFavorite":false,
    "analysisDate":"2019-11-27T20:37:05+0800",
    "tags":[],
    "visibility":"public"
}
```

2. 根据component，查询不同的metrics： 
API:  /api/measures/search
Para: 
    projectKeys? # 逗号分隔多个项目
    metricKeys? # 查询的各个标准
example: 
http://example.sonar.com/api/measures/search?projectKeys=projectGroup1:projectNameExample&metricKeys=alert_status,bugs,reliability_rating,vulnerabilities,security_rating,code_smells,sqale_rating,duplicated_lines_density,coverage,ncloc,ncloc_language_distribution
result:
```
{
    "measures":[
        {
            "metric":"alert_status",
            "value":"OK",
            "component":"projectGroup1:projectNameExample"
        },
        {
            "metric":"bugs",
            "value":"6",
            "component":"projectGroup1:projectNameExample",
            "bestValue":false
        },
        {
            "metric":"code_smells",
            "value":"185",
            "component":"projectGroup1:projectNameExample",
            "bestValue":false
        },
        {
            "metric":"coverage",
            "value":"0.0",
            "component":"projectGroup1:projectNameExample",
            "bestValue":false
        },
        {
            "metric":"duplicated_lines_density",
            "value":"64.6",
            "component":"projectGroup1:projectNameExample",
            "bestValue":false
        },
        {
            "metric":"ncloc",
            "value":"14566",
            "component":"projectGroup1:projectNameExample"
        },
        {
            "metric":"ncloc_language_distribution",
            "value":"java=4756;js=9582;web=228",
            "component":"projectGroup1:projectNameExample"
        },
        {
            "metric":"reliability_rating",
            "value":"3.0",
            "component":"projectGroup1:projectNameExample",
            "bestValue":false
        },
        {
            "metric":"security_rating",
            "value":"2.0",
            "component":"projectGroup1:projectNameExample",
            "bestValue":false
        },
        {
            "metric":"sqale_rating",
            "value":"1.0",
            "component":"projectGroup1:projectNameExample",
            "bestValue":true
        },
        {
            "metric":"vulnerabilities",
            "value":"27",
            "component":"projectGroup1:projectNameExample",
            "bestValue":false
        }
    ]
} 
```

3. 查询增量内容——


项目新增的内容：
http://example.sonar.com/api/measures/component?additionalFields=metrics%2Cperiods&component=projectGroup1%3AprojectNameExample&metricKeys=alert_status%2Cquality_gate_details%2Cbugs%2Cnew_bugs%2Creliability_rating%2Cnew_reliability_rating%2Cvulnerabilities%2Cnew_vulnerabilities%2Csecurity_rating%2Cnew_security_rating%2Ccode_smells%2Cnew_code_smells%2Csqale_rating%2Cnew_maintainability_rating%2Csqale_index%2Cnew_technical_debt%2Ccoverage%2Cnew_coverage%2Cnew_lines_to_cover%2Ctests%2Cduplicated_lines_density%2Cnew_duplicated_lines_density%2Cduplicated_blocks%2Cncloc%2Cncloc_language_distribution%2Cprojects%2Cnew_lines


---

接口调用可以使用Basic的认证方式：[参考](https://stackoverflow.com/questions/22244163/how-do-i-pass-credentials-to-sonar-api-calls)

-  [Use token](https://docs.sonarqube.org/7.6/user-guide/user-token/)
- `curl -u admin:SuPeRsEcReT "https://sonar.mydomain.com/api/resources?resource=com.mydomain.project:MY&metrics=ncloc&format=json"`
- `curl -u THIS_IS_MY_TOKEN: https://sonarqube.com/api/user_tokens/search`
- note that the colon after the token is required in curl to set an empty password 


---

[URL 转换](http://tools.jb51.net/transcoding/urlencode_decode)  

[在线JSON解析](https://www.json.cn/)





