+++
title = "STF之Docker-for-Mac失败"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "STF"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/285868401ca1)

###容器访问
[外部访问容器](https://docker_practice.gitee.io/network/port_mapping.html) 
启动容器`docker container ls`
进入容器 `docker attach container_id`
退出容器不关闭容器 `Ctrl+q Ctrl+q`

### adb basic
[adb detail](https://developer.android.google.cn/studio/command-line/adb)
[How ADB works](https://events.static.linuxfound.org/images/stories/pdf/lf_abs12_kobayashi.pdf)
[How ADB works cnblogs](https://www.cnblogs.com/ifantastic/p/5186362.html)

- **A client**, which sends commands. The client runs on your development machine. You can invoke a client from a command-line terminal by issuing an adb command.
- **A daemon (adbd)**, which runs commands on a device. The daemon runs as a background process on each device.
- **A server**, which manages communication between the client and the daemon. The server runs as a background process on your development machine.

### Use adb from Docker Container
[Use ADB to Connect to Your Android Device From a Docker Container](https://dustinoprea.com/2018/12/05/use-adb-to-connect-to-your-android-device-from-a-docker-container/)
[Connecting to a USB Android device in a Docker container via ADB ](https://stackoverflow.com/a/49003099/1087122)
[You want to use devices like /dev/ttyUSB0 or need to access raw USB devices under /dev/bus/usb in a docker container](http://marc.merlins.org/perso/linux/post_2018-12-20_Accessing-USB-Devices-In-Docker-_ttyUSB0_-dev-bus-usb-_-for-fastboot_-adb_-without-using-privileged.html) 在linux宿主机环境下可行
[Docker Network and ADB Devices](http://wykvictor.github.io/2016/03/14/Docker-Network.html)依然需要virtual box的方式，依赖usb 识别工具
[Getting a USB device to show up in a Docker container on OS X](https://gist.github.com/stonehippo/e33750f185806924f1254349ea1a4e68)
以上所有的讨论方案都依赖linux宿主环境

### 结论
[stf 讨论](https://github.com/sorccu/docker-adb/issues/8)
[docker 官方讨论](https://github.com/docker/for-mac/issues/900)
[sorccu docker-adb](https://github.com/sorccu/docker-adb) 这里都使用说明也表面都需要将文件挂载到docker中，否则无法获取到usb设备

[Mounts denied. The paths … are not shared from OS X and are not known to Docker](https://stackoverflow.com/questions/45122459/docker-mounts-denied-the-paths-are-not-shared-from-os-x-and-are-not-known)

在Mac环境下都Docker，暂时无法直接接入宿主机上连接都设备
