+++
title = "appium-支持多设备"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "draft"
]
toc = true
draft = true
+++


### 支持多设备，每个设备启动一个appium的session

` appium -a 127.0.0.1 -p 4727 -bp 4728  -U deviceid --session-override`
- 先安装appium库，在窗口执行appium返回appium版本就对了
- appium --help 查看帮助
- 多设备的时候，修改-p和-bp端口号就可以了，记住 --session-override 一定要加
- -U UDID, --udid UDID
