+++
title = "Appium, Android, iOS"
description = "YKYL"
tags = [
    "appium"
]
date = "2022-09-19"
topics = [
    "appium"
]
draft = true
toc=true
+++

## 环境搭建

- 完整版环境搭建 [Setup Appium on Mac OS for Android and iOS App Automation [2022 Update]](https://krishnachetan.medium.com/setup-appium-on-mac-1e06f1178427)
- 简单说明版本[Install Appium on Mac OS for iOS and setup](https://shivamtiwari2996.medium.com/install-appium-on-mac-os-for-ios-and-setup-19b2442a188)
- [Appium API](https://appium.io/docs/en/about-appium/api/)

使用现有的环境，还没有按照上面的文档进行搭建操作。

### 手动搬运Mac环境下的appium

目标机器网络环境不佳，另外一台机器上有appium 1.22.3版本的环境。

```shell
# 打包appium， 目录 /usr/local/lib/node_modules
tar -czvf appium.tar.gz appium

# 解压到目标机器对应相同目录 /usr/local/lib/node_modules
tar -zxvf appium.tar.gz -C ./

# 创建软连接启动项目 目录   /usr/local/bin 
ln -s ../lib/node_modules/appium/build/lib/main.js appium 
# 这里的坑是：记得前面是源地址，后面的是目标地址。 将后面的软连接连接到源地址上。一开始整错了，没修改；又重新连接了一回，结果就循环了 
# 检查方式：https://www.baeldung.com/linux/too-many-levels-of-symlinks
find -L appium  # Mac 下提示 find: Symlink loop resolving appium
file appium 

# 检查appium版本
appium -v 
```

### 脚本编写

- [Find Element](https://appium.io/docs/en/commands/element/find-element/)
- 安装对应的client端，例如 `pip install Appium-Python-Client`，
- 安装pytest框架 `pip install pytest`
- 使用appium inspector进行录制，转换为对应的脚本，再做对应的修改
- 执行`pytest`，详情[参考](https://docs.pytest.org/en/7.1.x/getting-started.html)

```python
# This sample code uses the Appium python client
# pip install Appium-Python-Client
# Then you can paste this into a file and simply run with Python

import os
from time import sleep

from appium import webdriver
from selenium.webdriver.common.by import By


class TestIOS:

    def setup(self):
        # https://appium.io/docs/en/drivers/ios-xcuitest/index.html 
        caps = {}
        caps["platformName"] = "ios"
        caps["automationName"] = "xcuitest"
        caps["bundleId"] = "com.ecpmobile.ecp"
        caps["deviceName"] = "xx的 iPhone"
        caps["udid"] = "d6afe671ffcf3efdadb686f296a29cb4a7452aea"
        caps["xcodeOrgId"] = "2Z7MNS5L34"
        caps["xcodeSigningId"] = "iPhone Developer"
        self.driver = webdriver.Remote("http://127.0.0.1:4723/wd/hub", caps)

    def test_real_buttons(self):
        el3 = self.driver.find_element(By.ID, "login aboutus")
        el3.click()

        el2 = self.driver.find_element(By.ID, "更多信息")
        el2.click()
        self.driver.back()
        self.driver.back()
        el5 = self.driver.find_element(By.XPATH, "//XCUIElementTypeStaticText[@name=\"忘记密码？\"]")
        el5.click()

        directory = '%s/' % os.getcwd()
        fn = 'screenshot.png'
        self.driver.get_screenshot_as_file(directory+fn)

    def teardown(self):
        sleep(3)
        self.driver.quit()
```

脚本保存到 testappium文件夹下，命名为`test.py`

### 命令行方式执行

```shell
# 创建虚拟环境
python3 -m venv testappium
# 激活当前环境
source testappium/bin/activate 
# 安装依赖
pip3 install pytest
pip3 install Appium-Python-Client
# 执行测试，确保已经有appium server启动
pytest test.py
```


### 打包WDA

- 下载项目，使用xcode打开
- WDA签名使用导入的方式，不勾选 "Automatically manage signing"，使用import方式
  - Targets中的 `WebDriverAgentRunner`必须导入签名， `WebDriverAgentLib`可以不添加证书
- 根据provisiioning profile文件中的AppId修改项目的`bundle identifier`
- 选择Product菜单栏对应的Destination和Scheme，分别设置为连接的iPhone设备和 WebDriverAgentRunner
- 执行 Product菜单栏的 Test

将自动打包安装WDA到对应的手机。详细的操作可以[参考](https://blog.csdn.net/weixin_41765699/article/details/124322812)，或者[这里](https://testerhome.com/topics/7220)

也可以采用 xcodebuild 命令行模式执行——

```shell
# 解锁keychain，以便可以正常的签名应用，PASSWORD是你自己mac电脑的开机密码
PASSWORD="你自己的开机密码" 
security unlock-keychain -p $PASSWORD ~/Library/Keychains/login.keychain

# 获取设备的UDID
UDID=$(idevice_id -l | head -n1)
xcodebuild -project WebDriverAgent.xcodeproj -scheme WebDriverAgentRunner -destination "id=$UDID" USE_PORT=8100 test 

# with 

```

#### Xcode提示“codesign 想要访问您的钥匙串中的密钥”

在 Keychain Access中选择 login.keychain中的登录选项中导入的证书，双击私钥，修改“访问控制 access controll”，选择 所有应用都可以访问。更精细的可以将 codesign应用添加到列表中（没找到如何选中 `/usr/bin/codesign`的方法）。 [图文参考](https://www.jianshu.com/p/4a58d4967b1f)

#### 安装WDA时报错 “The code signature version is no longer supported”

在iOS 15以上版本安装WDA时提示报错。因为“Apple has changed the codesign signature to include DER (Distinguished Encoding Rules)  encoded entitlements in addition to the plist encoded entitlements. ” 采用了新的编码规范，低版本Xcode默认不支持，需要进行设置。[参考](https://stackoverflow.com/a/68467307/1087122)

- Xcode: `Build Setting -> Other Code Signing Flags` 添加 `--generate-entitlement-der`
- xcodebuild: `xcodebuild OTHER_CODE_SIGN_FLAGS='--generate-entitlement-der' -project WebDriverAgent.xcodeproj -scheme WebDriverAgentRunner -destination "id=$UDID" USE_PORT=8100 test `


更直接的办法是升级Xcode版本:) 

#### Xcode 14.0打包WDA报错

遇到链接问题: ld: cannot link directly with dylib/framework, your binary is not an allowed client of /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/PrivateFrameworks/XCTAutomationSupport.framework/XCTAutomationSupport for architecture arm64

官方论坛[相关问题](https://developer.apple.com/forums/thread/712039)，appium的github描述了[原因](https://github.com/appium/appium/issues/17174)：苹果官方更严格的限制，导致XCTAutomationSupport问题，在appium2中修复了这个问题。Fastbot也出现类似的[问题](https://github.com/bytedance/Fastbot_iOS/issues/103)，建议使用低版本的进行代替——实操可行。

替换之后，遇到的问题时之前截图的命令现在提示——

```
 Encountered internal error running command: UnableToCaptureScreen: Error Domain=XCTDaemonErrorDomain Code=39 "Legacy screen requests are no longer supported" UserInfo={NSLocalizedDescription=Legacy screen requests are no longer supported}
[debug] [W3C (4754dddc)]     at errorFromW3CJsonCode (/usr/local/lib/node_modules/appium/node_modules/appium-base-driver/lib/protocol/errors.js:780:25)
[debug] [W3C (4754dddc)]     at ProxyRequestError.getActualError (/usr/local/lib/node_modules/appium/node_modules/appium-base-driver/lib/protocol/errors.js:663:14)
[debug] [W3C (4754dddc)]     at JWProxy.command (/usr/local/lib/node_modules/appium/node_modules/appium-base-driver/lib/jsonwp-proxy/proxy.js:272:19)
[debug] [W3C (4754dddc)]     at runMicrotasks (<anonymous>)
[debug] [W3C (4754dddc)]     at processTicksAndRejections (node:internal/process/task_queues:96:5)
[debug] [W3C (4754dddc)]     at XCUITestDriver.proxyCommand (/usr/local/lib/node_modules/appium/node_modules/appium-xcuitest-driver/lib/commands/proxy-helper.js:96:12)
[debug] [W3C (4754dddc)]     at getScreenshotFromWDA (/usr/local/lib/node_modules/appium/node_modules/appium-xcuitest-driver/lib/commands/screenshots.js:11:18)
[debug] [W3C (4754dddc)]     at wrapped (/usr/local/lib/node_modules/appium/node_modules/asyncbox/lib/asyncbox.js:60:13)
[debug] [W3C (4754dddc)]     at retry (/usr/local/lib/node_modules/appium/node_modules/asyncbox/lib/asyncbox.js:43:13)
[debug] [W3C (4754dddc)]     at retryInterval (/usr/local/lib/node_modules/appium/node_modules/asyncbox/lib/asyncbox.js:70:10)
[debug] [W3C (4754dddc)]     at XCUITestDriver.getScreenshot (/usr/local/lib/node_modules/appium/node_modules/appium-xcuitest-driver/lib/commands/screenshots.js:42:10)
```


### 现状总结

使用appium 1.22.3版本，打包自带的WDA工程(位于 `/usr/local/lib/node_modules/appium/node_modules/appium-webdriveragent`)，然后写自动化脚本进行验证

- Mac 12.6 + Xcode 13.4.1 + Appium 1.22.3 可以正常支持 iOS 16.0 版本，操作、截图正常
- Mac 13.0 + Xcode 14.0 + Appium 1.22.3 由于Xcode中的XCTAutomationSupport问题导致无法正常编译；使用Xcode 13.4版本的XCTAutomationSupport替换之后，正常点击操作OK，但截图提示错误


重新离线下载Xcode 13.4.1，但无法安装到Mac13.0 Ventura系统，[官方描述](https://developer.apple.com/forums/thread/707626)，即使使用命令行启动，依然会报错——

```shell
utfdeMini-148:~  /Applications/Xcode13.4.1.app/Contents/MacOS/Xcode 
Killed: 9
utfdeMini-148:~ utf$ The application cannot be opened for an unexpected reason, error=Error Domain=NSOSStatusErrorDomain Code=-10664 "kLSIncompatibleApplicationVersionErr: The app is incompatible with the current OS" UserInfo={_LSLine=4047, _LSFunction=_LSOpenStuffCallLocal}
```

[Utilize a Different Xcode Version for Build Process ](https://support.macincloud.com/support/solutions/articles/8000042681-how-to-utilize-a-different-xcode-version-for-build-process-on-mac)这里的方式……好像多尝试几次之后居然成功了？

需要先设置变量

```shell 
export DEVELOPER_DIR=/Applications/Xcode13.4.1.app/Contents/Developer # make sure set this value first， or else the Xcode cann't open. 
/Applications/Xcode13.4.1.app/Contents/MacOS/Xcode
```

效果验证
```shell 
# example
utfdeMini-148:~ utf$ export DEVELOPER_DIR=/Applications/Xcode13.4.1.app/Contents/Developer
utfdeMini-148:~ utf$ /Applications/Xcode13.4.1.app/Contents/MacOS/Xcode 
objc[38398]: Class ASVError is implemented in both /Applications/Xcode13.4.1.app/Contents/SharedFrameworks/GPUToolsCore.framework/Versions/A/GPUToolsCore (0x12c851050) and /Applications/Xcode13.4.1.app/Contents/PlugIns/GPUDebugger.ideplugin/Contents/Frameworks/GPUToolsASVC.framework/Versions/A/GPUToolsASVC (0x125c30338). One of the two will be used. Which one is undefined.
objc[38398]: Class DYShaderAnalyzerNextGPU is implemented in both /Applications/Xcode13.4.1.app/Contents/SharedFrameworks/MTLToolsShaderProfiler.framework/Versions/A/MTLToolsShaderProfiler (0x12cc7ac60) and /Applications/Xcode13.4.1.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/GPUTools/PlugIns/GLToolsShaderProfilerMobileSupport.gtplugin/Contents/MacOS/GLToolsShaderProfilerMobileSupport (0x12ce2a9c0). One of the two will be used. Which one is undefined.
2022-09-29 17:36:29.376 Xcode[38398:1616211] Looking for Simulator.app in (
    "<DVTFilePath:0x60000294bf00:'/Applications/Xcode13.4.1.app/Contents/Developer/Applications'>"
)
2022-09-29 17:36:33.295 Xcode[38398:1617460]  DVTPortal: Service '<DVTPortalViewDeveloperService: 0x600001a018c0; action='viewDeveloper'>' encountered an unexpected result code from the portal ('1100')
2022-09-29 17:36:33.295 Xcode[38398:1617460]  DVTPortal: Error:
...
```

执行`sudo xcode-select --switch /Applications/Xcode13.4.1.app`强制切换后(图标已经显示为禁止操作)。重新执行成功了(尽管命令行启动时会有报错信息)