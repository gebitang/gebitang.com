+++
title = "Appium-实战(一)"
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



[link on JianShu](https://www.jianshu.com/p/006d050e3ab9)

[npm、cnpm和nvm](https://www.jianshu.com/p/acb33f7ea6b4)

STF目前使用的是8.16版本的Node环境，需要使用多版本Node。

```
# 查看长期稳定版本
nvm ls-remote --lts
...
v10.17.0   (Latest LTS: Dubnium)
v12.13.0   (Latest LTS: Erbium)

#安装node 12版本
nvm install v12.13.0
```

### 淘宝镜像安装appium server

ZH：[https://npm.taobao.org/](https://npm.taobao.org/) EN：[https://cnpmjs.org/](https://cnpmjs.org/)

安装cnpm：`npm install -g cnpm --registry=https://registry.npm.taobao.org` 
安装appium server：`cnpm install -g appium`
检查版本：`appium -v` 

### 环境检查

安装appium-doctor，android方面的环境需要：

必选项—— ANDROID_HOME、JAVA_HOME、adb、android、emulator 
可选性——可忽略

```
#windows
info AppiumDoctor Appium Doctor v.1.12.1
info AppiumDoctor ### Diagnostic for necessary dependencies starting ###
info AppiumDoctor  ✔ The Node.js binary was found at: C:\Program Files\nodejs\node.EXE
info AppiumDoctor  ✔ Node version is 12.11.1
info AppiumDoctor  ✔ ANDROID_HOME is set to: C:\Users\joechin\AppData\Local\Android\Sdk
info AppiumDoctor  ✔ JAVA_HOME is set to: C:\Program Files\Java\jdk1.8.0_181
info AppiumDoctor  ✔ adb exists at: C:\Users\joechin\AppData\Local\Android\Sdk\platform-tools\adb.exe
info AppiumDoctor  ✔ android exists at: C:\Users\joechin\AppData\Local\Android\Sdk\tools\android.bat
info AppiumDoctor  ✔ emulator exists at: C:\Users\joechin\AppData\Local\Android\Sdk\tools\emulator.exe
info AppiumDoctor  ✔ Bin directory of %JAVA_HOME% is set
info AppiumDoctor ### Diagnostic for necessary dependencies completed, no fix needed. ###
info AppiumDoctor
info AppiumDoctor ### Diagnostic for optional dependencies starting ###
WARN AppiumDoctor  ✖ opencv4nodejs cannot be found.
WARN AppiumDoctor  ✖ ffmpeg cannot be found
WARN AppiumDoctor  ✖ mjpeg-consumer cannot be found.
WARN AppiumDoctor  ✖ bundletool.jar cannot be found
info AppiumDoctor ### Diagnostic for optional dependencies completed, 4 fixes possible. ###
info AppiumDoctor
info AppiumDoctor ### Optional Manual Fixes ###
info AppiumDoctor The configuration can install optionally. Please do the following manually:
WARN AppiumDoctor  ➜ Why opencv4nodejs is needed and how to install it: https://github.com/appium/appium/blob/master/docs/en/writing-running-appium/image-comparison.md
WARN AppiumDoctor  ➜ ffmpeg is needed to record screen features. Please read https://www.ffmpeg.org/ to install it
WARN AppiumDoctor  ➜ mjpeg-consumer module is required to use MJPEG-over-HTTP features. Please install it with 'npm i -g mjpeg-consumer'.
WARN AppiumDoctor  ➜ bundletool.jar is used to handle Android App Bundle. Please read http://appium.io/docs/en/writing-running-appium/android/android-appbundle/ to install it. Also consider adding the ".jar" extension into your PATHEXT environment variable in order to fix the problem for Windows
info AppiumDoctor
info AppiumDoctor ###
info AppiumDoctor
info AppiumDoctor Bye! Run appium-doctor again when all manual fixes have been applied!
info AppiumDoctor

# linux
info AppiumDoctor Appium Doctor v.1.12.1
info AppiumDoctor ### Diagnostic for necessary dependencies starting ###
info AppiumDoctor  ✔ The Node.js binary was found at: /home/stf/.nvm/versions/node/v12.13.0/bin/node
info AppiumDoctor  ✔ Node version is 12.13.0
info AppiumDoctor  ✔ ANDROID_HOME is set to: /home/stf/tools/androidtools
info AppiumDoctor  ✔ JAVA_HOME is set to: /usr/lib/jvm/java-8-openjdk-amd64
info AppiumDoctor  ✔ adb exists at: /home/stf/tools/androidtools/platform-tools/adb
info AppiumDoctor  ✔ android exists at: /home/stf/tools/androidtools/tools/android
info AppiumDoctor  ✔ emulator exists at: /home/stf/tools/androidtools/tools/emulator
info AppiumDoctor  ✔ Bin directory of $JAVA_HOME is set
info AppiumDoctor ### Diagnostic for necessary dependencies completed, no fix needed. ###
info AppiumDoctor
info AppiumDoctor ### Diagnostic for optional dependencies starting ###
WARN AppiumDoctor  ✖ opencv4nodejs cannot be found.
WARN AppiumDoctor  ✖ ffmpeg cannot be found
WARN AppiumDoctor  ✖ mjpeg-consumer cannot be found.
WARN AppiumDoctor  ✖ bundletool.jar cannot be found
info AppiumDoctor ### Diagnostic for optional dependencies completed, 4 fixes possible. ###
info AppiumDoctor
info AppiumDoctor ### Optional Manual Fixes ###
info AppiumDoctor The configuration can install optionally. Please do the following manually:
WARN AppiumDoctor  ➜ Why opencv4nodejs is needed and how to install it: https://github.com/appium/appium/blob/master/docs/en/writing-running-appium/image-comparison.md
WARN AppiumDoctor  ➜ ffmpeg is needed to record screen features. Please read https://www.ffmpeg.org/ to install it
WARN AppiumDoctor  ➜ mjpeg-consumer module is required to use MJPEG-over-HTTP features. Please install it with 'npm i -g mjpeg-consumer'.
WARN AppiumDoctor  ➜ bundletool.jar is used to handle Android App Bundle. Please read http://appium.io/docs/en/writing-running-appium/android/android-appbundle/ to install it
info AppiumDoctor
info AppiumDoctor ###
info AppiumDoctor
info AppiumDoctor Bye! Run appium-doctor again when all manual fixes have been applied!
info AppiumDoctor

```



