+++
title = "Sonar-Web-api(二)"
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



[link on JianShu](https://www.jianshu.com/p/aa947abf0e69)

每个项目主页：
http://example.sonar.com/dashboard?id=growth_client:restful-api-seller

主页的核心数据来源：
http://example.sonar.com/api/measures/component?additionalFields=metrics,periods&component=projectGroup1:projectNameExample&metricKeys=alert_status,quality_gate_details,bugs,new_bugs,reliability_rating,new_reliability_rating,vulnerabilities,new_vulnerabilities,security_rating,new_security_rating,code_smells,new_code_smells,sqale_rating,new_maintainability_rating,sqale_index,new_technical_debt,coverage,new_coverage,new_lines_to_cover,tests,duplicated_lines_density,new_duplicated_lines_density,duplicated_blocks,ncloc,ncloc_language_distribution,projects,new_lines

最近一次current扫描时间信息：
http://example.sonar.com/api/ce/component?component=projectGroup1:projectNameExample
```
{
    "queue":[

    ],
    "current":{
        "id":"AW7LG4x5rEHtiKcxk1nN",
        "type":"REPORT",
        "componentId":"AWvpaqUHahV-mFVL-A1-",
        "componentKey":"projectGroup1:projectNameExample",
        "componentName":"projectGroup1:projectNameExample",
        "componentQualifier":"TRK",
        "analysisId":"AW7LG5GFQzwg82DVBAfm",
        "status":"SUCCESS",
        "submittedAt":"2019-12-03T17:33:30+0800",
        "submitterLogin":"admin",
        "startedAt":"2019-12-03T17:33:31+0800",
        "executedAt":"2019-12-03T17:33:33+0800",
        "executionTimeMs":1694,
        "logs":false,
        "hasScannerContext":true,
        "organization":"default-organization",
        "warningCount":1,
        "warnings":[

        ]
    }
}
```


查询metric的历史数据：ps参数表示返回几次的数据
http://example.sonar.com/api/measures/search_history?component=projectGroup1:projectNameExample&metrics=sqale_index,duplicated_lines_density,ncloc,coverage,bugs,code_smells,vulnerabilities&ps=1000
```
{
    "paging":{
        "pageIndex":1,
        "pageSize":1000,
        "total":20
    },
    "measures":[
        {
            "metric":"bugs",
            "history":Array[20]
        },
        {
            "metric":"code_smells",
            "history":Array[20]
        },
        {
            "metric":"coverage",
            "history":Array[20]
        },
        {
            "metric":"duplicated_lines_density",
            "history":Array[20]
        },
        {
            "metric":"ncloc",
            "history":Array[20]
        },
        {
            "metric":"sqale_index",
            "history":Array[20]
        },
        {
            "metric":"vulnerabilities",
            "history":Array[20]
        }
    ]
}
```

扫描历史数据：ps参数表示返回几次的数据
http://example.sonar.com/api/project_analyses/search?project=projectGroup1:projectNameExample&ps=3
```
{
    "paging":{
        "pageIndex":1,
        "pageSize":3,
        "total":20
    },
    "analyses":[
        {
            "key":"AW7LG5GFQzwg82DVBAfm",
            "date":"2019-12-03T17:33:15+0800",
            "events":[
                {
                    "key":"AW7LG5bjQzwg82DVBAfo",
                    "category":"VERSION",
                    "name":"master"
                }
            ]
        },
        {
            "key":"AW61nn_9Qzwg82DV8pVB",
            "date":"2019-11-29T13:24:42+0800",
            "events":[

            ]
        },
        {
            "key":"AW6xeRnRQzwg82DV8DFL",
            "date":"2019-11-28T18:05:22+0800",
            "events":[

            ]
        }
    ]
}
```
