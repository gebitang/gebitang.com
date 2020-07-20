+++
title = "STF"
description = "Everything about STF"
tags = [
]
date = "2020-04-27"
topics = [
    "stf",
    "tech"
]
toc = true
+++

STF的核心功能可以理解为：“同步图像” + “点击”。前者使用minicap完成，后者依赖minitouch。

虽然目前依然是3.4.1版本，但由于增加了很多定制化的内容，所以从——

>commit: 16e64b51ff3dbb5cf7492fa773c123ceb12454b0   
>Merge pull request #770 from neofreko/adb-key-api Implements addAdbPublicKey API endpoint  
> Simo Kinnunen on 7/11/2019, 2:44:32 AM  

之后就没同步最新代码。

手机自动被升级到了Android 10，手动同步了minicap，但发现minitouch已经不被支持了——

### For Android 10 and up

[minitouch about Android 10](https://github.com/openstf/minitouch#for-android-10-and-up)

Minitouch can't handle Android 10 by default, due to a new security policy. The workaround is to forward touch commands to STFService. If you are using minicap standalone (without STF), you need to take care of running the service and agent, before running minicap. Instructions on how to do that can be found [here](https://github.com/openstf/STFService.apk#running-the-service).

```
 minitouch says: "Unable to open device /dev/input/event0 for inspectionopen: Permission denied"
```

解决[方案](https://github.com/openstf/minitouch/issues/49#issuecomment-582946443)：

Minitouch cannot support Android 10 due to a new security policy. To overcome that a fallback has been added to forward touch events to STFService.apk. Latest changes in STFService.apk aim at managing those events at the framework level.
Right now if you are using minitouch (outside of openstf scope), you simply have to:

*   [run the STFService agent](https://github.com/openstf/STFService.apk#running-the-agent)
*   [then run minitouch](https://github.com/openstf/minitouch#running)


### 本地编译场景错误

人肉同步合并一下最新代码试试。

保留了yarn.lock文件和package.json文件，需要先更新一下包。

最终是可以正常使用了，遇到的错误基本都是之前就遇到过的。所以三天不练手生。

犯错列表：  

- node版本使用错误（本地环境默认使用的为v12.13.1的环境）
- 在没有更新yarn.lock的情况下使用yarn进行打包
- 编译后找不到zmq服务（npm install过程包含`node-gyp rebuild`过程）需要进行`npm rebuild`
- 没有启动rethinkdb就开始启动`bin/stf local` 

合并代码之后，可以正常镜像远程控制（图像同步，点击操作），但`stf local`场景下的用户登录有点问题（使用ldap登录似乎又是正常的），关键是屏幕截图却不行了。一直报错`connect ECONNREFUSED`。

[Error: connect ECONNREFUSED 127.0.0.1:8087](https://blog.csdn.net/u010411264/article/details/53899599)

```
npm config set proxy null
```

从这个文章获得启发，我本地环境使用了VPN，terminal里做了export的动作，尽管VPN已经关闭，但转发依然可能生效。重启了一个terminal环境，果然正常了。

### 定制化错误一

在做代码合并的时候就遇到了类似下面的问题。上线后使用的LDAP方式，似乎没有问题。就没管怎么回事。后来发现线上也挂了。

最终是因为：添加了两个同名方法，但将旧版本的方法放到了新版本后面。导致新用户登录后创建的纪录对于必选字段`privilege`没有设置值。

所以会报错`Error: Missing at least one required field for Message .UserField: .UserField.privilege` 

登录后，在认证逻辑里`lib/units/app/middleware/auth.js`里调用 `lib/db/api.js`中的`saveUserAfterLogin`方法。这个方法后来将`createUser`方法抽象出来。

```
/home/stf/RemoteDeviceNew/node_modules/bluebird/js/main/async.js:43
        fn = function () { throw arg; };
                           ^

Error: Illegal value for Message.Field .UserChangeMessage.user: .UserField (Error: Missing at least one required field for Message .UserField: .UserField.privilege)
    at Field.ProtoBuf.Reflect.FieldPrototype.encode (/home/stf/RemoteDeviceNew/node_modules/protobufjs/dist/ProtoBuf.js:2811:27)
    at Message.ProtoBuf.Reflect.MessagePrototype.encode (/home/stf/RemoteDeviceNew/node_modules/protobufjs/dist/ProtoBuf.js:2342:31)
    at MessagePrototype.encode (/home/stf/RemoteDeviceNew/node_modules/protobufjs/dist/ProtoBuf.js:1981:31)
    at Object.envelope (/home/stf/RemoteDeviceNew/lib/wire/util.js:31:53)
    at sendUserChange (/home/stf/RemoteDeviceNew/lib/units/groups-engine/watchers/users.js:19:16)
    at /home/stf/RemoteDeviceNew/lib/units/groups-engine/watchers/users.js:50:9
    at /home/stf/RemoteDeviceNew/node_modules/rethinkdb/cursor.js:261:20
    at tryCatcher (/home/stf/RemoteDeviceNew/node_modules/bluebird/js/main/util.js:26:23)
    at Promise.successAdapter (/home/stf/RemoteDeviceNew/node_modules/bluebird/js/main/nodeify.js:23:30)
    at Promise._settlePromiseAt (/home/stf/RemoteDeviceNew/node_modules/bluebird/js/main/promise.js:582:21)
    at Promise._settlePromises (/home/stf/RemoteDeviceNew/node_modules/bluebird/js/main/promise.js:700:14)
    at Async._drainQueue (/home/stf/RemoteDeviceNew/node_modules/bluebird/js/main/async.js:123:16)
    at Async._drainQueues (/home/stf/RemoteDeviceNew/node_modules/bluebird/js/main/async.js:133:10)
    at Immediate.Async.drainQueues [as _onImmediate] (/home/stf/RemoteDeviceNew/node_modules/bluebird/js/main/async.js:15:14)
    at runCallback (timers.js:810:20)
    at tryOnImmediate (timers.js:768:5)
2020-5-12 15:40:16 FTL/cli:local 19678 [*] Child process had an error ExitError: Exit code "1"
    at ChildProcess.<anonymous> (/home/stf/RemoteDeviceNew/lib/util/procutil.js:49:23)
    at emitTwo (events.js:126:13)
    at ChildProcess.emit (events.js:214:7)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:198:12)
2020-5-12 15:40:16 INF/cli:local 19678 [*] Shutting down all child processes
```

前面还是有惧怕心理，遇到问题，直接google最直接的错误信息——反而找不到对应的信息，因为这本来就是`其他人更大概率不会遇到的问题`——却没有分析代码逻辑。

直接读代码是更直观的方式。上次mixin安装时的[electron问题](https://www.jianshu.com/p/cb83dd80454f)也是类似情况。

### 定制化错误二

上传本地apk进行安装时，进行到50%之后就报错——导致系统退出。


```
2020-7-20 15:38:02 INF/storage:temp 17950 [*] Uploaded "67a219fbe4ce8e2ab4c1fc5ce60195d3" to "/home/stf/tempSaveDir/upload_a0cfd8e997b752e14aecb2202cb0c1c4"
2020-7-20 15:38:02 INF/websocket 17944 [*] installMsg manifest  undefined
/home/stf/RemoteDeviceNew/node_modules/protobufjs/dist/ProtoBuf.js:2641
                    throw Error("Illegal value for "+this.toString(true)+" of type "+this.type.name+": "+val+" ("+msg+")");
                    ^

Error: Illegal value for Message.Field .InstallMessage.manifest of type string: undefined (not a string)
    at Field.<anonymous> (/home/stf/RemoteDeviceNew/node_modules/protobufjs/dist/ProtoBuf.js:2641:27)
    at Field.ProtoBuf.Reflect.FieldPrototype.verifyValue (/home/stf/RemoteDeviceNew/node_modules/protobufjs/dist/ProtoBuf.js:2721:29)
    at MessagePrototype.set (/home/stf/RemoteDeviceNew/node_modules/protobufjs/dist/ProtoBuf.js:1799:63)
    at new Message (/home/stf/RemoteDeviceNew/node_modules/protobufjs/dist/ProtoBuf.js:1728:42)
    at Socket.<anonymous> (/home/stf/RemoteDeviceNew/lib/units/websocket/index.js:830:15)
    at emitThree (events.js:136:13)
    at Socket.emit (events.js:217:7)
    at /home/stf/RemoteDeviceNew/node_modules/socket.io/lib/socket.js:528:12
    at _combinedTickCallback (internal/process/next_tick.js:132:7)
    at process._tickCallback (internal/process/next_tick.js:181:9)
2020-7-20 15:38:02 FTL/cli:local 17862 [*] Child process had an error ExitError: Exit code "1"
    at ChildProcess.<anonymous> (/home/stf/RemoteDeviceNew/lib/util/procutil.js:49:23)
    at emitTwo (events.js:126:13)
    at ChildProcess.emit (events.js:214:7)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:198:12)
```

重新加载为数不多的前端知识，先搜索对应的关键字，在对应的位置增加log信息。

`lib/units/websocket/index.js`的830行报错，同时提示`.InstallMessage.manifest of type string: undefined`，对应的定义在`lib/wire/wire.proto`。可以推测出没有获取到apk的manifest信息。

```javascript
// lib/units/websocket/index.js
.on('device.install', function(channel, responseChannel, data) {
          joinChannel(responseChannel)
          push.send([
            channel
          , wireutil.transaction(
              responseChannel
            , new wire.InstallMessage(
                data.href
              , data.launch === true
              , JSON.stringify(data.manifest)
              )
            )
          ])
        })

// wire.proto 
message InstallMessage {
  required string href = 1;
  required bool launch = 2;
  optional string manifest = 3;
}
```

拿到原始的apk分析后，是正常的文件，可以正常安装。 

前端处理上传文件并进行安装的服务为`res/app/components/stf/install/install-service.js`。进度到50%时，实际上已经将文件上传成功了，然后开始进行安装的动作。通过文件后缀来区分apk和ipa的区别(这块是定制的内容)。看起来是像是通过http强求获取manifest文件内容的?这块先忽略。

看本地上传文件的处理` lib/units/storage/temp.js` 注意到log内容包含` Uploaded "67a219fbe4ce8e2ab4c1fc5ce60195d3" to "/home/stf/tempSaveDir/upload_a0cfd8e997b752e14aecb2202cb0c1c4"` 似乎文件名做了处理。

跟踪了一下这个temp.js文件的git历史记录，看起来是同步代码时增加了一部分处理文件的操作——
```javascript
form.on('fileBegin', function(name, file) {
      var md5 = crypto.createHash('md5')
      file.name = md5.update(file.name).digest('hex')
    })
```
将这段代码删除后，恢复正常。

### `openstf/stf` to `DeviceFarmer/stf`

顺手查一下STF的最新代码，发现这个仓库已经存档。

[openstf/stf](https://github.com/openstf/stf)存档，新fork到[DeviceFarmer](https://github.com/DeviceFarmer)组织下的[DeviceFarmer/stf](https://github.com/DeviceFarmer/stf)继续维护开发。

两边分支的最后同步commitID为5月19日的`632d80fbe3520ee58af4fa0f1aefdc9d6db21dbf`： [Merge pull request openstf #1229 from openstf/stf-service-apk](https://github.com/DeviceFarmer/stf/commit/632d80fbe3520ee58af4fa0f1aefdc9d6db21dbf)

后面openstf上进行了一下文档的更新说明，代码相关的更新都在新仓库进行了。


但最新代码[`lib/units/storage/temp.js#L93`](https://github.com/DeviceFarmer/stf/blob/master/lib/units/storage/temp.js#L93)的确定义了对文件的处理逻辑。
```javascript
form.on('fileBegin', function(name, file) {
      var md5 = crypto.createHash('md5')
      file.name = md5.update(file.name).digest('hex')
    })
```
[#filebegin](https://www.npmjs.com/package/formidable#filebegin)  
 
>Emitted whenever a new file is detected in the upload stream. Use this event if you want to stream the file to somewhere else while buffering the upload on the file system.

这样导致只要上传文件就会走这一个逻辑。那最新的代码是如何确保文件可以正常获取到manifest文件的呢？
