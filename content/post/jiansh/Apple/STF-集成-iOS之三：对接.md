+++
title = "STF-集成-iOS之三：对接"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "Apple"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/9829abb40149)

[昨天](https://www.jianshu.com/p/cfe66350c5fa)遇到的问题 `Error: Unknown security handler: accessTokenAuth `是因为启动过程中有类似下面的提示——

No YAML parser loaded.  Suggest adding js-yaml dependency to your package.json file.
WARNING: No configurations found in configuration directory:/Users/gebitang/projects/project-remote/RemoteDevice/lib/units/api/config
WARNING: To disable this warning set SUPPRESS_NO_CONFIG_WARNING in the environment.

没有yaml的解析文件，自然就无法使用yaml中定义的安全module了。不清楚原理是什么。 使用yarn重新install之后OK了。

---

关于swagger的yaml文件，这里有对应的资料——
- [示例说明](https://scotch.io/tutorials/speed-up-your-restful-api-development-in-node-js-with-swagger) 
- 关于默认的配置文件内容：[swagger-node configuration.md](https://github.com/swagger-api/swagger-node/blob/master/docs/configuration.md)
- [官方文档：swagger-node: Swagger module for node.js [http://swagger.io](http://swagger.io) ](https://github.com/swagger-api/swagger-node/blob/master/docs/README.md)

--- 

新问题是只移植后台部分，前端暂时无法显示屏幕内容——需要看看对应的屏幕展示逻辑是怎样的：需要看ios-device下的stream.js是如何使用到的

- `res/app/components/stf/control/control-service.js`新增加了 `screendump`方法发送zmq消息通知，
- `lib/units/device/plugins/screen/dump.js`，接收到消息后完成截图动作。
- `res/app/components/stf/screen/screen-directive.js` 增加图像质量控制
