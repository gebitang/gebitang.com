+++
title = "STF之环境搭建可能的几个坑"
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



[link on JianShu](https://www.jianshu.com/p/82e0d75db14b)

1.  Error: listen EADDRINUSE :::8080     at Server.setupListenHandle 
这是端口被占用导致的，更换端口即可

2. fatal: unable to access 'https://github.com/AdiDahan/ng-context-menu.git/': Could not resolve proxy: localhost
或者 proxy had an error Error: getaddrinfo ENOTFOUND localhost

本地的hosts文件导致的无法解析localhost，[参考](https://github.com/angular/angular-cli/issues/2227)，使用SwitchHosts之类的软件更改hosts之后，生效的host文件需要保留以下内容——
```
127.0.0.1       localhost
255.255.255.255 broadcasthost
::1 localhost
fe80::1%lo0     localhost
```

3. 初始化设备时，提示类似 `Setup had an error Error: Service had an error: "Error: Not found; no service started."` 
原因是[STFservice 启动失败](https://github.com/openstf/stf/issues/407)，手机端清理后台程序也可能导致这个现象。需要手动重启这个服务——直接启动服务可能无效，需要先手动启动STFservice应用的主页面，对应的Service会自动启动（后续可以优化这里）[参考](https://blog.csdn.net/wpyily/article/details/53519227)

```
adb shell am start -n jp.co.cyberagent.stf/.IdentityActivity
```

4. [macbook adb cannot open interface](https://stackoverflow.com/questions/35650024/macbook-adb-cannot-open-interface)
出现类似下列错误——
```
adb E   661  9881 usb_osx.cpp:331] Could not open interface: e00002c5
adb E   661  9881 usb_osx.cpp:265] Could not find device interface
```
可能是[chrome://inspect/#devices](https://stackoverflow.com/a/36662403) 导致的问题。关闭这个页面，重启adb server

5. 如果由于依赖导致编译失败。 先确保依赖的应用都安装成功 `brew install graphicsmagick zeromq protobuf yasm pkg-config `
然后在根目录下执行 `npm rebuild `，然后再执行 `npm install`

6. `npm install`失败都话，先执行 `npm cache clean --force` ，然后重新执行这个命令

