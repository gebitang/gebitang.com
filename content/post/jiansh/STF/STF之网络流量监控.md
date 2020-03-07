+++
title = "STF之网络流量监控"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "STF"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/c6f1a4e35299)

STF部署在内网环境上，办公网络访问时时快时慢。需要定位一下到底是什么原因。

使用ping周期性访问服务器地址，记录显示的确网络不稳定——

```
#!/bin/bash

saveName=pingstf.log
startTime=`date +%Y%m%d-%H:%M:%S`
startTime_s=`date +%s`

echo $startTime >>$saveName
#sleep 5
ping -c 300 10.112.78.174 >> $saveName

endTime=`date +%Y%m%d-%H:%M:%S`
endTime_s=`date +%s`
sumTime=$[ $endTime_s - $startTime_s ]

echo "$startTime ---> $endTime total:"  $sumTime " seconds" >> $saveName

echo -e "-----------\n" >>$saveName
```

结果显示——
```
106:round-trip min/avg/max/stddev = 3.526/19.663/141.330/24.808 ms
214:round-trip min/avg/max/stddev = 3.501/34.131/315.329/50.720 ms
233:round-trip min/avg/max/stddev = 3.226/11.238/33.710/8.526 ms
260:round-trip min/avg/max/stddev = 5.176/19.594/120.324/26.470 ms
470:round-trip min/avg/max/stddev = 3.538/23.756/879.931/66.947 ms
780:round-trip min/avg/max/stddev = 2.929/19.074/183.116/22.933 ms
1090:round-trip min/avg/max/stddev = 2.077/26.112/369.143/46.236 ms
1103:round-trip min/avg/max/stddev = 14.699/44.024/90.027/32.936 ms
1719:round-trip min/avg/max/stddev = 3.019/13.239/173.212/15.396 ms
2028:round-trip min/avg/max/stddev = 2.411/15.965/215.530/28.650 ms
➜  ~
```

[iftop slurm bmon for bandwidth monitoring on ubuntu](https://www.debugpoint.com/2016/10/3-best-command-line-tool-network-bandwidth-monitoring-ubuntu-linux/) 
[iftop](http://www.ex-parrot.com/~pdw/iftop/), [slurm](https://github.com/mattthias/slurm), [bmon](https://github.com/tgraf/bmon)

```
# iftop 
sudo apt install iftop
sudo iftop -i eth0

# slurm
sudo apt install slurm
slurm -z -i wlp3s0

# (a.k.a. Bandwidth Monitor)
sudo apt install bmon
bmon eth0
```

服务器端使用`iftop`监控网卡流量 `eno1`，结果显示服务器端可以提供最高20mb每秒的上行流量（it部分反馈说最大可以到100mb）
```
# install first
sudo apt install iftop 
# monitor network 
sudo iftop -i eno1 
```

本地Mac环境可以使用  `nettop`[watch-network-traffic-mac-os-x-nettop](http://osxdaily.com/2013/06/07/watch-network-traffic-mac-os-x-nettop/)
查看特定进程，如firefox（PID:34696）到网络流量情况`nettop -p 34696`。

也可以通过自带的 `活动监控器`下的 `网络`，过滤要监控的应用如 firefox，查看网络流量情况。

结论：在网络情况良好的情况下，播放视频也很流畅；但网络不佳时，点击操作的响应都很慢，延时到5秒以上。

后续：尝试降低图片压缩大小（minicap应该支持压缩比），降低网络消耗。
