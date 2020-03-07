+++
title = "STF之安装失败问题"
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



[link on JianShu](https://www.jianshu.com/p/c597e0a2aca4)

遇到实际的apk包打包问题导致在低版本上不兼容，导致安装失败。 这种情况下STF是给不出来错误提示的。 

看后台log，报错信息类似——
```
2019-08-27T12:36:04.694Z INF/storage:temp 87143 [*] Uploaded "app-major-release_test.apk" to "/Users/gebitang/projects/project-remote/tempSaveDir/upload_7241077d603d046174fbb6a38baf117c"
2019-08-27T12:36:04.934Z ERR/storage:plugins:apk 87145 [*] Unable to read manifest of "1d6c3a9d-01c8-462b-b350-7dec6e59b5ad" Error: end of central directory record signature not found
    at /Users/gebitang/projects/project-remote/RemoteDevice/node_modules/yauzl/index.js:183:14
    at /Users/gebitang/projects/project-remote/RemoteDevice/node_modules/yauzl/index.js:618:5
    at /Users/gebitang/projects/project-remote/RemoteDevice/node_modules/fd-slicer/index.js:32:7
    at FSReqWrap.wrapper [as oncomplete] (fs.js:658:17)
```

`lib/units/storage/plugins/apk`中抛出的错误信息，前端log里可以看到类似——
```
GET http://localhost:7100/s/apk/1890768c-118c-4839-afaa-b7053dfd9c40/app-major-release_test.apk/manifest 500 (Internal Server Error)
```

强制后台server报错，导致前台无法处理返回信息。在上面的后台处理逻辑中，依然让api强求返回正常的200值，提供对应的错误信息/或者是改造前端的返回结果。不直接throw error？`res/app/commponent/stf/install/install-service.js`中处理。
