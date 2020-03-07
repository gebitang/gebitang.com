+++
title = "Basic-Kibana"
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



[link on JianShu](https://www.jianshu.com/p/3ac48be859b9)

假装天天接触大数据的样子= =|

Basic的意思就是：了解这个东西是做什么的？如何配置安装？简单实用方法是怎样的？

有了这个概念，就可以上手操作。 

--- 

上云后，重新部署之后，对应的pod会被删除释放，但log信息存储到了ES里，所以可以从kibana上进行搜索。图形化工具可以简单完成过滤的需求。但需要连续的上下文时，就需要手写查询语句了。

这里可以看到 [Elasticsearch](https://www.elastic.co/)的[入门教程](http://www.ruanyifeng.com/blog/2017/08/elasticsearch.html) ，配套使用的是 [kibana](https://www.elastic.co/products/kibana)，kibana的简单安装教程[参考这里](https://segmentfault.com/a/1190000015107023)，这是一个[系列文章](https://segmentfault.com/a/1190000015140854)。


>[全文搜索](https://baike.baidu.com/item/%E5%85%A8%E6%96%87%E6%90%9C%E7%B4%A2%E5%BC%95%E6%93%8E)属于最常见的需求，开源的 [Elasticsearch](https://www.elastic.co/) （以下简称 Elastic）是目前全文搜索引擎的首选。

>它可以快速地储存、搜索和分析海量数据。维基百科、Stack Overflow、Github 都采用它。

下面这个操作，就可以返回连续时间片段内的200条内容。
```
POST /apps-log-2019-12-06/_search
{
  "size": 200, 
  "_source": "msg", 
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "namespace": "testtest"
          }
          
        },
        {
          "term": {
            "cluster": "dev1"
          }
        },
        {
          "term": {
            "deployname": "pod name"
          }
        },
      {
        "range": {
          "@timestamp": {
            "gte": "2019-12-06T06:30:28.626Z",
            "lte": "2019-12-06T06:38:51.710Z"
          }
        }
      }
      ]
    }
  }
}
```
