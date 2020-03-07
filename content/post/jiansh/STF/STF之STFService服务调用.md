+++
title = "STF之STFService服务调用"
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



[link on JianShu](https://www.jianshu.com/p/380edecf102b)

[STF之环境搭建可能的几个坑](https://www.jianshu.com/p/82e0d75db14b)里提到过
>初始化设备时，提示类似 Setup had an error Error: Service had an error: "Error: Not found; no service started."

官方认为有可能是杀毒软件的问题；实测需要手动先启动一个UI页面。现在都是手动到后台进行操作的，希望可以把这个添加到业务逻辑里进行处理。

STF启动时，device模块启动是由`lib/units/device/index.js`处理。这里到依赖就包括里STFService的处理`.dependency(require('./plugins/service')`

- 这里先启动Agent——使用命令行 exec 的方式启动apk中的main方法`export CLASSPATH='/data/app/jp.co.cyberagent.stf-1.apk'; exec app_process /system/bin 'jp.co.cyberagent.stf.Agent'`中手机端启动socket服务提供：输入、点击、转屏？、唤醒？的服务（TODO）

- 然后就会启动service服务`am startservice --user 0 -a jp.co.cyberagent.stf.ACTION_START -n jp.co.cyberagent.stf/.Service`会优先使用带参数 `--user 0`方式，失败带情况下再使用不带此参数方式启动。也启动socket服务，提供包括：电池监控、连接监控、转屏、浏览器等跟身边本身有关的服务。

Agent和Service都来自STFService.apk，源码可参考[这里](https://github.com/openstf/STFService.apk)


目前都方式是：再抛出服务异常之前，主动启动异常IdentityActivity。这样下次执行设备初始化时，Service就可以正常启动了。 ——不破坏目前的业务逻辑。观察一下效果再说
