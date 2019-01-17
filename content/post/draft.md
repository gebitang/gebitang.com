+++
title = "GRIT"
description = "perseverance and passion for long-term goals"
tags = [
    "draft",
    "grit"
]
date = "2017-11-07"
topics = [
    "draft",
    "grit"
]
draft = true
+++


```
DDD				debug标识
deviceUI		加载本地设备、更新设备信息
runningDevices	获取到的远程任务
RTT				准备好的设备
XCRun			任务开始、提交	
USF				更新设备信息
```

提高效率的尝试：
一月～三月，驻场短信服务平台搭建，现场讨论实现
五月～六月，web版本大工位验证；iTestinPlus联动方式；扩大单工位支持32台设备；星辰大海模式——websocket
八月，星辰大海模式重构http方式

OCR上线支持
视频记录功能
自建数据统计功能
防锁屏设计
账号信息、动态获取账号信息
自动化干预支持
同步真机功能、自动化功能
海外工位支持

OCR微服务、证书代理、数据统计正式使用
apk合并上线

公司文化“团队、目标、专业”：
一群具有专业能力素质的人组建出一个专业的团队为了同一个目标努力奋斗。 

价值观：以客户为中心，为客户创作价值
服务好、质量好、运营成本低，优先满足客户需求，提升产品质量和市场竞争力。
遵循“服务创造价值”的理念，以专业的服务、快速的响应，为客户提供全面的服务方案，竭尽所能协助客户改善产品质量和市场竞争力、降低客户成本。 
让应用更有价值


业务线：
征途客户端全面上线重构版本：提高设备利用率
自动化脚本格式化重构
远程真机的流媒体方式图像传输

技术线：
Go Deeper, Not Wilder.
产品化思维：征途、云途
Z-Y-X：星途

全自动能否实现：征途；征途操作人员+数据标记
反馈闭环如何建立：问题解决、验证数据
应该需要怎样的能力：维护多年项目之技术栈；Go Deeper, Not Wilder.



[maven in five minutes](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)

[What is H.264?](http://www.streamingmedia.com/Articles/Editorial/What-Is-.../What-is-H.264-74735.aspx)
[H.264 wiki](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC)
[Streaming H264 video from PiCamera to a JavaFX ImageView](https://codereview.stackexchange.com/questions/163042/streaming-h264-video-from-picamera-to-a-javafx-imageview)


trigger：
当日数据大小：`du -h --max-depth=1` 得到适配信息，查询对应的步骤报错；定位到具体的步骤信息

```
-- 报错排序
SELECT COUNT('stepIndex') AS step, stepIndex, adaptId, errCode
	, errMsg
FROM `dumpResult`
WHERE adaptId = 'tt99378b7c92ec7b3d1a416735574a3b'
GROUP BY stepIndex
ORDER BY step DESC

-- 详情
SELECT FROM_UNIXTIME(createTime / 1000), adaptId, taskId
    , imei, stepIndex, errCode, errMsg, infoPath
    , imgPath, logPath
FROM `dumpResult`
WHERE adaptId = 'tt26758b7c92ec7b1ff75166f76004ac'
    AND stepIndex = 18;

-- 当日完成适配数
SELECT FROM_UNIXTIME(startTime / 1000)
    , FROM_UNIXTIME(endTime / 1000), consumeTime, id, type
    , name, resultCode, resultMsg, xmlStatisticSteps, actualStatisticSteps
    , interveneSteps, adaptId, taskId, imei
    , manualSteps, monitorTimes
FROM `taskInterveneData`
WHERE id > 49500 and  startTime > unix_timestamp('2018-11-22 00:00:00') * 1000
	AND endTime < unix_timestamp('2018-11-23 00:00:00') * 1000 group by adaptId 

-- 具体适配结果
SELECT FROM_UNIXTIME(startTime / 1000)
    , FROM_UNIXTIME(endTime / 1000), consumeTime, id, type
    , name, resultCode, resultMsg, xmlStatisticSteps, actualStatisticSteps
    , interveneSteps, adaptId, taskId, imei
    , manualSteps, monitorTimes
FROM `taskInterveneData`
WHERE adaptId = 'tt49978b7c92ec7b29426166ece517b0';
```
