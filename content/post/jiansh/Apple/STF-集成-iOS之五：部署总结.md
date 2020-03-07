+++
title = "STF-集成-iOS之五：部署总结"
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



[link on JianShu](https://www.jianshu.com/p/35b390b2c409)

在测试旧的MacPro机器上搭建正式的部署环境。 

Mac系统为13.3 + Xcode9.4。 根据测试环境搭建的经验，选择Xcode10.1 

###系统环境
苹果的开发者网站可以正常访问，但下载Xcode时却需要翻墙，文件过大[Xcode 10.1](https://download.developer.apple.com/Developer_Tools/Xcode_10.1/Xcode_10.1.xip)有5.6GB，chrome下载过程中，多次出现下载失败情况，resume后无法续传。网上的旧方案为使用wget+localCookie，但目前有人反馈也不好使。

- 最终方案为：使用Safari进行下载，Safari下载Xcode支持续传功能。

- Mac系统需要升级到10.14 Mojave才可以使用Xcode10.1

这基本就折腾了一天。

###iOS-STF环境

好在环境比较干净，执行下面的安装比较正常。依赖环境安装——
```
    brew install --HEAD usbmuxd
    brew unlink usbmuxd
    brew link usbmuxd
    brew install --HEAD libimobiledevice
    brew install --HEAD ideviceinstaller
    brew install carthage
    brew install socat
```

brew安装时，会检查Xcode版本，可能问题——

1. 需要执行`xcdoe-select --install`安装对应的Develop Command tools；
2. 提示Xcode过时。

>Error: Your Xcode (10.1) is too outdated. Please update to Xcode 10.2.1 (or delete it). Xcode can be updated from the App Store.

[Xcode outdated](http://gebitang.com/post/how/apple-dev/#brew-%E5%AE%89%E8%A3%85%E6%8F%90%E7%A4%BAxcode-outdated)问题：编辑brew对应的配置脚本，删除对xcode版本的检查。

3. Xcod10.1默认不支持12.3.1系统的iOS文件，需要找个Xcode10.3的版本，复制对应版本的支持文件： `/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport/xxversion`

###WDA环境
- clone项目[https://github.com/mrx1203/stf.git](https://github.com/mrx1203/stf.git)到本地，执行`./Scripts/bootstrap.sh`进行编译安装。
- 使用Xcode打开此项目，登录Apple ID，将几个对应工程（lib、runner、test...）下的签名方案选择为自动签名（目前Xcode都可以自动识别账户+设备添加）
- 项目根目录下执行`xcodebuild -project WebDriverAgent.xcodeproj -scheme WebDriverAgentRunner -destination 'id=ed042146799cb2d009e4a05d9215f5f30f446d98' test` 进行，详细过程可参考[这里](https://www.jianshu.com/p/4cceb043107b)

###源码环境
集成[https://github.com/mrx1203/stf.git](https://github.com/mrx1203/stf.git)到现有的STF工程下，使用现有的前端规范

package更新了以下npm包：

```shell
npm install --save android-device-list@^1.1.85 
npm install --save bplist@0.0.4
npm install --save images@^3.0.2
npm install --save plist@^3.0.1
npm install --save request-promise@^4.2.4
npm install --save sharp@^0.22.0
npm install --save sleep@^6.0.0
npm install --save teen_process@^1.14.1
npm install --save unzip2@^0.2.5
npm install --save xml2js@^0.4.19
npm install --save xml2map@^1.0.2
npm install --save xutil@1
```

后端部分——

- 拷贝lib/cli下的ios-device,ios-provider,local三个文件夹，local文件夹可以直接覆盖（新增参数+ios-provider线程）
- 拷贝lib/cli下的index.js文件，直接覆盖（只是增加了新的依赖）
- 拷贝lib/units下的ios-device,ios-provider文件夹；覆盖lib/units/storage/plugins/apk/task/manifest.js文件（支持ipa类型）
- 拷贝lib/util下的ipareader.js和download.js文件；ipa文件解析和下载
- 拷贝res/app/components/stf/install/install-service.js文件；ipa文件的安装
- 拷贝res/app/components/stf/storage/storage-service.js文件；上传ipa文件
- 拷贝lib/wire/wire.proto文件；加了消息

前端部分——

- `res/app/components/stf/control/control-service.js`新增加了 `screendump`方法发送zmq消息通知，
- `lib/units/device/plugins/screen/dump.js`，接收到消息后完成截图动作。

---

正常情况下，完成上面的动作之后，执行`npm install`，然后执行 `stf local  --wda-path /Users/gebitang/projects/tryout/WebDriverAgent/ --wda-port 8100` 之后就可以识别iOS设备了

以provider方式启动——

`
bin/stf ios-provider --no-cleanup --name $localIp.local --min-port 7400 --max-port 7700 --connect-sub tcp://$serverip:7114 --connect-push tcp://$serverip:7116 --group-timeout 10000 --public-ip $localIp --storage-url http://$serverip:$serverport --wda-path /Users/gebitang/projects/tryout/WebDriverAgent/ --wda-port 8100  --vnc-initial-size 600x800 --mute-master never --allow-remote  
`
