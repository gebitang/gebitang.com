+++
title = "Window系统使用"
description = "windows usage tricks "
tags = [
    "windows",
    "work"
]
date = "2020-05-14"
topics = [
    "windows",
    "work"
]
toc = true
+++


重新整理日常工作中Windows环境下的配置以及使用技巧。默认最前面的内容为最新的内容。


<!-- more -->

## youtube-dl 

- 安装` pip install --upgrade youtube-dl`
- 下载`youtube-dl -F https://www.youtube.com/watch?v=x0uinJvhNxI` 列出所有可选择的格式
- 实际执行下载`youtube-dl -f 136 https://www.youtube.com/watch?v=x0uinJvhNxI`下载选择的对应文件到本地(执行命令的目录)
- 默认支持断点续传

## Mac PC远程访问Window PC

- 同一个局域网
- Mac上下载安装[Microsoft Remote Desktop](https://apps.apple.com/us/app/microsoft-remote-desktop/id1295203466) 需要非大陆账号的App Store才可以。参考[Apple美区账号注册之路](https://www.jianshu.com/p/7d00f6cef828)
- Window PC上开启允许远程访问权限[Remote Desktop - Allow access to your PC](https://docs.microsoft.com/en-us/windows-server/remote/remote-desktop-services/clients/remote-desktop-allow-access) 开始菜单--设置--系统--远程桌面

## Bose耳机蓝牙连接Mac电脑

[bose qc35 如何连接](https://www.zhihu.com/question/49721590/answer/154711284)

- 1. 手机上下载APP"Bose Connect"  
- 2. 打开耳机，用APP连接手机和耳机  
- 3. 打开APP，点里面左下角那个蓝牙符号，然后选择"连接新设备"  
- 4. 在Mac的偏好设置的蓝牙里，就可以找到耳机了，连接即可用



## Eclipse报错SWTError: No more handles

调研Jacoco，尝试用Eclipse打开源码——平时使用IDEA——Eclipse启动失败，log中提示`Caused by: org.eclipse.swt.SWTError: No more handles` 

```
!ENTRY org.eclipse.osgi 4 0 2020-07-15 16:12:47.033
!MESSAGE Application error
!STACK 1
org.eclipse.swt.SWTException: Failed to execute runnable (org.eclipse.swt.SWTError: No more handles)
	at org.eclipse.swt.SWT.error(SWT.java:4723)
	at org.eclipse.swt.SWT.error(SWT.java:4638)
	at org.eclipse.swt.widgets.Synchronizer.runAsyncMessages(Synchronizer.java:188)
	at org.eclipse.swt.widgets.Display.runAsyncMessages(Display.java:3897)
	at org.eclipse.swt.widgets.Display.readAndDispatch(Display.java:3527)
	at org.eclipse.swt.widgets.Display.release(Display.java:3579)
	at org.eclipse.swt.graphics.Device.dispose(Device.java:276)
	at org.eclipse.ui.internal.ide.application.IDEApplication.start(IDEApplication.java:166)
	at org.eclipse.equinox.internal.app.EclipseAppHandle.run(EclipseAppHandle.java:203)
	at org.eclipse.core.runtime.internal.adaptor.EclipseAppLauncher.runApplication(EclipseAppLauncher.java:137)
	at org.eclipse.core.runtime.internal.adaptor.EclipseAppLauncher.start(EclipseAppLauncher.java:107)
	at org.eclipse.core.runtime.adaptor.EclipseStarter.run(EclipseStarter.java:401)
	at org.eclipse.core.runtime.adaptor.EclipseStarter.run(EclipseStarter.java:255)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
	at java.lang.reflect.Method.invoke(Unknown Source)
	at org.eclipse.equinox.launcher.Main.invokeFramework(Main.java:657)
	at org.eclipse.equinox.launcher.Main.basicRun(Main.java:594)
	at org.eclipse.equinox.launcher.Main.run(Main.java:1447)
Caused by: org.eclipse.swt.SWTError: No more handles
	at org.eclipse.swt.SWT.error(SWT.java:4749)
	at org.eclipse.swt.SWT.error(SWT.java:4638)
	at org.eclipse.swt.SWT.error(SWT.java:4609)
	at org.eclipse.swt.widgets.Widget.error(Widget.java:432)
	at org.eclipse.swt.widgets.Control.internal_new_GC(Control.java:1771)
	at org.eclipse.swt.graphics.GC.<init>(GC.java:171)
	at org.eclipse.swt.graphics.GC.<init>(GC.java:135)
	at org.eclipse.swt.custom.CTabFolder.updateTabHeight(CTabFolder.java:3838)
	at org.eclipse.swt.custom.CTabFolder.runUpdate(CTabFolder.java:3871)
	at org.eclipse.swt.custom.CTabFolder.lambda$3(CTabFolder.java:3861)
	at org.eclipse.swt.widgets.RunnableLock.run(RunnableLock.java:40)
	at org.eclipse.swt.widgets.Synchronizer.runAsyncMessages(Synchronizer.java:185)
	... 17 more

```

[You may be creating resources (such as Font, Image or GC objects) that you aren't correctly disposing.](https://stackoverflow.com/a/2020777/1087122)——这个解释看起来靠谱。


[官方这个issue的comment](https://bugs.eclipse.org/bugs/show_bug.cgi?id=292464#c13)有建议修改注册表：

>I was able to workaround it temporarily until they fix this blocker with setting registry entry `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows\GDIProcessHandleQuota`  to some huge number like 50000 on Windows10

准备按照这个提示操作。本机默认值为  2710

![](https://upload-images.jianshu.io/upload_images/3296949-0d00a44b0f15a851.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[What is the cause of the "org.eclipse.swt.SWTError: No more handles" occurring in Process Designer](https://www.ibm.com/mysupport/s/question/0D50z000062kpwiCAA/what-is-the-cause-of-the-orgeclipseswtswterror-no-more-handles-occurring-in-process-designer?language=en_US)

这里提到出现问题的原因可能是GDI对象过多。[https://blogs.solidworks.com/tech/2017/05/warning-solidworks-resources-running-low.html](https://blogs.solidworks.com/tech/2017/05/warning-solidworks-resources-running-low.html)


>Windows 8 and later, the system wide GDI Objects are limited to max out at 65,536; and the maximum  single process is 16,384.  
>最多的GDI对象为65536个（2的16次方），单个进程最多16384个  
>The default limit set by Windows for any single process is 10,000 GDI objects. If your application GDI Objects exceeds this amount, that process is likely to crash.  
>默认的单进程GDI对象为10000个，如果超出，则可能崩溃  

打开任务管理器，在`详细信息`栏的表头右键进行`选择列`，在弹出窗口里可以看到`GDI对象`、`用户对象`的列名。

- Open Task Manager (right click on start bar > Task Manager OR through CTRL+ALT+DEL)
- Click on ‘Details’ Tab
- Right click on one of the columns and click on ‘Select Columns’
- Tick the GDI objects

![](https://upload-images.jianshu.io/upload_images/3296949-07d978121f154106.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意Eclipse中打开的项目的报错信息，即使处理错误信息。我只是清理了错误日志，重新打开Eclipse又OK了。另外，重启大法也好使。


## 禁止升级

[禁止Win 10自动更新](https://mp.weixin.qq.com/s/qoZu2Jo3gZ7ppjZayqDO7Q)   

[Disable Windows 10 Update in Every Way](https://www.easeus.com/todo-backup-resource/how-to-stop-windows-10-from-automatically-update.html#part4) 

[stop automatic updates on Windows 10](https://www.windowscentral.com/how-stop-updates-installing-automatically-windows-10)

- HKEY_LOCAL_MACHINE/SOFTWARE/Policies/Microsoft/Windows/WindowsUpdate "WindowsUpdate"可以手动创建
- 在WindowsUpdate下新建Key `AU`
- 新建一个` DWORD (32-bit)`类型的值，名称为`AUOptions`，选择16进制值，设置为`2`

```
2 — Notify for download and auto install.
3 — Auto download and notify for install.
4 — Auto download and schedule the install.
5 — Allow local admin to choose settings.
```

## 多桌面+分屏

类似Mac上的三指滑动切换工作桌面的效果。

- 默认左下角的`任务视图`点击后，最上方可以`新建桌面`
- 使用`Win` + `Ctl` + 左右方向键切换不同的桌面

## ConEmu 
在新tab页面批量执行不同的任务。[official doc](https://conemu.github.io/en/LaunchNewTab.html) 

ConEmu默契启动后在 ` C:\Users\userName`目录，可以将不同的执行脚本保存在当前目录下。例如：启动hugo、启动jekyll等

- 启动hugo脚本 `startHugoSite.bat`
```
#启动hugo站点
D:
cd  D:/openSources/site/hugositefolder
hugo server --watch --verbose --buildDrafts
```

- 启动jekyll脚本 `startJekyllSite.bat`
```
# 启动jekyll站点
D:
cd  D:/openSources/site/jekyllsitefolder
bundle exec jekyll serve
```

- 启动桌面应用脚本`startDesktop.bat`
```
#启动桌面应用
D:
cd D:\openSources\site\desktop-app
yarn electron:serve
```

- 其他上述脚本的脚本 `startAll.bat`
```
@echo off 
echo start trio
# https://superuser.com/a/593648
start "trio" "D:\tools\conEmu\ConEmu64.exe" /cmdlist ^> cmd /k startHugoSite.bat ^|^|^| cmd /k startJekyllSite.bat ^|^|^| cmd /k startDesktop.bat
echo finish jobs.
```

上述命令会新启动一个ConEmu窗口并在其中打开三个tab页面，分别执行三个启动脚本。强迫症如我，就主动退出当前窗口。使用新创建的window了。

bat脚本内容：`start "c" cmd /k call b.bat`中：
- “trio”是一段字符串，代表新打开的cmd窗口的名字，可以随便起名。
- /k是表示新打开的cmd窗口在执行完命令后保存打开状态，如果希望执行完就关闭窗口就使用/c

如果在当前脚本环境中执行，可直接使用`call`目录：
- call b.bat表示call命令，即调用b.bat文件；该命令可以用”“括起来，即：”call b.bat”




## Word里替换换行符 

查找`^p`替换 为空字符串即可。 在线[图片OCR识别](https://www.onlineocr.net/zh_hans/)准确率非常高，保留了图片的宽度，导致文本自动在固定宽度就换行了。 

软回车（垂直向下的箭头）为`^l`。`^p`称为硬回车（向下再左转的箭头）。

## 设置命令行代理

```
# 持续到cmd窗口关闭，非 系统环境变量
set http_proxy=http://127.0.0.1:1189
set https_proxy=http://127.0.0.1:1189
```


## 命令行查看MD5值

```
certutil -hashfile xxx MD5
certutil -hashfile xxx SHA1
certutil -hashfile xxx SHA256
```


## outlook证书弹出提示

- 查看证书；
- 选择安装证书；
- 选择自定义的存储区；
- 选择"受信任的根证书颁发机构"

（因为自定义的证书通常没有其他证书的签名，所以需要接受为“根证书”）

## 修改用户名


[How to Change Your Account Name in Windows 10](https://www.groovypost.com/howto/change-account-name-windows-10/)</br>
[How to Find Security Identifier (SID) of User in Windows](https://www.tenforums.com/tutorials/84467-find-security-identifier-sid-user-windows.html)</br>
[Security identifiers official doc](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/security-identifiers)

查看SID值：使用管理员打开cmd目录，执行`wmic useraccount`会展示所有用户的列表信息

```
PS D:\> whoami /user

用户信息
----------------

用户名           SID
================ ==============================================
gebitang\joechin S-1-5-21-3801529287-1954174758-2671260994-1001
PS D:\>
```

基本流程：</br>

1. 新建一个管理员账号并使用这个账号登陆
2. 修改原用户的名称
3. 修改注册码`regedit `修改`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Profilelist `下原用户名的security identifier (SID)的ProfileImagePath的值。
4. 重启切换到原用户登陆


上面的操作可以实际修改了用户名，但在系统设置-->环境变量中的UI显示还是修改之前的名称。可以使用`netplwiz`修改显示的用户名

Press Windows key + R, type: `netplwiz` or `control userpasswords2` then hit Enter. Select the account then click Properties.

## win10 鼠标一直显示转圈
[windows 10 鼠标光标经常出现蓝色转圈](https://answers.microsoft.com/zh-hans/windows/forum/windows_10-performance/windows-10/6f0f590c-997b-4daf-9b7b-7280dcdb080b)

通常为有应用一直在占用资源。我遇到的是 小米wifi网口一直未启动导致的（拔掉小米wifi后恢复正常），在设备管理器-->网络适配器中的adapter有个向下的箭头，右键可以选择启用设备。启用设备后，转圈提示消失。

## 系统自检wifi热点（需要无线网卡支持）

推荐使用小米wifi：即可以提供wifi热点给其他设备使用；又可以作为无线网卡连接其他wifi上网。

```
#0. 使用管理员权限运行cmd命令行
#1. 检查是否包含无线网卡 （没有无线网卡的就不可用）
netsh wlan show driver

接口名称: WLAN

    驱动程序                  : Broadcom BCM943228HMB 802.11abgn 2x2 Wi-Fi Adapter
    供应商                    : Broadcom
    提供程序                  : Broadcom
    日期                      : 2014/6/9
    版本                      : 6.30.223.245
    INF 文件                  : C:\windows\INF\oem46.inf
    文件                      : 4 total
                                C:\windows\system32\DRIVERS\BCMWL63a.SYS
                                C:\windows\system32\bcmihvsrv64.dll
                                C:\windows\system32\bcmihvui64.dll
                                C:\windows\system32\drivers\vwifibus.sys
    类型                      : 本机 WLAN 驱动程序
    支持的无线电类型          : 802.11n 802.11a 802.11g 802.11b
    支持 FIPS 140-2 模式: 是
    支持 802.11w 管理帧保护 : 是
    支持的承载网络  : 是
    基础结构模式中支持的身份验证和密码:
                                开放式             无
                                开放式             WEP
                                WPA2 - 企业       TKIP
                                WPA2 - 个人       TKIP
                                WPA2 - 企业       CCMP
                                WPA2 - 个人       CCMP
                                供应商定义的          供应商定义的
                                供应商定义的          供应商定义的
                                WPA - 企业        TKIP
                                WPA - 个人        TKIP
                                WPA - 企业        CCMP
                                WPA - 个人        CCMP
    临时模式中支持的身份验证和密码:
                                WPA2 - 个人       CCMP
                                开放式             无
                                开放式             WEP
    是否存在 IHV 服务         : 是
    IHV 适配器 OUI            : [00 10 18]，类型: [00]
    IHV 扩展 DLL 路径         : C:\windows\System32\bcmihvsrv64.dll
    IHV UI 扩展 ClSID         : {aaa6dee9-31b9-4f18-ab39-82ef9b06eb73}
    IHV 诊断 CLSID            : {00000000-0000-0000-0000-000000000000}

#2. 创建无线热点 网络连接中会多出一个网卡“Microsoft Virtual WiFi Miniport Adapter”
# mode：是否启用虚拟WIFI网卡，改为disallow则为禁用。
# ssid：无线网名称，最好用英文（以cai为例）。
# key：无线网密码，
netsh set hostednetwork mode=allow ssid=nameOfwifi key=pwdForssid
承载网络模式已设置为允许。
已成功更改承载网络的 SSID。
已成功更改托管网络的用户密钥密码。

 
#3. 启动/停止此连接 netsh wlan start/stop hostednetwork
C:\windows\system32>netsh wlan start hostednetwork
已启动承载网络。

#4. 设置本地的有线连接为共享模式
在“网络连接”窗口中，右键单击已连接到Internet的网络连接，
选择“属性”→“共享”，勾上“允许其他······连接（N）”并选择新创建的wifi连接点

```
netsh wlan set hostednetwork mode=allow ssid=cai key=12345678

## 开机自启动

```
#1. 编辑注册表 Ctrl+R --> regedit 
计算机\HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run

#2. 右键新建 "字符串"" 类型，自定义名字
#3. 知道该名称的数据值，即要启动应用的全目录，如 F:\Tools\ss\Shadowsocks.exe

# win10环境可以直接启动powershell类型脚本。
# a). 新建脚本保存为.ps1的后缀; b). 修改默认打开应用，选择 powershell
#（默认路径在 %SystemRoot%\system32\WindowsPowerShell\v1.0\powershell.exe）
# 即 %HOMEDRIVE%%HOMEPATH% = C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

## 获取时间戳
[获取系统日期、时间戳记](http://atgfss.iteye.com/blog/354054)</br>
[get current time in windows command line](https://stackoverflow.com/questions/203090/how-do-i-get-current-datetime-on-the-windows-command-line-in-a-suitable-format/)


格式： %date:~x,y%以及%time:~x,y% </br>
说明： x是开始位置，y是取得字符数 </br>
```
# 获取完整的日期和时间， 
格式： %date:~0,4%%date:~5,2%%date:~8,2%%time:~0,2%%time:~3,2%%time:~6,2% 
结果： 20180316175313 
```

## bat注释 
[bat 的注释方法](https://blog.csdn.net/zhangmiaoping23/article/details/56839106)

在批处理中，段注释有一种比较常用的方法：

     goto start
      = 可以是多行文本，可以是命令
      = 可以包含重定向符号和其他特殊字符
      = 只要不包含 :start 这一行，就都是注释
     :start

这样会跳过之间的三行，也就相当于注释


另外，还有其他各种注释形式，比如：

1. :: 注释内容（第一个冒号后也可以跟任何一个非字母数字的字符）
2. rem 注释内容（不能出现重定向符号和管道符号）
3. echo 注释内容（不能出现重定向符号和管道符号）〉nul
4. if not exist nul 注释内容（不能出现重定向符号和管道符号）
5. :注释内容（注释文本不能与已有标签重名）
6. %注释内容%（可以用作行间注释，不能出现重定向符号和管道符号）
7. goto 标签 注释内容（可以用作说明goto的条件和执行内容）
8. :标签 注释内容（可以用作标签下方段的执行内容）

## CMD重定向

[cmd命令的重定向输出](https://www.cnblogs.com/dewin/archive/2009/09/19/1569869.html)
```
# 2>&1
java -Dshow.debug.info=true -jar ./App.jar >> log-%date:~0,4%%date:~5,2%%date:~8,2%%time:~0,2%%time:~3,2%%time:~6,2%.txt 2>&1
```

## PowerShell 执行 mvn -D 参数问题

在Powershell环境下执行类似 `mvn clean package assembly:assembly -Dmaven.test.skip=true`命令时，参数会被截断，导致提示 test.skip不识别的错误。

[参考](https://stackoverflow.com/a/6351739/1087122) 需要把最后的参数使用单引号: `mvn clean package assembly:assembly '-Dmaven.test.skip=true'`


## 杀死进程
```
#命令行杀死进程
#查看进程 
tasklist
   
#杀死进程 查看帮助 taskkill /?
taskkill /PID 11112
# 强制kill
taskkill /F /PID 4684
taskkill /IM notepad.exe
```

## bat目录提示‘此处不该有xxx’
```
@echo off
set port=%1
set EXISTS_FLAG=false
for /f "tokens=2" %%i in ('netstat -ano ^|findstr ":%port%"') do echo %%i|findstr /E ":%port%" && set EXISTS_FLAG=true && exit
```
for/f中的命令如果有特殊字符需要加转义字符^，您的批处理改成这样就行了。

## 提高CMD显示行数

修改属性、布局、屏幕缓冲区大小、高度。

如果命令行输出被截断，进行重定向吧:(

mode命令显示当前窗口设置状态
```
F:\>mode

设备状态 CON:
---------
    行:　       3000
    列:　　     80
    键盘速度:   31
    键盘延迟:　 1
    代码页:     936

```

## 窗口保护色 #c7edcc

[在线RGB取色器](http://xiaohudie.net/RGB.html)

[修改窗口背景颜色为护眼颜色](https://blog.csdn.net/u010739973/article/details/45675515)，需要重启生效。

```
202 234 206 #caeace
199 237 204 #c7edcc

# HKEY_CURRENT_USER\Control Panel\Colors\下的Window的值默认为255 255 255（白色）， 设置为199 237 204（浅绿）
# HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Themes\DefaultColors\Standard下的Window的数值默认 ffffff，修改为
```

## CMD here for win10

[How to return the 'Open command window here' option to Windows 10's context menu](https://www.windowscentral.com/add-open-command-window-here-back-context-menu-windows-10)这个链接操作了半天还是不好使。直接修改注册表吧，不能迷信外语资料:)

报错下面的内容到文本文件，重命名为.reg，双击执行即可。 

说明：修改不同的注册表值。
```

```
给以下注册表添加了新内容：
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\background\shell]
增加一个条目cmd_here：指定名称、icon和执行的命令

[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Folder\shell]
增加一个条目cmdPrompt：指定名称、icon和执行的命令

[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\shell]
增加一个条目cmd_here：指定名称、icon和执行的命令
```
Windows Registry Editor Version 5.00
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\background\shell\cmd_here]
@="cmd here"
"Icon"="cmd.exe"
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\background\shell\cmd_here\command]
@="\"C:\\Windows\\System32\\cmd.exe\""
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Folder\shell\cmdPrompt]
@="cmd here"
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Folder\shell\cmdPrompt\command]
@="\"C:\\Windows\\System32\\cmd.exe\" \"cd %1\""
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\shell\cmd_here]
@="cmd here"
"Icon"="cmd.exe"
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\shell\cmd_here\command]
@="\"C:\\Windows\\System32\\cmd.exe\""

```


## Notepad++ 插件管理

默认的安装是不带PluginManager的，官方下载，指向到[github地址](https://github.com/bruderstein/nppPluginManager/releases)。 32为的选择后缀为UNI的，下载解压到对应的安装目录plugin、update文件夹下。

通常用的三个插件：compare、JSON Viewer、XML tools。通过PM安装、重启即可。

## 经纬度查询告别百度

基本告别百度了。搜索“经纬度查询”，第一页没看到可用的信息。

搜索 `search map by`， 自动提示选择有 ` latitude and longitude` 然后第一个答案就是官方的支持内容。

更简单的是直接在map的搜索框里输入经纬度值即可。支持三种数值格式。

手机自带的指南针应用提供的正好是第一种格式，类似`39°57'26"N 116°30'19"E`

---

[18539](https://support.google.com/maps/answer/18539)

### Enter coordinates to find a place

1.  On your computer, open [Google Maps](https://www.google.com/maps).
2.  In the search box at the top, type your coordinates. Here are examples of formats that work:
    *   Degrees, minutes, and seconds (DMS): 41°24'12.2"N 2°10'26.5"E
    *   Degrees and decimal minutes (DMM): 41 24.2028, 2 10.4418
    *   Decimal degrees (DD): 41.40338, 2.17403
3.  You'll see a pin show up at your coordinates.

### Get the coordinates of a place

1.  On your computer, open [Google Maps](https://www.google.com/maps). If you're using Maps in [Lite mode](https://support.google.com/maps/answer/3031966), you’ll see a lightning bolt at the bottom and you won't be able to get the coordinates of a place.
2.  Right-click the place or area on the map.
3.  Select **What's here?**
4.  At the bottom, you’ll see a card with the coordinates.

### Tips for formatting your coordinates

Here are some tips for formatting your coordinates so they work on Google Maps:

*   Use the degree symbol instead of "d".
*   Use periods as decimals, not commas. Incorrect: `41,40338, 2,17403`. Correct: `41.40338, 2.17403`. 
*   List your latitude coordinates before longitude coordinates.
*   Check that the first number in your latitude coordinate is between -90 and 90.
*   Check that the first number in your longitude coordinate is between -180 and 180.

