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

## appium2.0 

- [Running Appium from Source](https://appium.io/docs/en/contributing-to-appium/appium-from-source/)
- [Appium in a Nutshell](https://appium.io/docs/en/contributing-to-appium/appium-packages/index.html)，架构关系图
- 2.0依赖node 14+ 和 npm 8+ 
- 至今（2023-01-16）由于对应的文档还没准备好，[尚未正式发布](https://github.com/appium/appium/discussions/15828)
- 关键修改是将不同的包进行独立管理，启用了默认变量`$APPIUM_HOME` = `~/.appium`，将会在这里管理各种插件和驱动
  - appium driver list
  - appium plugin list 



使用nvm安装到话，会自动替换appium的软连接。 

```
➜  ~ npm install -g appium@next
➜  ~ appium -v
2.0.0-beta.52
➜  ~ which appium
/Users/geb/.nvm/versions/node/v16.17.1/bin/appium
➜  ~ appium -g /tmp/appium2.log
[Appium] Welcome to Appium v2.0.0-beta.52 (REV 9600617c52d0d2e48493424c529ac6c945d2775b)
[Appium] Appium REST http interface listener started on 0.0.0.0:4723
[Appium] No drivers have been installed in /Users/geb/.appium. Use the "appium driver" command to install the one(s) you want to use.
[Appium] No plugins have been installed. Use the "appium plugin" command to install the one(s) you want to use.

^C[Appium] Received SIGINT - shutting down
[debug] [AppiumDriver@9511] There are no active sessions for cleanup
[HTTP] Waiting until the server is closed
[HTTP] Received server close event
```

nvm管理不同的文件夹以支持不同版本的node；安装的应用连接到对应的bin目录下。

```
➜  ~ ls -l /Users/geb/.nvm/versions/node/v16.17.1/bin/
total 152400
lrwxr-xr-x@ 1 geb  staff        35 Jan 16 15:10 appium -> ../lib/node_modules/appium/index.js
lrwxr-xr-x  1 geb  staff        45 Sep 23 10:44 corepack -> ../lib/node_modules/corepack/dist/corepack.js
-rwxr-xr-x  1 geb  staff  78027072 Sep 23 10:44 node
lrwxr-xr-x  1 geb  staff        38 Sep 23 10:44 npm -> ../lib/node_modules/npm/bin/npm-cli.js
lrwxr-xr-x  1 geb  staff        38 Sep 23 10:44 npx -> ../lib/node_modules/npm/bin/npx-cli.js
```

独立安装driver和plgin，检查对应依赖是否下载完整（首次安装手动结束后，只有文件夹好像也被认为安装成功，但启动appium后会报错。卸载重新安装对应的driver即可）

```
➜  ~ appium driver install xcuitest
✔ Installing 'xcuitest' using NPM install spec 'appium-xcuitest-driver'
ℹ Driver xcuitest@4.16.9 successfully installed
- automationName: XCUITest
- platformNames: ["iOS","tvOS"]
➜  ~ appium driver install uiautomator2
✔ Installing 'uiautomator2' using NPM install spec 'appium-uiautomator2-driver'
ℹ Driver uiautomator2@2.12.2 successfully installed
- automationName: UiAutomator2
- platformNames: ["Android"]
```

### 从appium 1.x 迁移到 appium 2.x

升级须知—— 

- 有哪些不兼容的设置，参考[Breaking Changes](https://appium.github.io/appium/docs/en/2.0/guides/migrating-1-to-2/#breaking-changes)
- 针对这些不兼容的点，目前项目中是否涉及到，如果改造？ one by one 
- 除了不兼容的部分，项目中哪些功能依赖到appium到哪些功能？
- 升级后这些功能点是否依然正常？ one by one 

**Breaking Changes** 

- 默认的server base path，之前为`/wd/hub`，现在不需要了，可以使用 `appium --base-path=/wd/hub`进行设置兼容，不需要修改项目中的设置
- driver的安装和目录 

1.0 所有的driver和主程序是一起安装的，默认的安装目录在 `/path/to/appium/node_modules`，例如 `/path/to/appium/node_modules/appium-xcuitest-driver/node_modules/appium-webdriveragent`  
2.0 主程序与driver分离，可以单独指定安装所需的driver， 引入新的变量 `$APPIUM_HOME`，wda的目录在`$APPIUM_HOME/node_modules/appium-xcuitest-driver/node_modules/appium-webdriveragent`  

- driver的命令行参数——可忽略，未使用
- driver的自动化参数——可忽略，只是返回code不一致
- driver升级：之前只能随appium一起升级；现在可以独立升级，例如 `appium driver update xcuitest`
- 协议变更： 2.0依然支持旧的client端。appium api基于 [W3C WebDriver Protocol](https://www.w3.org/TR/webdriver/)，包括"JSONWP" (JSON Wire Protocol) and "MJSONWP" (Mobile JSON Wire Protocol)
- Capabilities 变更较大，但依然支持旧版本的格式（因为官方维护的client已经默认采用2.0的方式）

标准的capabilities，即官方W3C WebDriver Protocol中指定的capabilities，依然保持现有格式；其他的capabilities必须知道前缀`appium:`，例如 `appium:app`。所有不支持W3C协议的WebDriver客户端将不能创建appium session，可以以类似如下的方式做兼容

```json
{
  "platformName": "iOS",
  "browserName": "Safari",
  "appium:options": {
    "platformVersion": "14.4",
    "deviceName": "iPhone 11",
    "automationName": "XCUITest"
  }
}
```

- 不再支持的命令（待更新）包括——所有变更到driver层实现的；属于老版本但不属于 W3C Protocol 的。 ——可忽略，未使用
- 图片分析功能提取到plugin中实现 —— 确认没有使用 appium的'以图定位元素'api的话，可忽略
- 执行driver 脚本 —— 未使用，可忽略
- 不支持的参数列表？——针对配置文件的，可忽略
- 删除老的driver，本身也不再使用 ——可忽略
- 内部包重命名，例如 `appium-base-driver` --> `@appium/base-driver`
- ["WD"](https://github.com/admc/wd) javascrip 客户端不再支持，推荐使用 [WebdriverIO](https://webdriver.io/) 
- [Appium Inspector](https://github.com/appium/appium-inspector) 变更为独立的app，不再依赖 [GUI Appium Desktop](https://github.com/appium/appium-desktop)。需要注意，Desktop版本低于1.21的依赖WD 客户端，所以不兼容2.0

**新Feature** 
- 支持独立的[配置文件](https://appium.github.io/appium/docs/en/2.0/guides/config/)
- 可定制 [server 插件](https://appium.github.io/appium/docs/en/2.0/ecosystem/build-plugins/)
- 自由独立安装 driver，[定制driver](https://appium.github.io/appium/docs/en/2.0/ecosystem/build-drivers/)

## WDA 

[How To Set Up And Customize WebDriverAgent Server](https://github.com/appium/appium-xcuitest-driver/blob/master/docs/wda-custom-server.md) 入门文档。

- appium在ios端使用WDA作为后端server； wda基于XCTest framework

设置——

- 使用Xcode打开WDA工程
- 选择 WebDriverAgentRunner 项目
- 选择 设备 作为 测试目标
- 选择 `Product --> Test`

启动——可以使用多种启动方式 
- [iproxy](https://github.com/libimobiledevice/libusbmuxd#iproxy), 使用xcodebuild命令行启动，然后使用iproxy转发端口请求
- [go-ios](https://github.com/danielpaulus/go-ios) 
- [tidevice](https://github.com/alibaba/taobao-iphone-device) 进行启动，WDA已经安装的情况下 `tidevice xctest -B com.facebook.wda.WebDriverAgent.Runner`

前提是针对iOS项目已经有通用的签名证书并设置了对应的bundleId等信息。

### 证书信任 

企业级证书需要先添加到信任列表之后，才能通过WDA启动起来，否则将会超时报错。 `NoHttpResponseException: xxx failed to respond` 

- 低版本：“设置”>“通用”>“关于本机”>“证书信任设置”。在“针对根证书启用完全信任”下，开启对这个证书的信任。
- 高版本：“设置”>“通用”>“VPN与设备管理”。在“企业级App”下，开启对这个证书的信任。（不同的应用可能使用了不同的企业级证书）


### libimobiledevice 系列

[github libimobiledevice](https://github.com/libimobiledevice) A cross-platform FOSS library written in C to communicate with iOS devices natively. [official site: libimobiledevice.org](https://libimobiledevice.org/)

包含多个类库——
- [usbmuxd](https://github.com/libimobiledevice/usbmuxd/) 基础库 A socket daemon to multiplex connections from and to iOS devices. 
  - 上层库 [libimobiledevice](https://github.com/libimobiledevice/libimobiledevice) A library to communicate with services on iOS devices using native protocols. 包括多种工具与iOS设备交互，例如 idevice_id，ideviceinfo， idevicescreenshot， ideviceprovision 等
  - 底层库 [libusbmuxd](https://github.com/libimobiledevice/libusbmuxd) A client library for applications to handle usbmux protocol connections with iOS devices.  包括 iproxy，inetcat工具 

### appium-webdriveragent 项目构建

当前使用的版本[v3.16.0](https://github.com/appium/WebDriverAgent/tags?after=v4.0.0)，下载zip源码后，nodejs项目，执行`node ./Scripts/build-webdriveragent.js`——需要确保依赖的npm包已经安装，可先执行`npm install`，即使整体任务失败也没关系，打包wda的依赖安装成功即可。

业务逻辑：  
- 获取xcode版本
- 删除用户目录下的DerivedData中wda相关的编译结果 `~/Library/Developer/Xcode/DerivedData/WebDriverAgent-*` 
- 执行wda编译脚本，默认针对的模拟器构建 `/bin/bash ./Scripts/build.sh`，传递的参数 runner，sim
- 创建 `uncompressed` 和 `bundles`目录，
- 将xcode项目文件复制到此目录下，
- 并将编译生成的 `WebDriverAgent-*` 文件保存当前目录的`DerivedData/WebDriverAgent`
- 将此目录整体打包为 webdriveragent-xcode_xcodeversion.zip 文件，并保存中 bundles 目录下
- 将编译打包的`DerivedData/WebDriverAgent/Build/Products/Debug-iphonesimulator/WebDriverAgentRunner-Runner.app`压缩为zip文件 `WebDriverAgentRunner-Runner.app.zip`保存在工程主目录
- 删除uncopressed目录


## 环境搭建

- 完整版环境搭建 [Setup Appium on Mac OS for Android and iOS App Automation [2022 Update]](https://krishnachetan.medium.com/setup-appium-on-mac-1e06f1178427)
- 简单说明版本[Install Appium on Mac OS for iOS and setup](https://shivamtiwari2996.medium.com/install-appium-on-mac-os-for-ios-and-setup-19b2442a188)
- [Appium API](https://appium.io/docs/en/about-appium/api/)

使用现有的环境，还没有按照上面的文档进行搭建操作。

简单理解：appium作为web server，实现了JSONWP协议。这样客户端可以语言无关，调用符合规范的http接口即可。后端对接不同的实现，支持不同的平台，例如android、iOS、windows、Web等等。 

### 多版本支持

[How to install multiple appium versions on my Mac](https://discuss.appium.io/t/how-to-install-multiple-appium-versions-on-my-mac/33085)

本质上，appium是一个[nodejs版本的web server](https://appium.io/docs/en/contributing-to-appium/developers-overview/)，安装好依赖环境之后，只需要将appium安装到不同的目录即可—— 

```shell
#Install version 1.22.2
npm install --prefix /opt/appium1.22.2 appium@1.22.2
#Install version 1.22.3
npm install --prefix /opt/appium1.22.3 appium@1.22.3

#Start the version 1.22.2
cd /opt/appium1.20.1
./node_modules/appium/build/lib/main.js --version
1.22.2
% ./node_modules/appium/build/lib/main.js
[Appium] Welcome to Appium v1.22.2
[Appium] Appium REST http interface listener started on 0.0.0.0:4723


#Start the version 1.22.3
cd /opt/appium1.22.3
./node_modules/appium/build/lib/main.js --version
1.22.3
./node_modules/appium/build/lib/main.js -p 4724
[Appium] Welcome to Appium v1.22.3
[Appium] Non-default server args:
[Appium] port: 4724
[Appium] Appium REST http interface listener started on 0.0.0.0:4724
```


### 指定appium使用的Xcode版本

[Running Appium with multiple Xcode versions installed](https://github.com/appium/appium-xcuitest-driver/blob/master/docs/multiple-xcode-versions.md)  

如果有多个Xcode版本——

- 使用 `xcode-select` 工具 

```shell
# show current default Xcode
xcode-select --print-path
# Set default Xcode.
sudo xcode-select -s /Applications/Xcode13.app/Contents/Developer
# Run Appium (from command line or with GUI).
appium
```

- 使用环境变量
```shell
#Set DEVELOPER_DIR environment variable.
export DEVELOPER_DIR=/Applications/Xcode12.app/Contents/Developer
#Run Appium from the same shell.
appium 
```

### node环境

Mac： 下载pkg文件，命令行安装  `sudo installer -pkg /path/to/you_pkg_file.pkg -target / `

### install dmg 

```shell
# 命令行下安装 dmg
# 1. 挂在 dmg
sudo hdiutil attach <image>.dmg
# 2. 安装 pkg
sudo installer -pkg /Volumes/<image>/<image>.pkg -target /
# 3. 卸载 dmg
sudo hdituil detach <image>.dmg
```

### Permission denial

各个手机设置不同——

- 小米：在开发者选项里，把“USB调试（安全设置）"打开即可，允许USB调试修改权限或模拟点击
- oppo/一加：在开发者选项里，把"禁止权限监控"打开（在“应用”部分，最下面）

```
[UiAutomator2] Relaxing hidden api policy
[debug] [ADB] Running '/Users/geb/Library/Android/sdk/platform-tools/adb -P 5037 -s 18a281e9 shell 'settings put global hidden_api_policy_pre_p_apps 1;settings put global hidden_api_policy_p_apps 1;settings put global hidden_api_policy 1''
[debug] [UiAutomator2] Deleting UiAutomator2 session
[debug] [ADB] Running '/Users/geb/Library/Android/sdk/platform-tools/adb -P 5037 -s 18a281e9 shell am force-stop com.coloros.calculator'
[UiAutomator2] Restoring hidden api policy to the device default configuration
[debug] [ADB] Running '/Users/geb/Library/Android/sdk/platform-tools/adb -P 5037 -s 18a281e9 shell 'settings delete global hidden_api_policy_pre_p_apps;settings delete global hidden_api_policy_p_apps;settings delete global hidden_api_policy''
[debug] [BaseDriver] Event 'newSessionStarted' logged at 1665558963920 (15:16:03 GMT+0800 (China Standard Time))
[debug] [W3C] Encountered internal error running command: Error executing adbExec. Original error: 'Command '/Users/geb/Library/Android/sdk/platform-tools/adb -P 5037 -s 18a281e9 shell 'settings delete global hidden_api_policy_pre_p_apps;settings delete global hidden_api_policy_p_apps;settings delete global hidden_api_policy'' exited with code 255'; Command output: 
[debug] [W3C] Exception occurred while executing 'delete':
[debug] [W3C] java.lang.SecurityException: Permission denial: writing to settings requires:android.permission.WRITE_SECURE_SETTINGS
[debug] [W3C] 	at com.android.providers.settings.SettingsProvider.enforceWritePermission(SettingsProvider.java:2640)
[debug] [W3C] 	at com.android.providers.settings.SettingsProvider.mutateGlobalSetting(SettingsProvider.java:1632)
[debug] [W3C] 	at com.android.providers.settings.SettingsProvider.mutateGlobalSetting(SettingsProvider.java:1624)
[debug] [W3C] 	at com.android.providers.settings.SettingsProvider.deleteGlobalSetting(SettingsProvider.java:1595)
[debug] [W3C] 	at com.android.providers.settings.SettingsProvider.call(SettingsProvider.java:661)
[debug] [W3C] 	at android.content.ContentProvider.call(ContentProvider.java:2473)
[debug] [W3C] 	at android.content.ContentProvider$Transport.call(ContentProvider.java:521)
[debug] [W3C] 	at com.android.providers.settings.SettingsService$MyShellCommand.deleteForUser(SettingsService.java:408)
[debug] [W3C] 	at com.android.providers.settings.SettingsService$MyShellCommand.onCommand(SettingsService.java:282)
[debug] [W3C] 	at com.android.modules.utils.BasicShellCommandHandler.exec(BasicShellCommandHandler.java:97)
[debug] [W3C] 	at android.os.ShellCommand.exec(ShellCommand.java:38)
[debug] [W3C] 	at com.android.providers.settings.SettingsService.onShellCommand(SettingsService.java:50)
[debug] [W3C] 	at android.os.Binder.shellCommand(Binder.java:970)
[debug] [W3C] 	at android.os.Binder.onTransact(Binder.java:854)
[debug] [W3C] 	at android.os.Binder.execTransactInternal(Binder.java:1226)
[debug] [W3C] 	at android.os.Binder.execTransact(Binder.java:1163)
```

### 安装libimobiledevice 最新版本

目前brew上的最新版本为1.3.0，但不支持iOS 16.0。使用 `brew install --HEAD libimobiledevice`时，提示缺少libimobiledevice-glue-1.0

参考[issue 1217下的comment](https://github.com/libimobiledevice/libimobiledevice/issues/1217#issuecomment-1009402345)，手动创建blue的formula

- brew手动安装`libimobiledevice-glue`

```shell
#手动创建formula
brew create "https://github.com/libimobiledevice/libimobiledevice-glue.git"
#自动进入编辑模式，或手动进入，按照下面的格式编辑
brew edit libimobiledevice-glue

# 执行安装
brew install --HEAD libimobiledevice-glue
```

```ruby
 class LibimobiledeviceGlue < Formula
   desc ""
   homepage ""
   license ""
   head "https://github.com/libimobiledevice/libimobiledevice-glue.git"

   depends_on "autoconf" => :build
   depends_on "automake" => :build
   depends_on "libtool" => :build
   depends_on "pkg-config" => :build
   depends_on "libplist"

   def install
     system "./autogen.sh", "--prefix=#{prefix}"
     system "make", "install"
   end

 end
```

- 修改libimobiledevice的formula之后安装HEAD

`brew edit libimobiledevice`修改formula，增加 `depends_on "libimobiledevice-glue"`，然后执行 `brew install --HEAD libimobiledevice`


### 手动搬运Mac环境下的appium

目标机器网络环境不佳，另外一台机器上有appium 1.22.3版本的环境。

```shell
# 打包appium， 目录 /usr/local/lib/node_modules
tar -czvf appium.tar.gz appium

# 解压到目标机器对应相同目录 /usr/local/lib/node_modules
tar -zxvf appium.tar.gz -C ./

# jar 命令有类似的功能，但不能指定解压目录， 将以下war包解压到当前目录
jar -xvf test.war 

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
        caps["platformVersion"] = "16.1"
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

独立命令行启动appium `appium -g /tmp/appium.log` 确保使用 node版本为v16.17.1的长期版本，否则appium可能提示`failed to receive to receive any data within the timeout 5000` ，参考官方的[issue 16399](https://github.com/appium/appium/issues/16399)

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