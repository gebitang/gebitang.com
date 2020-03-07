+++
title = "STF之Windows环境"
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



[link on JianShu](https://www.jianshu.com/p/519488b80c5e)

Node环境 + 依赖 rethinkdb graphicsmagick zeromq protobuf yasm pkg-config 

[rethinkdb windows/](https://rethinkdb.com/docs/install/windows/)

```
#  go to the directory that you unpacked rethinkdb.exe in
rethinkdb.exe -d c:\RethinkDB\data\
```

[graphicsmagick ](https://sourceforge.net/projects/graphicsmagick/files/graphicsmagick-binaries/1.3.33/)
正常安装完成，可执行gm命令即可。系统安装后会自动添加环境变量 `c:\program files\graphicsmagick-1.3.33-q16`


[https://zeromq.org/download/](https://zeromq.org/download/)


[protobuf releases](https://github.com/protocolbuffers/protobuf/releases)

https://github.com/protocolbuffers/protobuf/releases/download/v3.9.1/protoc-3.9.1-win64.zip


[yasm Download ](https://yasm.tortall.net/Download.html)



npm install 

```
D:\gprojects\RemoteDevice>npm rebuild

> bufferutil@1.3.0 install D:\gprojects\RemoteDevice\node_modules\bufferutil
> node-gyp rebuild


D:\gprojects\RemoteDevice\node_modules\bufferutil>if not defined npm_config_node_gyp (node "C:\Program Files\nodejs\node_modules\npm\node_modules\npm-lifecycle\node-gyp-bin\\..\..\node_modules\node-gyp\bin\node-gyp.js" rebuild )  else (node "C:\Program Files\nodejs\node_modules\npm\node_modules\node-gyp\bin\node-gyp.js" rebuild )


#   执行：https://gist.github.com/jtrefry/fd0ea70a89e2c3b7779c
npm install -g node-gyp
npm install -g socket.io
```

npm install -g bower



---

一直卡在这个问题上了：

[node-gpy到底是干嘛的](https://blog.csdn.net/qq_33826977/article/details/78620363)

[node-gyp#installation ](https://github.com/nodejs/node-gyp#installation)

[Microsoft/nodejs-guidelines ](https://github.com/Microsoft/nodejs-guidelines/blob/master/windows-environment.md#compiling-native-addon-modules)

这三个链接看起来是有关系

主要是 zeromq在windows环境下一直没跑起来——解决这个核心问题先



