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
