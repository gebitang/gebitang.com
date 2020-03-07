+++
title = "STF之Provider系列二"
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



[link on JianShu](https://www.jianshu.com/p/0d624601b8af)

[EventEmitter](https://www.runoob.com/nodejs/nodejs-event.html)
[node.js 如何使用Promise](https://www.jianshu.com/p/745e0760cff5)

[系列一](https://www.jianshu.com/p/8984575174b4)后续，现学现卖，逻辑梳理如下——

```
// Wait for others to acknowledge the device
var register = new Promise(function(resolve) {
        // Tell others we found a device
        push.send([
          wireutil.global
        , wireutil.envelope(new wire.DeviceIntroductionMessage(
            device.id
          , wireutil.toDeviceStatus(device.type)
          , new wire.ProviderMessage(
              solo
            , options.name
            )
          ))
        ])

        //只有触发 register事件（只触发一次）后，才会调用resolve--》相当于后面的then方法
        privateTracker.once('register', resolve)
      })
```

后面的zmp消息收到后会触发'register'的传递，下面的1、2、3步之后，触发上面的register事件，调用resolve

```
# 3
function deviceListener(type, updatedDevice) {
        // Okay, this is a bit unnecessary but it allows us to get rid of an
        // ugly switch statement and return to the original style.
        privateTracker.emit(type, updatedDevice)
      }
# 2
flippedTracker.on(device.id, deviceListener)

# 1
sub.on('message', wirerouter()
      .on(wire.DeviceRegisteredMessage, function(channel, message) {
        log.info('to emit "%s" on channel: "%s"', message.serial, channel)
        flippedTracker.emit(message.serial, 'register')
      })
      .handler())
```
进入“设备注册流程”
```
register.then(function() {
        log.info('Registered device "%s"', device.id)
        check()
      })
```

所以如果一直无法进入到register流程，需要先判断zmp消息通讯是否正常？

这说明没能收到zmq的消息信息，利用[这个例子](https://rastating.github.io/using-zeromq-with-node-js/)验证通讯可以正常建立，说明提供的路径有问题。

```
#默认的端口为
.option('bind-dev-pub', {
      describe: 'The address to bind the device-side ZeroMQ PUB endpoint to.'
    , type: 'string'
    , default: 'tcp://127.0.0.1:7114'
    })
    .option('bind-dev-pull', {
      describe: 'The address to bind the device-side ZeroMQ PULL endpoint to.'
    , type: 'string'
    , default: 'tcp://127.0.0.1:7116'
    })

#需要需要这启动stf local时进行修改，运行对外提供服务
bind_dev_pub="tcp://0.0.0.0:7114"
bind_dev_pull="tcp://0.0.0.0:7116"
--bind-dev-pull $bind_dev_pull --bind-dev-pub $bind_dev_pub
```

这样，本地的 provider终于可以提供服务了。

下一个问题：
进行远程连接时，提示 `Firefox can’t establish a connection to the server at ws://test.stf.com:7416/.` 

只能看到，还不能使用- -||
