+++
title = "STF"
description = "Everything about STF"
tags = [
]
date = "2020-04-27"
topics = [
    "stf",
    "tech"
]
toc = true
+++

STF的核心功能可以理解为：“同步图像” + “点击”。前者使用minicap完成，后者依赖minitouch。

虽然目前依然是3.4.1版本，但由于增加了很多定制化的内容，所以从——

>commit: 16e64b51ff3dbb5cf7492fa773c123ceb12454b0   
>Merge pull request #770 from neofreko/adb-key-api Implements addAdbPublicKey API endpoint  
> Simo Kinnunen on 7/11/2019, 2:44:32 AM  

之后就没同步最新代码。

手机自动被升级到了Android 10，手动同步了minicap，但发现minitouch已经不被支持了——

### For Android 10 and up

[minitouch about Android 10](https://github.com/openstf/minitouch#for-android-10-and-up)

Minitouch can't handle Android 10 by default, due to a new security policy. The workaround is to forward touch commands to STFService. If you are using minicap standalone (without STF), you need to take care of running the service and agent, before running minicap. Instructions on how to do that can be found [here](https://github.com/openstf/STFService.apk#running-the-service).

```
 minitouch says: "Unable to open device /dev/input/event0 for inspectionopen: Permission denied"
```

解决[方案](https://github.com/openstf/minitouch/issues/49#issuecomment-582946443)：

Minitouch cannot support Android 10 due to a new security policy. To overcome that a fallback has been added to forward touch events to STFService.apk. Latest changes in STFService.apk aim at managing those events at the framework level.
Right now if you are using minitouch (outside of openstf scope), you simply have to:

*   [run the STFService agent](https://github.com/openstf/STFService.apk#running-the-agent)
*   [then run minitouch](https://github.com/openstf/minitouch#running)

--- 

人肉同步合并一下最新代码试试。

合并代码之后，可以正常镜像远程控制（图像同步，点击操作），但`stf local`场景下的用户登录有点问题（使用ldap登录似乎又是正常的），关键是屏幕截图却不行了。一直报错`connect ECONNREFUSED`。

[Error: connect ECONNREFUSED 127.0.0.1:8087](https://blog.csdn.net/u010411264/article/details/53899599)

```
npm config set proxy null
```

从这个文章获得启发，我本地环境使用了VPN，terminal里做了export的动作，尽管VPN已经关闭，但转发依然可能生效。重启了一个terminal环境，果然正常了。