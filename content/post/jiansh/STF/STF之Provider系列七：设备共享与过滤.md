+++
title = "STF之Provider系列七：设备共享与过滤"
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



[link on JianShu](https://www.jianshu.com/p/36c1dce838f0)

[STF之复用ADB一](https://www.jianshu.com/p/02a561879ef6)里满足了设备复用的场景。 

JAVA应用自己管理设备，只需要在使用时提前向STF申请占用即可。 参考这里的[API: post-userdevices](https://github.com/openstf/stf/blob/master/doc/API.md#post-userdevices) 只需要预设好足够的时间即可`{serial: 'yyyy', timeout: 900000 }`

使用完毕后，再使用 [API: delete-userdevicesserial](https://github.com/openstf/stf/blob/master/doc/API.md#delete-userdevicesserial)释放设备即可。

--- 

与之相对应的场景是，**如果独占某一个具体的设备**。比如只提供给Java应用使用，不在STF上显示出来。 思路为：指定的设备不进行初始化即可，在provider中找找（学习使用了3个月，又落下了2个月……真·三天打鱼两天晒网:(）

`lib/units/provider/index.js`中其实已经提供了一些过滤手段，只需要对应的添加一个额外的方法即可。例如——
```
// don't provider custome devices to stf. 
    function isCustomDevice(device) {
      return device.id === '5dcee012'
    }
```
添加到现有的逻辑中
```
// Helper for ignoring unwanted devices
    function filterDevice(listener) {
      return function(device) {
        if (isWeirdUnusableDevice(device)) {
          log.warn('ADB lists a weird device: "%s"', device.id)
          return false
        }

        if (isCustomDevice(device)) {
          log.warn('filter out custome devices: "%s"', device.id)
          return false
        }

        if (!options.allowRemote && isRemoteDevice(device)) {
          log.info(
            'Filtered out remote device "%s", use --allow-remote to override'
          , device.id
          )
          return false
        }
        if (options.filter && !options.filter(device)) {
          log.info('Filtered out device "%s"', device.id)
          return false
        }
        return listener(device)
      }
    }
```
这样，STF启动后，就不会再对这个设备进行初始化的动作——
```
2019-12-21 11:46:45 INF/provider 94447 [*] Subscribing to permanent channel "Z6vkYKLtRneJW2jDCd+t9A=="
2019-12-21 11:46:45 INF/provider 94447 [*] Sending output to "tcp://192.168.1.174:7116"
2019-12-21 11:46:45 INF/provider 94447 [*] Receiving input from "tcp://192.168.174:7114"
2019-12-21 11:46:45 INF/provider 94447 [*] Tracking devices
2019-12-21 11:46:45 WRN/provider 94447 [*] Filtered out remote device: "5dcee012"
```
