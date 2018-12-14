+++
title = "DAILY GRIT"
description = "perseverance and passion for long-term goals"
tags = [
    "daily",
    "grit"
]
date = "2018-11-22"
topics = [
    "daily",
    "grit"
]
draft = true
+++


```
	****************************************************
	***                                              ***
	***  Plan – Focus – Motivate – Measure – Reward  ***
	***                                              ***
	****************************************************
```

locat输出重定向
需要插入的内容通过 insertLogcat 接口通知到手机端 

手机号发送到手机端

[maven in five minutes](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)

[What is H.264?](http://www.streamingmedia.com/Articles/Editorial/What-Is-.../What-is-H.264-74735.aspx)
[H.264 wiki](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC)
[Streaming H264 video from PiCamera to a JavaFX ImageView](https://codereview.stackexchange.com/questions/163042/streaming-h264-video-from-picamera-to-a-javafx-imageview)

base:
com.meeruu.sharegoods

1. 保存main logcat到指定目录——一直占用线程，直到结束(测试结束，或主动结束) 返回结果
Message startAndRedirectMainLog(String adbPath, String logSavePath);

避免异常结束重复执行的时候，相同的位置还有log文件
clearMainLog() 

2. 启动eventlog监控
startEventLog()


使用eventLog

https://www.zhihu.com/question/34606201


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
