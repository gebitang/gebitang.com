+++
title = "STF之Provider系列四"
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



[link on JianShu](https://www.jianshu.com/p/a3a1740dff18)

## 环境准备 

在一台新的机器上从头搭建STF的provider环境。 

- 需要adb服务
- 可以执行 stf provider服务模块

新机器上的Node环境为V12 版本，没有adb环境。 折腾到最后还是跑起来了。记录一下遇到的可能问题：

adb是独立的可执行文件。 可共享使用，只要确保可以在任意目录下执行类似`adb devices`的命令即可。 使用软连接的方式进行操作：
```
#假设adb保存在/Users/gebitang/stfwork/adb/adb下
# 进行软连接
ln -s /Users/gebitang/stfwork/adb/adb /urs/local/bin/adb
``` 

经验证，stf使用的node环境需要为node 8版本。 否则一些依赖模块可能会安装失败。如 `jpeg-turbo`依赖的`node-pre-gyp`模块在高版本下可能出现兼容问题。 

如果提示类似 zmq模块找不到，需要进行 `npm rebuild`的操作

出现`enoent error no such file or directory rename`的错误提示，需要将 package-lock.json文件删除，重新进行 `npm install`操作

另外，下载 **[phantomjs](https://github.com/Medium/phantomjs)**可能比较慢，需要有点耐心。 

总结一下：
1. adb进行软连接
2. 确保使用node8的版本
3. 进行必要的 rebuild或 删除 packag-lock.json的操作

[about node version](https://nodesource.com/blog/understanding-how-node-js-release-lines-work/)使用[nvm进行多版本node的管理](https://segmentfault.com/a/1190000010252661)

如果新建terminal提示`N/A: version "N/A -> N/A" is not yet installed.`
是因为[没有设置](https://stackoverflow.com/a/52305468/1087122)默认的node版本，可执行类似如下脚步——
```
# N/A: version "N/A -> N/A" is not yet installed.
#设置default的node版本
nvm use node 
nvm alias default node 
```

