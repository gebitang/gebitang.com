+++
title = "Electron安装失败"
description = "Electron failed to install correctly"
tags = [
    "jiansh"
]
date = "2020-03-17"
topics = [
    "jiansh",
    "Front-End"
]
toc = true
+++

Electron failed to install correctly, please delete node_modules/electron and try installing again

安装了多次，总是提示上面的错误。在Mac环境下是正常的。因为懒惰&恐惧的原因，一直没有**正面强攻**解决问题。尝试重新安装，清除仓库从头安装，都无疾而终。

[这里有针对window环境的解决](https://github.com/electron/electron/issues/20994)

- Create a file named path.txt in node_modules\electron folder.
- Write electron.exe in it.
- Download electron package manualy.
- Unpackage the electron files into node_modules\electron\dist
- Run your start script

看看源码+对比Mac环境下的内容：

首先是缺少`path.txt`文件。这样至少可以试试上面的解决方案。

```
    throw new Error('Electron failed to install correctly, please delete node_modules/electron and try installing again')
    ^

Error: Electron failed to install correctly, please delete node_modules/electron and try installing again
    at getElectronPath (D:\openSources\mixin\desktop-app\node_modules\electron\index.js:14:11)
    at Object.<anonymous> (D:\openSources\mixin\desktop-app\node_modules\electron\index.js:18:18)
    at Module._compile (internal/modules/cjs/loader.js:945:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:962:10)
    at Module.load (internal/modules/cjs/loader.js:798:32)
    at Function.Module._load (internal/modules/cjs/loader.js:711:12)
    at Module.require (internal/modules/cjs/loader.js:838:19)
    at require (internal/modules/cjs/helpers.js:74:18)
    at Object.<anonymous> (D:\openSources\mixin\desktop-app\node_modules\vue-cli-plugin-electron-builder\lib\testWithSpectron.js:2:22)
    at Module._compile (internal/modules/cjs/loader.js:945:30)
error Command failed with exit code 1.
```

`npm --registry https://registry.npm.taobao.org install electron`手动安装electron时依然报错—— 

```
> node install.js

Downloading electron-v7.1.14-win32-x64.zip: [====================================================================================================] 100% ETA: 0.0 seconds
(node:13260) UnhandledPromiseRejectionWarning: RequestError: read ECONNRESET
    at ClientRequest.<anonymous> (D:\openSources\mixin\desktop-app\node_modules\got\source\request-as-event-emitter.js:178:14)
    at Object.onceWrapper (events.js:300:26)
    at ClientRequest.emit (events.js:215:7)
    at ClientRequest.origin.emit (D:\openSources\mixin\desktop-app\node_modules\@szmarczak\http-timer\source\index.js:37:11)
    at TLSSocket.socketErrorListener (_http_client.js:406:9)
    at TLSSocket.emit (events.js:210:5)
    at emitErrorNT (internal/streams/destroy.js:91:8)
    at emitErrorAndCloseNT (internal/streams/destroy.js:59:3)
    at processTicksAndRejections (internal/process/task_queues.js:80:21)
(node:13260) UnhandledPromiseRejectionWarning: Unhandled promise rejection. This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). (rejection id: 1)
(node:13260) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
```

这样就正式到[github electron release](https://github.com/electron/electron/releases/)下载对应的版本[`electron-v7.1.14-win32-x64.zip`](https://github.com/electron/electron/releases/download/v7.1.14/electron-v7.1.14-win32-x64.zip)

可以完成electorn的安装。从github的官方issue里也可以看到这种情况是因为在安装过程中出现hang现象（手动强制暂停后，又重新进行安装）

>The error message actually occurs when you cancel the hanging install and nevertheless try to run the project.

靠谱，没报错，但也没启动起来应用:(

```
Failed to fetch extension, trying 4 more times
Failed to fetch extension, trying 3 more times
Failed to fetch extension, trying 2 more times
Failed to fetch extension, trying 1 more times
Failed to fetch extension, trying 0 more times
Vue Devtools failed to install: Error: net::ERR_CONNECTION_RESET

```