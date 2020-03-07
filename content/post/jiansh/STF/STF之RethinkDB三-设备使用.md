+++
title = "STF之RethinkDB三-设备使用"
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



[link on JianShu](https://www.jianshu.com/p/538e516643aa)

[STF之RethinkDB二](https://www.jianshu.com/p/fac393d47fb2)
获取到设备使用、停止使用的切入口之后，甚至不需要了解设备使用的业务逻辑，就可以直接进行数据库操作，完成**记录设备使用时长**的功能。

### 前端逻辑
入口app.js，开始加载各个模块，除了核心模块`ngRoute, ngTouch, gettext, angular-hotkeys`之外，还有其他到相对目录下到模块，如`./layout, ./device-list, ./control-panes`，最后一个 `stf/standalone`属于components目录下到对应模块（webpack.config.json中指定）

其中下一级到`control-panes/index.js`中又定义了再下一级到引用。其中`ControlPanesController`中传递到参数都是来自这里都引用。
```
module.exports =
  function ControlPanesController($scope, $http, gettext, $routeParams,
    $timeout, $location, DeviceService, GroupService, ControlService,
    StorageService, FatalMessageService, SettingsService) {
```
比如`DeviceService`就来自`stf/device/index.js`中定义的`.factory('DeviceService', require('./device-service'))` 使用socket.io的websocket协议进行设备添加、删除、变化的监听

### 设备使用

前台跳转到设备页面`#!/control/serial`之后，`res/app/control-panes/control-panes-controller.js`的ControlPanesController会调用`getDevice($routeParams.serial)`方法。

查找到设备后，会使用websocket发送group.invite消息——
```
DeviceService.get(serial, $scope)
        .then(function(device) {
          return GroupService.invite(device)
        })
```

收到消息后使用zeromq消息通知对应的provider设备被占用
```
# lib/units/websocket/index.js
.on('group.invite', function(channel, responseChannel, data) {
          joinChannel(responseChannel)
          push.send([
            channel
          , wireutil.transaction(
              responseChannel
            , new wire.GroupMessage(
                new wire.OwnerMessage(
                  user.email
                , user.name
                , user.group
                )
              , data.timeout || null
              , wireutil.toDeviceRequirements(data.requirements)
              )
            )
          ])
        })
```

收到`wire.GroupMessage`后，会进行查找匹配——
```
#lib/units/device/plugins/group.js
router
      .on(wire.GroupMessage, function(channel, message) {
        var reply = wireutil.reply(options.serial)
        grouputil.match(ident, message.requirements)
          .then(function() {
            return plugin.join(message.owner, message.timeout, message.usage)
          })
          .then(function() {
            push.send([
              channel
            , reply.okay()
            ])
          })
          .catch(grouputil.RequirementMismatchError, function(err) {
            push.send([
              channel
            , reply.fail(err.message)
            ])
          })
          .catch(grouputil.AlreadyGroupedError, function(err) {
            push.send([
              channel
            , reply.fail(err.message)
            ])
          })
      })
```

log输出里可以看到使用设备后，会有类似下面但log
```
2019-08-13T02:51:55.662Z IMP/device:plugins:group 25893 [3967b3c2] Now owned by "xxx@xxx.com"
2019-08-13T02:51:55.663Z INF/device:plugins:group 25893 [3967b3c2] Subscribing to group channel "jUKrIsV5SmCRtSaSRsjvBQ=="
```
然后再次发送`new wire.JoinGroupMessage` 消息——
```
push.send([
            wireutil.global
          , wireutil.envelope(new wire.JoinGroupMessage(
              options.serial
            , currentGroup
            , usage
            ))
          ])
```
服务端的`processor`收到消息，更新数据库。——这样就到了我们到入口

对应的还有`wire.LeaveGroupMessage`消息

### 数据库操作
有了切入口，后续只剩下数据库本身的操作——
- 初始化时，新增 usageRecord表
- 开始使用JoinGroupMessage时只devices表，新增起始时间、使用者字段
- 结束使用LeaveGroupMessage时更新devices，新增结束时间
在usageRecord表上插入记录： 最后一步操作成功之后，获取devices表上对应设备，

```
dbapi.unsetDeviceOwner(message.serial, new Date().getTime())
  .then(function(){
    return dbapi.getDeviceBySerial(message.serial)
  })
  .then(function(device){
    let duration = Math.floor((device.timestampEnd - device.timestampStart)/1000)
    dbapi.saveUsageRecord(message.serial, device.lastUser, device.timestampStart, device.timestampEnd, duration)
  })
```
