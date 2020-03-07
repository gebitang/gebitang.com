+++
title = "Lealone-快速入门"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "CPO"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/54e816217e5f)

[用户文档 快速入门](https://github.com/lealone/Lealone-Docs/blob/master/%E5%BA%94%E7%94%A8%E6%96%87%E6%A1%A3/%E7%94%A8%E6%88%B7%E6%96%87%E6%A1%A3.md)

安装这个说明进行编译。

下载源码之后，执行 `mvn clean package assembly:assembly -Dmaven.test.skip=true` 


遇到的几个小问题：

- 1. ~~Maven编译需要暂时去掉 `<module>lealone-test</module>`模块，否则会出现测试脚本失败的情况~~
这是因为我在Powershell环境下执行` mvn clean package assembly:assembly -Dmaven.test.skip=true`这个目录导致的。[参考](https://stackoverflow.com/a/6351739/1087122)这里，可以看到实际上跳过test的命令并没有生效。需要使用`mvn clean package assembly:assembly '-Dmaven.test.skip=true'`的方式执行才会生效。

效果还是很直接的——

![server 启动](https://upload-images.jianshu.io/upload_images/3296949-969f1fe012b279e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![命令行执行](https://upload-images.jianshu.io/upload_images/3296949-47e28bebab8e205e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![java main方法执行](https://upload-images.jianshu.io/upload_images/3296949-f4cc8a6aa7732925.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 2. 在linux下执行时注意Java -cp 指定classpath时才有`:`分割path的路径。同时需要设置环境变量`JAVA_HOME`。

手残，修改环境变量时，最后少打了一个H，`export PATH=${JAVA_HOME}/bin:$PATH`变成了`export PATH=${JAVA_HOME}/bin:$PAT`导致所以的命令无法使用。

正常的解决方案：

> 使用绝对路径进行操作；`/usr/bin/vi /home/geb/.zshrc`进行编辑  
>临时导出新的PATH路径`export PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin`然后继续操作

