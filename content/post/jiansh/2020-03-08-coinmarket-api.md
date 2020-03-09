+++
title = "服务预警"
description = ""
tags = [
    "jiansh"
]
date = "2020-03-08"
topics = [
    "jiansh",
    "BB"
]
toc = true
+++

听直播，才想起来看下自动化更新得内容，然后发现内容没更新成功。

- 需要有自己跟踪内容得自觉（或者更新成功后有个提醒）
- 如果更新异常也发送告警（邮件告警正常，但查看邮件的频率不高）

以上两个都可以通过webhook服务完成“消息提醒”。

--- 

CoinMarketCap Developers API  

更新失败都原因是因为使用都API发生了变动。`https://api.coinmarketcap.com/v1/ticker/Tether/?convert=CNY`这个API已经过期，提示——
```
curl https://api.coinmarketcap.com/v1/ticker/Tether/?convert=CNY 
{"statusCode": 410,"error": "Gone","message": "WARNING: This API is now offline. Please switch to the new CoinMarketCap API. (https://pro.coinmarketcap.com/migrate/)"}
```
CoinMarketCap的API服务进行了升级，现在免费调用也必须先申请个账号，通过API KEY才能调用对应的接口。类似这样——
[https://coinmarketcap.com/api/documentation/v1/#operation/getV1CryptocurrencyInfo](https://coinmarketcap.com/api/documentation/v1/#operation/getV1CryptocurrencyInfo)


```
curl -H "X-CMC_PRO_API_KEY: b54bcf4d-1bca-4e8e-9a24-22ff2c3d462c" -H "Accept: application/json" -d "start=1&limit=1&convert=USD,BTC" -G https://pro-api.coinmarketcap.com/v1/cryptocurrency/listings/latest
```


