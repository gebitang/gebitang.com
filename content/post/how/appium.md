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
- 安装对应的client端，例如 `pip install Appium-Python-Client`，安装pytest框架 `pip install pytest`
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
