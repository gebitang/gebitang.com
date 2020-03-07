+++
title = "Appium-Basic"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "Appium"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/7df1dc23c086)

官网过一遍，有个大概：[概述](https://appium.io/docs/en/about-appium/intro/?lang=en)、[历史](https://appium.io/history.html?lang=en)、[getting started](https://appium.io/docs/en/about-appium/getting-started/index.html)、[API](https://appium.io/docs/en/about-appium/api/)、[appium desktop](https://github.com/appium/appium-desktop/releases/latest)

先聊八卦，小伙有[前途](https://www.linkedin.com/in/dacuellar)：


### 理念
[Introduction to Appium](https://appium.io/docs/en/about-appium/intro/?lang=en) 概述appium主张以及如何实现——

1. You shouldn't have to recompile your app or modify it in any way in order to automate it.
2. You shouldn't be locked into a specific language or framework to write and run your tests.
3. A mobile automation framework shouldn't reinvent the wheel when it comes to automation APIs.
4. A mobile automation framework should be open source, in spirit and practice as well as in name!

*   iOS 9.3 and above: Apple's [XCUITest](https://developer.apple.com/reference/xctest)
*   iOS 9.3 and lower: Apple's [UIAutomation](https://web.archive.org/web/20160904214108/https://developer.apple.com/library/ios/documentation/DeveloperTools/Reference/UIAutomationRef/)
*   Android 4.2+: Google's [UiAutomator/UiAutomator2](https://developer.android.com/training/testing/ui-automator)
*   Android 2.3+: Google's [Instrumentation](http://developer.android.com/reference/android/app/Instrumentation.html). (Instrumentation support is provided by bundling a separate project, [Selendroid](http://selendroid.io/))
*   Windows: Microsoft's [WinAppDriver](http://github.com/microsoft/winappdriver)

Driver Doc：

*   The [XCUITest Driver](https://appium.io/docs/en/drivers/ios-xcuitest/index.html) (for iOS and tvOS apps)
*   The [UiAutomator2 Driver](https://appium.io/docs/en/drivers/android-uiautomator2/index.html) (for Android apps)
*   The [Windows Driver](https://appium.io/docs/en/drivers/windows/index.html) (for Windows Desktop apps)
*   The [Mac Driver](https://appium.io/docs/en/drivers/mac/index.html) (for Mac Desktop apps)
*   (BETA) The [Espresso Driver](https://appium.io/docs/en/drivers/android-espresso/index.html) (for Android apps)


### 核心概念：

- Client/Server Architecture： C/S架构
- Session：建立会话后使用sessionID进行后续的命令传递
- [Desired Capabilities](https://appium.io/docs/en/writing-running-appium/caps/index.html)：key-value的map结构进行环境定义
- [Appium Server](https://github.com/appium/appium)：NodeJS实现的server端
- [Appium Clients](https://appium.io/docs/en/about-appium/appium-clients/index.html)：各个语言的client端

### 协议
- [selenium webdriver api](https://www.seleniumhq.org/docs/03_webdriver.jsp) selinium端的应用
- [webdriver ](https://w3c.github.io/webdriver/)  known as the JSON Wire Protocol， 上面的成为事实标准，推动成为W3C协议草案
- [Mobile JSON Wire Protocol Specification](https://github.com/SeleniumHQ/mobile-spec/blob/master/spec-draft.md) 移动端的JSON Wire Protocol

>WebDriver is a remote control interface that enables introspection and control of user agents. It provides a platform- and language-neutral wire protocol as a way for out-of-process programs to remotely instruct the behavior of web browsers.


