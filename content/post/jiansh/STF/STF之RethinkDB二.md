+++
title = "STF之RethinkDB二"
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



[link on JianShu](https://www.jianshu.com/p/fac393d47fb2)

[STF之RethinkDB一](https://www.jianshu.com/p/917faff25782)结束了数据库rethinkdb的基本使用。

梳理一下STF项目中对于数据库到底是如何使用的。——基于目前的理解，错误的地方，欢迎指出。

>Note that you must have RethinkDB running first. 

STF项目强调启动之前必须先启动rethinkdb，是因为在执行`stf  local`时，首先会执行数据库的建库建表操作。`db.setup()` 

建立db连接之后，将连接信息传递到`lib.db.setup.js`，开始执行`建立数据库-->建立数据表-->建立索引`的流程，如果已经存在，打印log并返回。

下一步需要做的需求：记录设备使用时长——包含设备、使用人、使用开始时间、使用结束时间。

…… 

[STF之RethinkDB三](https://www.jianshu.com/p/538e516643aa)——
1. 设备使用逻辑梳理
2. 记录设备使用时长
