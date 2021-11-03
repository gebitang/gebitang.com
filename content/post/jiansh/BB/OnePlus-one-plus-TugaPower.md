+++
title = "OnePlus-one-plus-TugaPower"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "BB"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/6ffa4ee27e5a)

环境准备：
- Twrp3.3.0 工具
- TugaPower 安装包（底包TugaPowerFirmwareCM13_v4.zip、系统包：TugaPowerP12_OP1.zip）
- adb、fastboot工具（包含在android开发者环境中，也可以独立安装）

步骤：
- 0. 将twrp、底包、系统包复制到`sdcard`中
```
$> adb push .\twrp.img /sdcard/
.\twrp.img: 1 file pushed. 5.0 MB/s (14215168 bytes in 2.704s)
$> adb push .\TugaPowerFirmwareCM13_v4.zip /sdcard/
.\TugaPowerFirmwareCM13_v4.zip: 1 file pushed. 5.1 MB/s (32680540 bytes in 6.094s)
$> adb push .\TugaPowerP12_OP1.zip /sdcard/
.\TugaPowerP12_OP1.zip: 1 file pushed. 5.7 MB/s (457928626 bytes in 76.678s)
```
- 1. 进入bootloader环境 `adb reboot bootloader`
- 2. 刷如Twrp工具 `fastboot flash recovery twrp.img` 解锁oem(非必须)`fastboot oem unlock`  
- 3. 强制关机，然后`音量键下 + 电源键`， 进入recovery模式  
- 4. 进入当前模式之后，默认首先会进入twrp工具  
- 4.1 执行四清操作：wipe/清理以下四个文件夹(Dalvik/ART Cache、 System、 Data、 Cache)  
- 4.2 安装底包：install 安装 `TugaPowerFirmwareCM13_v4.zip`  
- 4.3 安装系统包：install `TugaPowerP12_OP1.zip`  
- 5. 重启系统，进入TugaPower系统  

安装完毕后，如果希望将旧系统的所有数据都清除，可以重复 3 ~ 4.1步骤，选择清除 internal storage即可。

系统占用 2.7G左右。

--- 

一加3T：

- 需要在开发者选项中打开“允许oem unlock”的选项
- 然后先执行`fastboot oem unlock`之后再刷twrp，详细[参考](https://segmentfault.com/a/1190000012314376)

然后我不小心五清了所有数据，导致进入twrp之后找不到安装文件，只能使用sideload方式进行安装——

- 进入twrp之后，选择advanced选项，进入sideload模式
- 直接刷机：`adb sideload TugaPowerP23_OP3.zip`，刷完之后，重启手机即可

--- 
[参考、物料](https://www.jianshu.com/p/8d5ac011e907)


TugaPower：
[论坛](https://board.tugapower.net/showthread.php?tid=26)  
[tugapower PIE download](https://tugapower.net/?dir=TP%2FOP1%2FPIE)


[一加一手机不支持电信网络制式](https://club.jd.com/consultation/1169454-36162072.html)

