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

https://yumichan.net/video-processing/video-compression/introduction-to-h264-nal-unit/

https://yumichan.net/video-processing/video-compression/free-download-h264-mp4-file-format-standard-specification/

ITU-T: Infrastructure of audiovisual services – Coding of moving video: Advanced video coding for generic audiovisual services

https://www.itu.int/rec/T-REC-H.264/en

ISO: Information technology — Coding of audio-visual objects — Part 10: Advanced Video Coding

https://www.itu.int/rec/T-REC-H.264/en

[What is H.264?](http://www.streamingmedia.com/Articles/Editorial/What-Is-.../What-is-H.264-74735.aspx)
[H.264 wiki](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC)
[Streaming H264 video from PiCamera to a JavaFX ImageView](https://codereview.stackexchange.com/questions/163042/streaming-h264-video-from-picamera-to-a-javafx-imageview)


H.264 / MPEG-4 Part 10, Advanced Video Coding (MPEG-4 AVC) is a common video compression format developed by ITU-T Video Coding Experts Group (VCEG) and ISO/IEC JTC1 Moving Picture Experts Group (MPEG). Network Abstraction Layer (NAL) and Video Coding Layer (VCL) are the two main concepts in H.264. A H.264 file consists of a number of NAL units (NALU) and each NALU can be classified as VCL or non-VCL. Video data is processed by the codec and packed into NAL units.


https://phoboslab.org/log/2013/09/html5-live-video-streaming-via-websockets
HTML5 LIVE VIDEO STREAMING VIA WEBSOCKETS

https://stackoverflow.com/questions/52376060/how-to-use-ffmpeg-in-javascript-to-decode-h-264-frames-into-rgb-frames

https://ek21.com/news/1/23223/ 



[一个广院工科生的视音频技术笔记-雷霄骅(leixiaohua1020)的专栏](https://blog.csdn.net/leixiaohua1020)

[[总结]FFMPEG视音频编解码零基础学习方法](https://blog.csdn.net/leixiaohua1020/article/details/15811977)

[[总结]视音频编解码技术零基础学习方法](https://blog.csdn.net/leixiaohua1020/article/details/18893769)

[最简单的基于FFMPEG+SDL的视频播放器 ver2 （采用SDL2.0）](https://blog.csdn.net/leixiaohua1020/article/details/38868499)

[《基于 FFmpeg + SDL 的视频播放器的制作》课程的视频](https://blog.csdn.net/leixiaohua1020/article/details/47068015)

-----------------


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
