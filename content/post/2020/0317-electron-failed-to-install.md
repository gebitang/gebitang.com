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

### 8.3.0版本

哈哈，我这个应该叫做跟着Mixin学Node乜。今天升级到0.9.6版本后启动遇到点小问题。`node-gyp`和`electron`安装问题

- yarn执行安装失败。`node-gyp`的安装应该需要翻墙。使用代理后的CMD窗口后编译正常。

用来进行交叉编译的？可以进行全局安装一下？之前搭建STF环境时也遇到过这个问题。

>[`node-gyp`](https://github.com/nodejs/node-gyp) is a cross-platform command-line tool written in Node.js for compiling native addon modules for Node.js. It contains a fork of the [gyp](https://gyp.gsrc.io/) project that was previously used by the Chromium team, extended to support the development of Node.js native addons.
>
>Note that `node-gyp` is not used to build Node.js itself.

```
 > node-gyp rebuild --release                                                                                                                                                
                                                                                                                                                                             
                                                                                                                                                                             
 D:\openSources\mixin\desktop-app\node_modules\better-sqlite3>if not defined npm_config_node_gyp (node "D:\tools\nvmm\v12.11.1\node_modules\npm\node_modules\npm-lifecycle\no
yp-bin\\..\..\node_modules\node-gyp\bin\node-gyp.js" rebuild --release )  else (node "D:\tools\nvmm\v12.11.1\node_modules\npm\node_modules\node-gyp\bin\node-gyp.js" rebuild 
lease )                                                                                                                                                                      
 info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.                                                                                        
                                                                                                                                                                             
                 errorOut=prebuild-install WARN install No prebuilt binaries found (target=8.3.0 runtime=electron arch=x64 libc= platform=win32)                             
 gyp WARN install got an error, rolling back install                                                                                                                         
 gyp ERR! configure error                                                                                                                                                    
 gyp ERR! stack Error: read ECONNRESET                                                                                                                                       
 gyp ERR! stack     at TLSWrap.onStreamRead (internal/stream_base_commons.js:201:27)                                                                                         
 gyp ERR! System Windows_NT 10.0.18363                                                                                                                                       
 gyp ERR! command "C:\\Program Files\\nodejs\\node.exe" "D:\\tools\\nvmm\\v12.11.1\\node_modules\\npm\\node_modules\\node-gyp\\bin\\node-gyp.js" "rebuild" "--release"       
 gyp ERR! cwd D:\openSources\mixin\desktop-app\node_modules\better-sqlite3                                                                                                   
 gyp ERR! node -v v12.11.1                                                                                                                                                   
 gyp ERR! node-gyp -v v5.0.3                                                                                                                                                 
 gyp ERR! not ok                                                                                                                                                             
 npm ERR! code ELIFECYCLE                                                                                                                                                    
 npm ERR! errno 1                                                                                                                                                            
 npm ERR! better-sqlite3@7.0.1 build-release: `node-gyp rebuild --release`                                                                                                   
 npm ERR! Exit status 1                                                                                                                                                      
 npm ERR!                                                                                                                                                                    
 npm ERR! Failed at the better-sqlite3@7.0.1 build-release script.                                                                                                           
 npm ERR! This is probably not a problem with npm. There is likely additional logging output above.                                                                          
                                                                                                                                                                             
```
- 启动再次遇到Electron问题

应该是yarn重新编译时将我之前手动安装的8.2.5版本清理了。最新版本根据`yarn.lock`文件看升级到了`8.3.0`版本。对应的判断逻辑也做了优化。

轻车熟路，手动下载`8.3.0`版本，解压到electron模块的`dist`文件夹，新建`path.txt`文件，内容写入`electron.exe`。启动成功。

Node看起来会默认自动检查dist目录。第一次写入`dist\electron.exe`的内容，启动报错`UnhandledPromiseRejectionWarning spawn D:\openSources\mixin\desktop-app\node_modules\electron\dist\dist\electron.exe`


```
D:\openSources\mixin\desktop-app\node_modules\electron\index.js:14
    throw new Error('Electron failed to install correctly, please delete node_modules/electron and try installing again')
    ^
...
```

--- 

### 8.2.5版本

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

---

[electron 官方 #20731 ](https://github.com/electron/electron/issues/20731)  

根据yarn.lock中定义的`electron@^v8.2.5`从[这里](https://github.com/electron/electron/releases/tag/v8.2.5)进行[下载](https://github.com/electron/electron/releases/download/v8.2.5/electron-v8.2.5-win32-x64.zip)(vpn需要使用直连模式，转发模式无法下载)后，解压到 `node_modules\electron`目录下，并且手动增加`path.txt`文件。写入绝对路径。

electron可以正常启动。项目也正常编译，但运行失败了——

```
-  Bundling main process...
No type errors found
Version: typescript 3.7.5
Time: 6574ms
 DONE  Compiled successfully in 2077ms17:46:29

  File                      Size                     Gzipped

  dist_electron\index.js    1424.07 KiB              277.96 KiB

  Images and other types of assets omitted.

 INFO  Launching Electron...
�ļ�����Ŀ¼��������﷨����ȷ��
(node:30396) UnhandledPromiseRejectionWarning: Error: spawn D:\openSources\mixin\desktop-app\node_modules\electron\dist\D:\openSources\mixin\desktop-app\node_modules\electron\electron.exe ENOENT
    at notFoundError (D:\openSources\mixin\desktop-app\node_modules\vue-cli-plugin-electron-builder\node_modules\cross-spawn\lib\enoent.js:6:26)
    at verifyENOENT (D:\openSources\mixin\desktop-app\node_modules\vue-cli-plugin-electron-builder\node_modules\cross-spawn\lib\enoent.js:40:16)
    at ChildProcess.cp.emit (D:\openSources\mixin\desktop-app\node_modules\vue-cli-plugin-electron-builder\node_modules\cross-spawn\lib\enoent.js:27:25)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:272:12)
(node:30396) UnhandledPromiseRejectionWarning: Unhandled promise rejection. This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). (rejection id: 1)
(node:30396) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
```

看起来是在对应的目录没找到electron.exe，再手动帮助它一波：新建dist目录，将下载的zip文件解压到dist目录下。

启动成功:)

```
  App running at:
  - Local:   http://localhost:8080/
  - Network: http://192.168.1.106:8080/

  Note that the development build is not optimized.
  To create a production build, run yarn build.

\  Bundling main process...

 DONE  Compiled successfully in 2116ms                                                                                                                                   18:16:43

  File                      Size                     Gzipped

  dist_electron\index.js    1424.07 KiB              277.96 KiB

  Images and other types of assets omitted.

 INFO  Launching Electron...
(electron) The default value of app.allowRendererProcessReuse is deprecated, it is currently "false".  It will change to be "true" in Electron 9.  For more information please check https://github.com/electron/electron/issues/18397
```