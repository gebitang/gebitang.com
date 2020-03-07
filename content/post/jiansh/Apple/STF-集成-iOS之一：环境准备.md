+++
title = "STF-集成-iOS之一：环境准备"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "Apple"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/8cf6bdb5a091)

根据这个帖子[STF 集成 iOS 之 开源了](https://testerhome.com/topics/19548)

目前但效果只是能发现设备，还无法正常完成初始化。 

```
Setup had an error TypeError: Cannot read property 'sdk' of null
    at Object.getDeviceInfo (/Users/gebitang/projects/tryout/stf/lib/units/ios-device/support/deviceinfo.js:56:28)
    at solve (/Users/gebitang/projects/tryout/stf/lib/units/ios-device/plugins/util/identity.js:13:33)
    at /Users/gebitang/projects/tryout/stf/lib/units/ios-device/plugins/util/identity.js:18:12
    at SerialSyrup.ParallelSyrup.invoke (/Users/gebitang/projects/tryout/stf/node_modules/stf-syrup/lib/parallel.js:54:24)
    at /Users/gebitang/projects/tryout/stf/node_modules/stf-syrup/lib/serial.js:43:33
    at tryCatch1 (/Users/gebitang/projects/tryout/stf/node_modules/stf-syrup/node_modules/bluebird/js/main/util.js:63:19)
    at Promise$_callHandler [as _callHandler] (/Users/gebitang/projects/tryout/stf/node_modules/stf-syrup/node_modules/bluebird/js/main/promise.js:708:13)
    at Promise$_settlePromiseFromHandler [as _settlePromiseFromHandler] (/Users/gebitang/projects/tryout/stf/node_modules/stf-syrup/node_modules/bluebird/js/main/promise.js:724:18)
    at Promise$_settlePromiseAt [as _settlePromiseAt] (/Users/gebitang/projects/tryout/stf/node_modules/stf-syrup/node_modules/bluebird/js/main/promise.js:896:14)
    at Promise$_fulfillPromises [as _fulfillPromises] (/Users/gebitang/projects/tryout/stf/node_modules/stf-syrup/node_modules/bluebird/js/main/promise.js:1041:14)
    at Async$_consumeFunctionBuffer [as _consumeFunctionBuffer] (/Users/gebitang/projects/tryout/stf/node_modules/stf-syrup/node_modules/bluebird/js/main/async.js:74:12)
    at Async$consumeFunctionBuffer (/Users/gebitang/projects/tryout/stf/node_modules/stf-syrup/node_modules/bluebird/js/main/async.js:37:14)
    at _combinedTickCallback (internal/process/next_tick.js:132:7)
    at process._tickCallback (internal/process/next_tick.js:181:9)
```

遇到的问题都是摸黑解决：  

[https://github.com/mrx1203/stf.git](https://github.com/mrx1203/stf.git)
[https://github.com/mrx1203/WebDriverAgent.git](https://github.com/mrx1203/WebDriverAgent.git)


WDA的环境搭建参考[为iOS真机安装WebDriverAgent](https://www.jianshu.com/p/4cceb043107b)

问题：在Xcode 10.3上编译会报错：
```
Undefined symbols for architecture x86_64:
  "_OBJC_CLASS_$_XCEventGenerator", referenced from:
      objc-class-ref in FBCustomCommands.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

根据[test failed on Xcode 10.2](https://github.com/facebookarchive/WebDriverAgent/issues/1093#issuecomment-481623523)这个建议，先使用9.4版本的XCTest.framework替换后，编译通过，但启动会报错——
```
2019-10-12 11:36:37.335636+0800 WebDriverAgentRunner-Runner[3948:11746943] libMobileGestalt MobileGestalt.c:890: MGIsDeviceOneOfType is not supported on this platform.
2019-10-12 11:36:37.885546+0800 WebDriverAgentRunner-Runner[3948:11746943] Running tests...
dyld: lazy symbol binding failed: Symbol not found: __XCTLoadTestConfiguration

```
需要使用10.1版本但Xcode对应但XCTest.framework才可以。

辅助根据问题：`could not connect to lockdownd, error code xx`只有idevice_id 可以使用。可能是libimobiledevice版本问题。

[Error: Cannot find module 'eslint-config-appium' while running ./Scripts/bootstrap.sh](https://github.com/appium/eslint-config-appium/issues/11)

[could not connect to lockdownd, error code xx](https://github.com/flutter/flutter/issues/22595)

 更换设备后，选择WebDriverAgentRunner，进行自动签名。

```
#此方案可行
brew update
brew uninstall --ignore-dependencies libimobiledevice
brew uninstall --ignore-dependencies usbmuxd
brew install --HEAD usbmuxd
brew unlink usbmuxd
brew link usbmuxd
brew install --HEAD libimobiledevice
```
目前还是一团浆糊的脑子，折腾得有点懵。

```sh
#环境准备
#  安装libimobiledevice等依赖工具，如果已经安装过，可能需要升级，先卸载，再安装最新版本
    brew uninstall --ignore-dependencies libimobiledevice
    brew uninstall --ignore-dependencies usbmuxd
    brew install --HEAD usbmuxd
    brew unlink usbmuxd
    brew link usbmuxd
    brew install --HEAD libimobiledevice
    brew install --HEAD ideviceinstaller
    brew install carthage
    brew install socat
```

[libimobiledevice 常用功能](https://www.jianshu.com/p/6423610d3293)

---

brew install  usbmuxd
brew link usbmuxd
brew install libimobiledevice
brew install ideviceinstaller
brew install carthage
brew install socat

### usbmuxd 
usbmuxd: stable 1.0.10 (bottled), HEAD
USB multiplexor daemon for iPhone and iPod Touch devices
https://www.libimobiledevice.org/

### libimobiledevice 
libimobiledevice: stable 1.2.0 (bottled), HEAD
Library to communicate with iOS devices natively
https://www.libimobiledevice.org/

### ideviceinstaller
ideviceinstaller: stable 1.1.0 (bottled), HEAD
Cross-platform library for communicating with iOS devices
https://www.libimobiledevice.org/

### carthage
carthage: stable 0.33.0 (bottled), HEAD
Decentralized dependency manager for Cocoa
https://github.com/Carthage/Carthage

### socat
socat: stable 1.7.3.3 (bottled)
netcat on steroids
http://www.dest-unreach.org/socat/




