+++
title = "linux and shell"
description = "linux work os"
tags = [
    "linux",
    "work"
]
date = "2018-12-17"
topics = [
    "work"
]
toc = true
+++


## linux 环境

Root can run commands as other users via the "su" command. I believe if you create or edit /etc/rc.local and add this line to it:

`su -c /path/to/your/script username`

### 命令行打开目录，设置别名 

```
nautilus

#设置别名： update profile such as ~/.bashrc file
alias open="nautilus"
```
 

### ubuntu 18.04开机启动

[设置开机启动脚本](http://www.cnblogs.com/defifind/p/9285456.html)， 18.04使用了 [SYSTEMD](https://coolshell.cn/articles/17998.html)。

[execute shell at start up](https://linuxconfig.org/how-to-automatically-execute-shell-script-at-startup-boot-on-systemd-linux), [开机启动脚本](http://www.cnblogs.com/defifind/p/9285456.html)， 手动创建 `/etc/systemd/system/rc-local.service`和 `/etc/rc.local`文件， 后者需要赋予可执行权限。

### use oh my zsh
[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)

```
# need to install zsh first
sudo apt-get -y instal zsh

# zsh --version to check version
```

### screenshots and screencasts

[screenshots and screencasts](https://help.ubuntu.com/stable/ubuntu-help/screen-shot-record.html)

- Prt Scrn to take a screenshot of the desktop.
- Alt+Prt Scrn to take a screenshot of a window.
- Shift+Prt Scrn to take a screenshot of an area you select.

You can also hold down Ctrl with any of the above shortcuts to copy the screenshot image to the clipboard instead of saving it.

### curl命令调用post方法

curl: (7) Failed to receive SOCKS4 connect request ack. 原因为本地网络设置了代理服务；关闭本地代理服务之后，需要新启动一个terminal才会生效。

```
curl -i -H "Accept:application/json" -H "Content-Type:application/json" -XPOST "http://sss.com/abc/" -d '{"op":"Device.restartDevice","device":{"imei":"866568023794679"}}'
```

### wireless adapter

first: [Fix ‘No WiFi Adapter Found’ with Ubuntu 18.04](http://ubuntuhandbook.org/index.php/2018/08/no-wifi-adapter-found-hp-laptops-ubuntu-18-04/), it says ["Required key not avilable"](https://askubuntu.com/questions/762254/why-do-i-get-required-key-not-available-when-install-3rd-party-kernel-modules), need to disable secure boot. and install new package as following: [Linux wireless driver for HP 14-am100na notebook](https://h30434.www3.hp.com/t5/Notebook-Wireless-and-Networking/Linux-wireless-driver-for-HP-14-am100na-notebook/td-p/6091885), finally, it works.

`sudo apt-get install bcmwl-kernel-source`

### Sublime Text 

[issue 60](https://github.com/lyfeyaj/sublime-text-imfix/issues/60)，目前不支持系统自带的中文输入法。使用VS Code代替。

[Official Doc](http://www.sublimetext.com/docs/3/)</br>
[Eclipse、Sublime快捷键](https://www.zybuluo.com/Gebitang/note/82413)
[install doc](http://www.sublimetext.com/docs/3/linux_repositories.html)
Install the GPG key:
```
wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
```
Ensure apt is set up to work with https sources:
```
sudo apt-get install apt-transport-https
```
Select the channel to use:

Stable
```
echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
```
Dev
```
echo "deb https://download.sublimetext.com/ apt/dev/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
```
Update apt sources and install Sublime Text
```
sudo apt-get update
sudo apt-get install sublime-text
```


### 16.04 同步时间
[How To Set Up Time Synchronization on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-time-synchronization-on-ubuntu-16-04)
时间不同步将导致log文件无法生产。
```

$>timedatectl
      Local time: Fri 2018-11-16 10:04:45 CST
  Universal time: Fri 2018-11-16 02:04:45 UTC
        RTC time: Fri 2018-11-16 02:04:45
       Time zone: Asia/Chongqing (CST, +0800)
 Network time on: yes
NTP synchronized: yes
 RTC in local TZ: no

# avaliable time zone 
timedatectl list-timezones

# set time zone 
timedatectl set-timezone Asia/Shanghai

# check time 
date 
```


### Gtk-Message: 23:27:47.368: Failed to load module "canberra-gtk-module"

1.  先按照对应的lib库；
2. 依然报错后，使用 cat + more( ` cat  log-20181105232733.txt|more`)的方式可以查看。 会先报错，然后可以恢复正常。 

[Failed to load module “canberra-gtk-module” … but already installed](https://askubuntu.com/q/342202)

```
# not work
apt-get install libcanberra-gtk-module:i386
# not work 
apt install libcanberra-gtk-module libcanberra-gtk3-module

##whole log
root@testin-OptiPlex-7050:/home/testin/桌面/xcdir/_logs# apt install libcanberra-gtk
libcanberra-gtk0            libcanberra-gtk3-dev        libcanberra-gtk-common-dev  libcanberra-gtk-module
libcanberra-gtk3-0          libcanberra-gtk3-module     libcanberra-gtk-dev
root@testin-OptiPlex-7050:/home/testin/桌面/xcdir/_logs# apt install libcanberra-gtk
libcanberra-gtk0            libcanberra-gtk3-dev        libcanberra-gtk-common-dev  libcanberra-gtk-module
libcanberra-gtk3-0          libcanberra-gtk3-module     libcanberra-gtk-dev
root@testin-OptiPlex-7050:/home/testin/桌面/xcdir/_logs# apt install libcanberra-gtk-module
正在读取软件包列表... 完成
正在分析软件包的依赖关系树
正在读取状态信息... 完成
libcanberra-gtk-module 已经是最新版 (0.30-5ubuntu1)。
升级了 0 个软件包，新安装了 0 个软件包，要卸载 0 个软件包，有 6 个软件包未被升级。
root@testin-OptiPlex-7050:/home/testin/桌面/xcdir/_logs# sed -n '1,30p' log-20181105232733.txt
Gtk-Message: 23:27:47.368: Failed to load module "canberra-gtk-module"
log4j:ERROR setFile(null,true) call failed.
java.io.FileNotFoundException: /home/testin/桌面/xcdir/logs/debug.log (权限不够)
        at java.io.FileOutputStream.open0(Native Method)
        at java.io.FileOutputStream.open(FileOutputStream.java:270)
        at java.io.FileOutputStream.<init>(FileOutputStream.java:213)
        at java.io.FileOutputStream.<init>(FileOutputStream.java:133)
        at org.apache.log4j.FileAppender.setFile(FileAppender.java:294)
        at org.apache.log4j.FileAppender.activateOptions(FileAppender.java:165)
        at org.apache.log4j.DailyRollingFileAppender.activateOptions(DailyRollingFileAppender.java:223)
        at org.apache.log4j.config.PropertySetter.activate(PropertySetter.java:307)
        at org.apache.log4j.config.PropertySetter.setProperties(PropertySetter.java:172)
        at org.apache.log4j.config.PropertySetter.setProperties(PropertySetter.java:104)
        at org.apache.log4j.PropertyConfigurator.parseAppender(PropertyConfigurator.java:842)
        at org.apache.log4j.PropertyConfigurator.parseCategory(PropertyConfigurator.java:768)
        at org.apache.log4j.PropertyConfigurator.parseCatsAndRenderers(PropertyConfigurator.java:672)

```

### 修改core文件配置

[setting the core dump name schema](http://www.linuxhowtos.org/Tips%20and%20Tricks/coredump.htm)</br>
[specify filename and path for core dumps](https://sigquit.wordpress.com/2009/03/13/the-core-pattern/)</br>
[to permanently edit the core_pattern file](https://askubuntu.com/a/420628)

修改 `/etc/sysctl.conf`文件；或在`/etc/sysctl.d`文件夹下创建一个以`.conf`结尾的文件。内容为`kernel.core_pattern = core.%e.%p.%h` 也可以在这里直接指定目录

```
%e is the filename
%g is the gid the process was running under
%p is the pid of the process
%s is the signal that caused the dump
%t is the time the dump occurred
%u is the uid the process was running under
```


### 查找文件中某字符串的个数

[统计一个文件中特定字符的个数](https://blog.csdn.net/wenwenxiong/article/details/49028223)

统计一个文件中某个字符串的个数，其实就是在在一块沙地里面找石头，有的人看到石头以后，在上面做个标记（grep），然后记住自己做了多少个标记；有的人看到石头以后，把它挖了（tr：匹配不了字符串，只能去匹配单个字符），最后统计自己挖了多少石头；有的人看到石头以后，把它跳过去（awk），然后统计自己跳了多少次。


```
# -c只能统计一行的；-o 即使在一行也会被计算在内
cat /data/itestin-m/config/devices.xml |grep -o "</device>" |wc -l

# -v 去设定一个变量的值，RS是记录的分隔符，默认的是新行(\n)
# 就是说awk按照一行一行读数据，但是现在RS为'</device>'后，就按'</device>'读数据
# NR为已读的记录数，n个记录是被n-1个分隔符分开的，所以就是--NR
awk -v RS="</device>" 'END {print --NR}' config/devices.xml
```


### Bad: new password is too simple 

使用root用户进行密码修改即可。 `sudo su; passwd username;` or `sudo passwd username`

### 查看本地网关

`netstat -rn`

### Netplan error in network definition expected mapping

[Netplan error in network definition expected mapping](https://askubuntu.com/questions/1088848/netplan-error-in-network-definition-expected-mapping)
a netplan configuration is base on yaml, when looking your configuration. i think the indentation is error because its have 3 space. 

### 修改18.04的网卡默认名称
[configure static IP address on Ubuntu 18.04](https://linuxconfig.org/how-to-configure-static-ip-address-on-ubuntu-18-04-bionic-beaver-linux)
[How to Configure Network Static IP Address in Ubuntu 18.04](https://www.tecmint.com/configure-network-static-ip-address-in-ubuntu/)
[Ubuntu 18.04 网卡配置IP](https://blog.csdn.net/peyte1/article/details/80509056)

编辑 `/etc/netplan/`文件夹下的 `/etc/netplan/01-network-manager-all.yaml` (最后的文件名可能不同)。然后执行 `netplan apply`命令立即生效

```
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: yes
    enp0s8:
      dhcp4: no
      dhcp6: no
      # /24意味着掩码为 255.255.255.0
      addresses: [192.168.56.110/24, ]
      gateway4:  192.168.56.1
      nameservers:
              addresses: [8.8.8.8, 8.8.4.4]
```
[ip段/数字，如192.168.0.1/24的意思是什么](https://blog.csdn.net/www3300300/article/details/38680843)

表示掩码用二进制表示时1的位数
比如/24表示11111111.11111111.11111111.00000000,就是255.255.255.0

### 修改16.04的网卡默认名称

[ubuntu 16.04 重命名网卡](https://blog.csdn.net/u013932687/article/details/70344686)
```
#1. ip a 或 ifconig查看网卡
#2. 查看eth名称变化
root@ubuntu:~# dmesg |grep -i eth
[    3.012281] r8169 Gigabit Ethernet driver 2.3LK-NAPI loaded
[    3.014877] r8169 0000:02:00.0 eth0: RTL8168evl/8111evl at 0xffffc90000646000, 74:d4:35:23:0f:2b, XID 0c900800 IRQ 24
[    3.016230] r8169 0000:02:00.0 eth0: jumbo features [frames: 9200 bytes, tx checksumming: ko]
[    3.090803] r8169 0000:02:00.0 enp2s0: renamed from eth0

#3. 编辑文件 etc/default/grub
# 修改 GRUB_CMDLINE_LINUX="" 为 GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"

#4. 生成新grub配置文件（该文件可能已经存在）
sudo grub-mkconfig -o /boot/grub/grub.cfg

#
# DO NOT EDIT THIS FILE
#
# It is automatically generated by grub-mkconfig using templates
# from /etc/grub.d and settings from /etc/default/grub
#

#5.编辑静态ip配置，如果是DHCP设置ip，则可以忽略此步
/etc/network/interfaces


# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto eth0
iface eth0 inet static
    address 10.34.16.93
    netmask 255.255.255.0
    network 10.34.16.0
    broadcast 10.34.16.255
    gateway 10.34.16.1
    # dns-* options are implemented by the resolvconf package, if installed
    dns-nameservers 114.114.114.114

#6. 需要重启生效

#7. 批量处理上述业务的脚本
#!/bin/bash

oldname=$(ifconfig |grep HWaddr |awk '{print $1}')

sed -i "s/GRUB_CMDLINE_LINUX=\"\"/GRUB_CMDLINE_LINUX=\"net.ifnames=0 biosdevname=0\"/g" /etc/default/grub

grub-mkconfig -o /boot/grub/grub.cfg

sed -i "s/$oldname/eth0/g" /etc/network/interfaces
```



### echo 换行 

```
# -e 参数将解析换行符，以及变量值的实际值
root@ubuntu:~# echo -e 'text\ntest'
text
test

# 更新profile
 echo 'export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64'>>/etc/profile; echo 'export JRE_HOME=${JAVA_HOME}/jre'>>/etc/profile;echo 'export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib'>>/etc/profile;echo 'export PATH=${JAVA_HOME}/bin:$PATH'>>/etc/profile

```
### 配置定时任务 crontab

[定时任务Crontab命令详解](https://www.cnblogs.com/intval/p/5763929.html)

查看crontab的log信息： `/var/log/cron`或 `/var/log/syslog`

minute hour day month week command

其中：

minute： 表示分钟，可以是从0到59之间的任何整数。

hour：表示小时，可以是从0到23之间的任何整数。

day：表示日期，可以是从1到31之间的任何整数。

month：表示月份，可以是从1到12之间的任何整数。

week：表示星期几，可以是从0到7之间的任何整数，这里的0或7代表星期日。

command：要执行的命令，可以是系统命令，也可以是自己编写的脚本文件。

```
星号（*）：代表所有可能的值，
例如month字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。

逗号（,）：可以用逗号隔开的值指定一个列表范围，
例如，“1,2,5,7,8,9”

中杠（-）：可以用整数之间的中杠表示一个整数范围，
例如“2-6”表示“2,3,4,5,6”

正斜线（/）：可以用正斜线指定时间的间隔频率，
例如“0-23/2”表示每两小时执行一次。
同时正斜线可以和星号一起使用，例如*/10，如果用在minute字段，表示每十分钟执行一次。

# crontab -e to edit this file via root user.
 crontab -l
# Edit this file to introduce tasks to be run by cron.
#
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
#
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').#
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
#
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
#
# For more information see the manual pages of crontab(5) and cron(8)
#
# m h  dom mon dow   command

```

### Make shell execute by dobule click
[How to make a shell file execute by double click ](https://askubuntu.com/questions/465531/how-to-make-a-shell-file-execute-by-double-click/465549#465549)

To run your script by double clicking on its icon, you will need to create a .desktop file for it:
```
[Desktop Entry]
Name=My script
Comment=Test hello world script
Exec=/home/user/yourscript.sh
Icon=/home/user/youricon.gif
Terminal=false
Type=Application
```
Save the above as a file on your Desktop with a .desktop extension. Change `/home/user/yourscript.sh` and `/home/user/youricon.gif` to the paths of your script and whichever icon you want it ot have respectively and then you'll be able to launch by double clicking it.

make yourscript.sh executable, and use it to run the actual script. such as:
```
#!/bin/bash
cd /home/user/Desktop/plus/
echo "to start"
exec ./run.sh
```

[参考资料](https://help.ubuntu.com/community/UnityLaunchersAndDesktopFiles)</br>
[不支持相对路径](https://stackoverflow.com/a/3879096/1087122)


### Ubuntu 显示桌面快捷键 ctrl+win+D
### ubuntu 18.04 启动自动的拼音输入法

[ubuntu 18.04 安装自带的中文输入法](https://blog.csdn.net/mint_ying/article/details/80267005)

英文页面进行安装。需要先安装输入法 `sudo apt-get install ibus-pinyin` 

然后在 settings 的 Region & Language 的 Input Sources设置栏中

1. 点击 `Manage Installed Language` ，初次进入会安装些字体等相关信息。重启后使之生效。
2. 点击 + 添加 `Chinese(Intelligent Pinyin)`。重启后使之生效。 

在中文输入情况下，Shift键可以切换简繁体

### CentOS Another app is currently holding the yum lock; waiting for it to exit...

```
Another app is currently holding the yum lock; waiting for it to exit...
  The other application is: PackageKit
    Memory :  31 M RSS (348 MB VSZ)
    Started: Wed Feb 14 20:49:26 2018 - 02:02 ago
    State  : Running, pid: 18008
```
[解决方案](https://www.cnblogs.com/lzxianren/p/4254059.html)

- 查找运行yum的pid，kill后再强制删除对应的pid文件 `rm -f /var/run/yum.pid`


### 安装ubuntu
[下载镜像](http://old-releases.ubuntu.com/releases/)，使用[**Rufus**](https://rufus.akeo.ie/)，[创建启动盘](https://tutorials.ubuntu.com/tutorial/tutorial-create-a-usb-stick-on-windows)。

```
1. 识别U盘，
2. 选择MBR分区方案用于BIOS或UEFI模式(MBR partition scheme for UEFI)
3. 文件系统选择FAT32、簇大小8192字节
4. 点击光盘icon，选择镜像文件，创建iso镜像
```

### vmware player安装vmware tools
提示“正在进行简易安装时，无法手动启动 vmware tools 安装”。

解决：

1. 在虚拟机配置中，将CD/DVD中的镜像移除，修改为“使用物理设备”。 然后重新选择安装vmware tools工具。——会将此工具自动挂载到CD中。
2. 进入此目录，将对应的gz包复制到本地工作目录，解压后执行`./vmware-install.pl`根据提示处理即可

提示找不到 ipconfig、gcc等，需要安装对应的工具。

```
sudo apt update
# for ipconfig
sudo apt install net-tools
# for gcc 
sudo apt install build-essential
# check gcc version
gcc --version

# check system version for gcc
cat /proc/version
Linux version 4.15.0-22-generic (buildd@lgw01-amd64-013) (gcc version 7.3.0 (Ubuntu 7.3.0-16ubuntu3)) #24-Ubuntu SMP Wed May 16 12:15:17 UTC 2018
# this means you need gcc 7.3 version

gcc --version
gcc (Ubuntu 7.3.0-16ubuntu3) 7.3.0
Copyright (C) 2017 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

```


### Linux下adb不能识别Android设备


[Linux下adb不能识别Android设备](http://blog.csdn.net/momo0853/article/details/45726583)</br>
[adb连接不上android手机的终极解决方案](http://blog.csdn.net/liuqz2009/article/details/7942569)

利用 `lsusb` 获取usb设备的vendor信息、将此信息添加到 `~/.android/adb_usb.ini` 文件下（文件不存在则手动创建）

但 `lsusb` 本身就获取不到设备就尴尬了
```
root@testin-MS-7788:~/.android# lsusb
Bus 002 Device 039: ID 093a:2510 Pixart Imaging, Inc. Optical Mouse
Bus 002 Device 006: ID 1a40:0201 Terminus Technology Inc. FE 2.1 7-port Hub
Bus 002 Device 004: ID 1a40:0101 Terminus Technology Inc. Hub
Bus 002 Device 051: ID 2b0e:179d
Bus 002 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 051: ID 1a40:0201 Terminus Technology Inc. FE 2.1 7-port Hub
Bus 001 Device 050: ID 1a40:0101 Terminus Technology Inc. Hub
Bus 001 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
root@testin-MS-7788:~/.android#
root@testin-MS-7788:~/.android#
root@testin-MS-7788:~/.android#
root@testin-MS-7788:~/.android# adb devices -l
List of devices attached
LP036878A7130052555[usb#2-1.4]          device usb:2-1.4 product:X7_CN model:LEX651 device:le_x7

root@testin-MS-7788:~/.android# adb devices -l
List of devices attached

root@testin-MS-7788:~/.android# lsusb
Bus 002 Device 039: ID 093a:2510 Pixart Imaging, Inc. Optical Mouse
Bus 002 Device 006: ID 1a40:0201 Terminus Technology Inc. FE 2.1 7-port Hub
Bus 002 Device 004: ID 1a40:0101 Terminus Technology Inc. Hub
Bus 002 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 051: ID 1a40:0201 Terminus Technology Inc. FE 2.1 7-port Hub
Bus 001 Device 050: ID 1a40:0101 Terminus Technology Inc. Hub
Bus 001 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

### tar命令基本操作

```
#打包
tar -czvf ycdh.tar.gz ycdh.jar ycdh_lib tools config log4j.properties build.properties dh.sh noLog.sh updateself.sh

#查看文件，但不解压
tar -tzvf ycdh.tar.gz

#解压
tar -zxvf ycdh.tar.gz;
```

### ubuntu多版本java支持

#### 使用ppa/源方式安装
1. 添加ppa
```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
```

2. 安装oracle-java-installer
```
sudo apt-get install oracle-java6-installer
sudo apt-get install oracle-java7-installer
sudo apt-get install oracle-java8-installer
```

查看java多版本
```
sudo update-java-alternatives -l
```
切换到其它java版本
``` 
sudo update-java-alternatives -s java-8-oracle
```

### 设置时区
```
# 安装ntp，非必要
apt-get install ntp

# tzselect timezone
tzselct 
# then select your timezone
date -R # to check the result.

cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime # make it permenent.
```

ubuntu14.04上提示错误：
```
#ubuntu14.04的一个bug
$ tzselect
/usr/bin/tzselect: line 171:/home/ubuntu/iso3166.tab: No such file or  directory
/usr/bin/tzselect: time zone files are not set up correctly

#解决办法:
vim /usr/bin/tzselect
${TZDIR=`pwd`}
改为
${TZDIR=/usr/share/zoneinfo}
```

### 设置时间 

```
# date命令将日期设置为2014年6月18日
date -s 06/18/14

# 将时间设置为14点20分50秒
date -s 14:20:50

# 将时间设置为2014年6月18日14点16分30秒（MMDDhhmmYYYY.ss）
date 0618141614.30
```

### 查看启动时间

[Linux查看系统开机时间](http://www.cnblogs.com/kerrycode/p/3759395.html)
```
#  查看最后一次系统启动的时间
gebitang@ubuntu# who -b
         system boot  2017-09-19 16:25
gebitang@ubuntu# who -r
         run-level 5  2017-09-19 16:25
gebitang@ubuntu# last reboot
reboot   system boot  4.4.0-62-generic Tue Sep 19 16:25   still running
reboot   system boot  4.4.0-62-generic Tue Sep 19 14:56   still running
reboot   system boot  4.4.0-62-generic Tue Sep 19 20:12   still running
reboot   system boot  4.4.0-62-generic Tue Sep 19 18:50   still running
# top命令的提示信息中也有
```


### 16.04root密码及用户管理

1. 默认root密码是随机的，即每次开机都有一个新的root密码
2. 在终端输命令 sudo passwd，然后输入当前用户的密码
3. 输入新的密码并确认，此时的密码就是root新密码
4. 修改成功后，输入命令 su root，再输入新的密码

#### 用户管理
[Linux查看和剔除当前登录用户](https://www.cnblogs.com/saptechnique/archive/2012/04/10/2441383.html)

root权限，更多信息，[参考](http://www.cnblogs.com/guangbei/archive/2010/04/26/1721163.html)

/etc/group 的内容包括用户组（Group）、用户组口令、GID及该用户组所包含的用户（User），每个用户组一条记录；格式如下：
`group_name:passwd:GID:user_list `
在/etc/group 中的每条记录分四个字段：〔用户组名称〕：〔用户组密码〕：〔GID〕:〔用户列表〕

###useradd 添加用户

[linux下创建用户](http://www.cnblogs.com/ylan2009/articles/2321177.html)

```
（也可以使用adduser）用来创建新的用户帐号。参数：
-d 设置新用户的登陆目录
-e 设置新用户的停止日期，日期格式为MM/DD/YY
-f 帐户过期几日后永久停权。当值为0时帐号则立刻被停权。而当值为-1时则关闭此功能。预设值为-1
-g 使新用户加入群组
-G 使新用户加入一个新组。每个群组使用逗号“，”隔开，不可以夹杂空白字
-s 指定新用户的登陆Shell
-u 设定新用户的ID值
如：useradd getbitang -d /home/gebitang -u 525 -s /bin/bash 默认新用户就在自己同名的组里
成功创建一个新用户以后，在/etc/passwd文件中就会增加一行该用户的信息，其格式如下：
〔用户名〕：〔密码〕：〔UID〕：〔GID〕：〔身份描述〕：〔主目录〕：〔登陆Shell〕
其中个字段被冒号“：”分成7各部分。
由于小于500的UID和GID一般都是系统自己保留，不用做普通用户和组的标志，
新增加的用户和组一般都是UID和GID大于500的。

```
**可以使用`usermod`修改用户的信息，如： `usermod –d/home/user2 –s/bin/bash user2`** 使用`-s /sbin/nologin`表示不用于登录

```
#查看当前登录用户，需要root权限 w
# 第一列是用户名,
# 第二列是连接的终端,tty表示显示器,pts表示远程连接,
# 第三列是登陆时间,
root@localhost:~# w
 10:48:38 up 17 days, 19:06,  1 user,  load average: 0.00, 0.02, 0.05
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    10.32.22.66      10:10    6.00s  0.15s  0.00s w

# 踢掉在线用户
pkill -kill -t pts/0

# 查看登陆用户历史
node8:/ # last
root     pts/0        10.35.0.4        Fri Jan 19 15:20   still logged in
home   pts/0        10.34.18.238     Mon Jan  8 08:08 - 08:28  (00:20)
root     pts/0        10.35.0.4        Thu Jan  4 15:39 - 15:41  (00:01)

# 新建用户 
```

### passwd 修改密码 
新用户需要制定密码才可以登录。 `passwd gebitang`
根据系统的提示信息输入两次密码，系统会显示：

> passwd ：:all authentication tokens updated successfully

删除用户： `userdel -r gebitang` -r将用户目录下的文档一并删除。在其他位置上的文档也将一一找出并删除。
新建群组：`groupadd -g 555 testgroup`
删除群组： `groupdel testgroup`


### 命令行快捷键

```
#光标移到行首， 辅助：Ctrl All必须从头开始
Ctrl + A 
#光标移到行尾， E represents end
Ctrl + E 

#向前移动一个字符 forward；在vim环境——包括man command也是vim环境——中，则表示向下翻页
Ctrl + F 
#向后移动一个字符 backward 在vim环境——包括man command也是vim环境——中，则表示向上翻页
Ctrl + B 

#向前移动一个单词 辅助：Alt意味着alternative取舍，选择。所以是按照有意义的操作进行
Alt + F 
#向后移动一个单词
Alt + B 
```

### 强制清空命令行已有数据 ctlr + c



### 配置DNS服务器
[资料](https://segmentfault.com/a/1190000002387446)

* 修改DNS
* 锁定配置文件

```shell
# 添加 阿里的dns 223.5.5.5与223.6.6.6
sudo vi /etc/resolv.conf

# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
#nameserver 127.0.1.1
nameserver 223.5.5.5
nameserver 223.6.6.6

#  chattr - change file attributes on a Linux file system
sudo chattr +i /etc/resolv.conf

#解锁 man chattr for details.
sudo chattr -i /etc/resolv.conf
```

### 查看系统版本 cat /proc/version 

```
cat /proc/version

root@testin-DH:~# lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.5 LTS
Release:	14.04
Codename:	trusty
root@testin-DH:~# cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=14.04
DISTRIB_CODENAME=trusty
DISTRIB_DESCRIPTION="Ubuntu 14.04.5 LTS"
root@testin-DH:~# cat /etc/issue
Ubuntu 14.04.5 LTS \n \l

root@testin-DH:~# cat /proc/version
Linux version 4.4.0-31-generic (buildd@lgw01-43) (gcc version 4.8.4 (Ubuntu 4.8.4-2ubuntu1~14.04.3) ) #50~14.04.1-Ubuntu SMP Wed Jul 13 01:07:32 UTC 2016
root@testin-DH:~#
```

### 测试I/O数据 dd

```
time dd bs=64k count=4k if=/dev/zero of=test
4096+0 records in
4096+0 records out
268435456 bytes (268 MB, 256 MiB) copied, 1.01641 s, 264 MB/s

real    0m1.025s
user    0m0.000s
sys     0m0.940s

```

### 查找网站ip 

```
## 10086短信被劫持
~#nslookup bjmobliis.cn
Server:         127.0.1.1
Address:        127.0.1.1#53

Non-authoritative answer:
Name:   bjmobliis.cn
Address: 45.124.113.90

# ping 
ping bjmobliis.cn
PING bjmobliis.cn (45.124.113.90) 56(84) bytes of data.
64 bytes from 45.124.113.90: icmp_seq=1 ttl=109 time=120 ms
64 bytes from 45.124.113.90: icmp_seq=2 ttl=109 time=128 ms
64 bytes from 45.124.113.90: icmp_seq=3 ttl=109 time=117 ms
64 bytes from 45.124.113.90: icmp_seq=4 ttl=109 time=120 ms
64 bytes from 45.124.113.90: icmp_seq=5 ttl=109 time=123 ms
64 bytes from 45.124.113.90: icmp_seq=6 ttl=109 time=119 ms
64 bytes from 45.124.113.90: icmp_seq=7 ttl=109 time=127 ms

```


### SSH Server安装
1. 先创建一个root账号，命令：
sudo passwd root
输入新密码 xxxxx

2. 安装ssh server. 如果是刚安装好系统，第一步是先更新一下源。
sudo apt-get update
更新完了后，安装ssh server 


3. 检查ssh server配置，允许root用户登录：

```
sudo gedit /etc/ssh/sshd_config
找到PermitRootLogin = without-password改成PermitRootLogin yes 最后执行一下
```

```
# 测试可用
sudo apt-get install ssh

#或
sudo apt-get install openssh-server --fix-missing
```

启动 sudo service ssh start

### 安装oracel jdk 8
```
# method 1: support now.
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer

# method 2: 需要手动安装：下载tar包、解压、修改环境变量
wget http://some.place.to.download.tar/install/jdk-8u192-linux-x64.tar.gz
tar -zxvf jdk-8u192-linux-x64.tar.gz
mv /data/jdk1.8.0_192 /usr/lib/
```

### 卸载 java 8

```
sudo apt-get remove --autoremove oracle-java8-installer oracle-java10-installer
```

### 环境变量

[Environment Variables](https://help.ubuntu.com/community/EnvironmentVariables)， use `env` to check all current variables.

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

修改 全局的 profile文件 `vi /etc/profile`，执行 `source /etc/profile`可立即生效。 查看profile的具体内容可知：实际执行的是 `/etc/profile.d/`目录下的所以.sh文件。

在/etc/profile文件中添加变量【对所有用户生效（永久的）】  
用vim在文件/etc/profile文件中增加变量，该变量将会对Linux下所有用户有效，并且是“永久的”。  
要让刚才的修改马上生效，需要执行以下代码  
`source /etc/profile`


  
方法二：  
在用户目录下的.bash_profile文件中增加变量【对单一用户生效（永久的）】  
用vim在用户目录下的.bash_profile文件中增加变量，改变量仅会对当前用户有效，并且是“永久的”。  
要让刚才的修改马上生效，需要在用户目录下执行以下代码  
`source .bash_profile `

方法三：  
直接运行export命令定义变量【只对当前shell（BASH）有效（临时的）】  
在shell的命令行下直接使用[export  变量名=变量值]定义变量，该变量只在当前的shell（BASH）或其子shell（BASH）下是有效的，shell关闭了，变量也就失效了，再打开新shell时就没有这个变量，需要使用的话还需要重新定义。  

### ulimit用法 

[ulimit 用法](http://man.linuxde.net/ulimit)

```
-a：显示目前资源限制的设定；
-c <core文件上限>：设定core文件的最大值，单位为区块；
-d <数据节区大小>：程序数据节区的最大值，单位为KB；
-f <文件大小>：shell所能建立的最大文件，单位为区块；
-H：设定资源的硬性限制，也就是管理员所设下的限制；
-m <内存大小>：指定可使用内存的上限，单位为KB；
-n <文件数目>：指定同一时间最多可开启的文件数；
-p <缓冲区大小>：指定管道缓冲区的大小，单位512字节；
-s <堆叠大小>：指定堆叠的上限，单位为KB；
-S：设定资源的弹性限制；
-t <CPU时间>：指定CPU使用时间的上限，单位为秒；
-u <程序数目>：用户最多可开启的程序数目；
-v <虚拟内存大小>：指定可使用的虚拟内存上限，单位为KB。


[root@localhost ~]# ulimit -a
core file size          (blocks, -c) 0           #core文件的最大值为100 blocks。
data seg size           (kbytes, -d) unlimited   #进程的数据段可以任意大。
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited   #文件可以任意大。
pending signals                 (-i) 98304       #最多有98304个待处理的信号。
max locked memory       (kbytes, -l) 32          #一个任务锁住的物理内存的最大值为32KB。
max memory size         (kbytes, -m) unlimited   #一个任务的常驻物理内存的最大值。
open files                      (-n) 1024        #一个任务最多可以同时打开1024的文件。
pipe size            (512 bytes, -p) 8           #管道的最大空间为4096字节。
POSIX message queues     (bytes, -q) 819200      #POSIX的消息队列的最大值为819200字节。
real-time priority              (-r) 0
stack size              (kbytes, -s) 10240       #进程的栈的最大值为10240字节。
cpu time               (seconds, -t) unlimited   #进程使用的CPU时间。
max user processes              (-u) 98304       #当前用户同时打开的进程（包括线程）的最大个数为98304。
virtual memory          (kbytes, -v) unlimited   #没有限制进程的最大地址空间。
file locks                      (-x) unlimited   #所能锁住的文件的最大个数没有限制。
```

### lsof 打开文件过多

[检查以及处理](https://blog.csdn.net/roy_70/article/details/78423880)

```
#1. 查看进程 
ps aux |grep java

#2. 查看进程打开的文件
lsof -p pid

#3. 修改ulimit条件
```


## Shell commands

[LDP](http://www.tldp.org/LDP/abs/html/abs-guide.html)

[Comparison Operators](http://www.tldp.org/LDP/abs/html/abs-guide.html#COMPARISON-OPS)</br>
[Special Characters](http://www.tldp.org/LDP/abs/html/abs-guide.html#SPECIAL-CHARS)

任何场景下，使用
```
man command
``` 
查看命令详情 

### LDP
The Linux Documentation Project [LDP](http://www.tldp.org/LDP/abs/html/abs-guide.html)

### add text to file head

[How to insert a text at the beginning of a file?](https://stackoverflow.com/a/9533736/1087122)

sed can operate on an address:
```
$ sed -i '1s/^/<added text> /' file
```
What is this magical 1s you see on every answer here? [Line addressing](https://www.gnu.org/software/sed/manual/html_node/Addresses.html)!.

Want to add <added text> on the first 10 lines?
```
$ sed -i '1,10s/^/<added text> /' file
```
Or you can use [Command Grouping](http://www.gnu.org/software/bash/manual/bash.html#Command-Grouping):
```
$ { echo -n '<added text> '; cat file; } >file.new
$ mv file{.new,}
```

### 创建大文本文件

[How to create a large file in UNIX?](https://unix.stackexchange.com/questions/269180/how-to-create-a-large-file-in-unix)
[Quickly create a large file on a Linux system](https://stackoverflow.com/questions/257844/quickly-create-a-large-file-on-a-linux-system)
[How To Quickly Generate A Large File On The Command Line](https://skorks.com/2010/03/how-to-quickly-generate-a-large-file-on-the-command-line-with-linux/)

```
yes "Some text" | head -n 100000 > large-file
```

### sed 查找替换冒号

sed如何转义单引号和双引号：如果外面是双引号，里面的双引号就要转义；写成\"，单引号则不用转义，如果外面是单引号就反过来。

```
# 替换 GRUB_CMDLINE_LINUX="" 为 GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0" 
sed -i "s/GRUB_CMDLINE_LINUX=\"\"/GRUB_CMDLINE_LINUX=\"net.ifnames=0 biosdevname=0\"/g" /etc/default/grub

```


### 使用jq格式化json

需要先安装 `sudo apt install -u jq`

```
cat config/devices.txt |jq . 

{
    "brand": "360",
    "displayed": true,
    "idealName": " 10.35.16.20 [360 1703-M01] _1-5 [864848031310516 23] ",
    "imei": "864848031310516",
    "keyID": "10.35.16.20__1-5",
    "model": "1703-M01",
    "online": true,
    "position": {
      "column": 7,
      "row": 2
}
```

### 收集进程的线程详细信息

```
#!/bin/bash
controller=$(ps -aux|grep java |grep -F "$1"|grep -vF grep|awk '{printf $0}' |awk '{print $2}')
top -b -n 1 -H -p $controller |grep Threads

# -b  :Batch-mode operation
#            Starts top in 'Batch' mode, which could be useful for sending output from top to other programs or to a file.  In this mode, top  will
#            not accept input and runs until the iterations limit you've set with the '-n' command-line option or until killed.
# -n  :Number-of-iterations limit as:  -n number

```

### 查看进程的线程信息
```
top -H -p <pid>
root@test:/root# top -H -p 12091
top - 10:33:34 up 12:58,  2 users,  load average: 14.39, 15.31, 16.61
Threads: 106 total,   4 running, 102 sleeping,   0 stopped,   0 zombie
%Cpu(s): 68.4 us,  0.4 sy,  0.0 ni,  6.3 id, 24.7 wa,  0.0 hi,  0.2 si,  0.0 st
KiB Mem : 16285716 total,   891832 free, 10665560 used,  4728324 buff/cache
KiB Swap:  2097148 total,  2097148 free,        0 used.  4740208 avail Mem

进▒ USER      PR  NI    VIRT    RES    SHR ▒ %CPU %MEM     TIME+ COMMAND
13503 root      20   0 14.222g 9.102g  26088 R 99.9 58.6 445:00.85 java
24782 root      20   0 14.222g 9.102g  26088 R 99.9 58.6 315:46.25 java
25118 root      20   0 14.222g 9.102g  26088 R 99.9 58.6 299:22.98 java
26443 root      20   0 14.222g 9.102g  26088 R 99.9 58.6 173:56.59 java
12104 root      20   0 14.222g 9.102g  26088 S 62.2 58.6 125:17.54 java
16155 root      20   0 14.222g 9.102g  26088 S  2.4 58.6   0:02.87 java
16154 root      20   0 14.222g 9.102g  26088 D  1.2 58.6   0:02.53 java
12117 root      20   0 14.222g 9.102g  26088 S  0.4 58.6   1:25.10 java
16136 root      20   0 14.222g 9.102g  26088 D  0.4 58.6   0:08.15 java
16157 root      20   0 14.222g 9.102g  26088 S  0.4 58.6   0:03.89 java
16285 root      20   0 14.222g 9.102g  26088 S  0.4 58.6   0:05.48 java
12091 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:00.00 java
12093 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:02.23 java
12094 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:24.36 java
12095 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:24.78 java
12096 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:23.79 java
12097 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:23.89 java
12098 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:24.28 java
12099 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:24.99 java
12100 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:24.57 java
12101 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:23.62 java
12102 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:33.86 java
12103 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:36.90 java
12105 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   1:14.74 java
12106 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:00.96 java
12107 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:00.87 java
12108 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:00.05 java
12109 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:00.00 java
12110 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:37.36 java
12111 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:39.42 java
12112 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:42.24 java
12113 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:15.51 java
12114 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:00.00 java
12115 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:12.55 java
12116 root      20   0 14.222g 9.102g  26088 S  0.0 58.6  12:19.01 java
12118 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   6:45.93 java
12119 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:00.14 java
12120 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:04.27 java
12121 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:00.00 java
12150 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:00.00 java

```
### /dev/null 2>&1含义

[What is /dev/null 2>&1?](https://stackoverflow.com/q/10508843/1087122)</br>
[2>&1>/dev/null](https://blog.csdn.net/zhongqi2513/article/details/78613768)</br>
[shell中命令>/dev/null 2>&1含义](http://tang9140.iteye.com/blog/2254020)</br>

标准输出被重定向到空的设备文件，即不显示，标准错误输出被重定向到标准输出，而标准输出已被重定向到空的设备文件，所以标准错误输出也不显示。

标准输入0    从键盘获得输入 /proc/self/fd/0 </br>
标准输出1    输出到屏幕（即控制台） /proc/self/fd/1 </br>
错误输出2    输出到屏幕（即控制台） /proc/self/fd/2 </br>

### split & cat使用


```
-b：值为每一输出档案的大小，单位为 byte。
-C：每一输出档中，单行的最大 byte 数。
-d：使用数字作为后缀。
-l：值为每一输出档的列数大小。

split -b 102400k all.log
~# ls
all.log   xaa  xab  xac  xad

cat xaa xab xac xad  >back.log
~# md5sum back.log
ab9a814d0d7f412555c48689d6037050  back.log
~# md5sum all.log
ab9a814d0d7f412555c48689d6037050  all.log
```


### 使用zsh

1. 安装 `sudo apt-get install zsh` 
2. 配置默认的sh `chsh -s /bin/zsh` 或 `chsh -s 'which zsh'`，切换为旧bash `chsh -s /bin/bash`
3. 安装 oh-my-zsh [github repo](https://github.com/robbyrussell/oh-my-zsh)
4. 手动更新： `upgrade_oh_my_zsh`

```
# curl方式
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

# wget 方式
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"

```

### 端口占用
利用netstat 或fuser 命令查找
```
netstat -ano |grep 9988
# -k 参数可以kill相应的进程
fuser 9988/tcp
##java可利用ManagementFactory.getRuntimeMXBean().getName()获取进程
```

0x0. 启动terminal 快捷键  
```
Ctrl+Alt+T
```

### 0x1. 等待
	
    sleep 1 #睡眠1秒 默认
	sleep 1s #睡眠1秒
	sleep 1m #睡眠1分
	sleep 1h #睡眠1小时
	

### 0x2. 日期
```
#%s表示当前时间的秒数，而%N表示当前时间的纳秒部分，即1秒以下的那部分
joechin@ubuntu:~$ date +%Y%m%d%H%M%S%N
20170824115956519750498
joechin@ubuntu:~$ date +%Y%m%d%H%M%S
20170824115957

```

### 0x3. 判断文件夹、进程是否存在

```
#!/bin/bash

# Create dir logs if it doesn't exist
if [ ! -d "./_logs" ]
then
    mkdir ./_logs
fi

# 判断是否存在文件 
# ls所有以.txt为后缀的文件，如果不存在，将标准错误重定向到标准输出，
# 2>&1 的意思就是将标准错误也输出到标准输出当中。
# 重定向中 0-标准输出，1-标准输出，2-标准错误，而No such file or directory是一个标准错误。
if ls *.txt >/dev/null 2>&1; then
　　echo "file exist."
else
    echo "no txt file"
fi

controller=$(ps -aux|grep -F "ycdh"|grep -vF grep|awk '{printf $0}' |awk '{print $2}')

# 中括号前后必须有空格，否则报 [: missing `]' 错误
# 需要两个中括号，因为当变量为空时，会报 "[: =: unary operator expected" 错误
if [[ $controller -ne 0 ]] ; then
   echo "try to kill $controller"
   kill -9 $controller
else
  echo "ycdh not running."
fi

```

```
#!/bin/sh  
  
myPath="/var/log/httpd/"  
myFile="/var /log/httpd/access.log"  
  
#这里的-x 参数判断$myPath是否存在并且是否具有可执行权限  
if [ ! -x "$myPath"]; then  
　　mkdir "$myPath"  
fi  

#这里的-d 参数判断$myPath是否存在  
if [ ! -d "$myPath"]; then  
　　mkdir "$myPath"  
fi  
  
#这里的-f参数判断$myFile是否存在  
if [ ! -f "$myFile" ]; then  
　　touch "$myFile"  
fi  
  
#其他参数还有-n,-n是判断一个变量是否是否有值  
if [ ! -n "$myVar" ]; then  
　　echo "$myVar is empty"  
　　exit 0  
fi  
  
#两个变量判断是否相等  
if [ "$var1" = "$var2" ]; then  
　　echo '$var1 eq $var2'  
else  
　　echo '$var1 not eq $var2'  
fi  


-e filename  如果 filename存在，则为真  [ -e /var/log/syslog ]
-d filename  如果 filename为目录，则为真  [ -d /tmp/mydir ]
-f filename  如果 filename为常规文件，则为真  [ -f /usr/bin/grep ]
-L filename  如果 filename为符号链接，则为真  [ -L /usr/bin/grep ]
-r filename  如果 filename可读，则为真  [ -r /var/log/syslog ]
-w filename  如果 filename可写，则为真  [ -w /var/mytmp.txt ]
-x filename  如果 filename可执行，则为真  [ -L /usr/bin/grep ]
filename1-nt filename2  如果 filename1比 filename2新，则为真 
 [ /tmp/install/etc/services -nt /etc/services ]
filename1-ot filename2  如果 filename1比 filename2旧，则为真  
 [ /boot/bzImage -ot arch/i386/boot/bzImage ]

```

### 0x4. 线程控制批量执行
- 循环体的命令用&符号放入后台运行实现多线程效果
- 利用有名管道特性实现线程控制

```
#!/bin/bash
# must /bin/bash add method not work while using /bin/sh

# http://www.cnblogs.com/chenjiahe/p/6268853.html
#定义脚本运行的开始时间
start_time=`date +%s`
# need to make sure root user can remote login
passwd='passwordForRoot'
#创建有名管道
[ -e /tmp/fd1 ] || mkfifo /tmp/fd1 
#创建文件描述符，以可读（<）可写（>）的方式关联管道文件，文件描述符9就有了有名管道文件的所有特性
#has to be number?
exec 9<>/tmp/fd1                   
#关联后的文件描述符拥有管道文件的所有特性,所以这时候管道文件可以删除，留下文件描述符即可
rm -rf /tmp/fd1                    
for ((i=1;i<=5;i++))
do
#&9代表引用文件描述符9，这条命令代表往管道里面放入了一个"令牌"
    echo >&9                   
done

for ip in 10.34.16.25 10.34.16.27 10.34.7.200
do
#代表从管道中读取一个令牌
read -u9
{
	if ping -c 1 -W 5 $ip > /dev/null;then
			#http://www.cnblogs.com/autopenguin/p/5918717.html
			/usr/bin/expect <<-EOF
			set timeout -1

			spawn ssh root@$ip "cd /data/itestin-m/; bash /data/itestin-m/dh.sh"
			expect {
					"*yes/no" { send "yes\r"; exp_continue }
					"*password:" { send "$passwd\r" }
			}
			expect eof

			EOF
			echo -e "~~~~good $ip~~~\n"
	else
			echo -e "=====can't connect@@@@$ip==\n\n"
	fi
	#代表命令执行到最后，把令牌放回管道
	echo >&9   
}&                           
done
wait

stop_time=`date +%s`  #定义脚本运行的结束时间
 
echo "TIME:`expr $stop_time - $start_time`"
#关闭文件描述符的读，必须分开写
exec 9<&-
#关闭文件描述符的写           
exec 9>&-

```

### 0x5. 过滤logcat输出并保存到文件
```shell
#使用 --line-buffered 参数 https://stackoverflow.com/a/18146279/1087122
adb logcat -v time | grep --line-buffered UIA > log.log
```

### 0x6. 查看端口占有
```
# 查看已经连接的服务端口（ESTABLISHED）
netstat -a

#查看所有的服务端口（LISTEN，ESTABLISHED）
netstat -ap

# 查看具体哪个进程占有对应的端口，端口号为特指
lsof -i:9988
```

### 0x7. apt-get安装卸载
```
# 查看已安装应用 dpkg -l配合grep
dpkg -l |grep 'deja-dup'

# 卸载应用 -q for quiet， 可以通过 man apt-get 搜索 -qq查看更多信息
apt-get -q remove deja-dup
```


### 0x8. Shell接受参数 $@

[linux中shell变量$#,$@,$0,$1,$2的含义解释](http://www.cnblogs.com/fhefh/archive/2011/04/15/2017613.html)

- $$  Shell本身的PID（ProcessID） 
- $! Shell最后运行的后台Process的PID 
- $? 最后运行的命令的结束代码（返回值） 
- $- 使用Set命令设定的Flag一览 
- $* 所有参数列表。如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。 
- $@ 所有参数列表。如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。 
- $# 添加到Shell的参数个数 
- $0 Shell本身的文件名 
- $1～$n  添加到Shell的各参数值。$1是第1参数、$2是第2参数…。 

接收来自命令行传入的参数，第一个参数用$1表示，第二个参数$2表示，。。。以此类推。注意：$0表示脚本文件名。另外一个在shell编程中经常用到 的是“$@”这个代表所有的参数，。你可以用一个循环来遍历这个参数。如果用java来类比的话，可以把$@看作是man函数中定义的那个数组
```
#!/bin/bash
for args in $@
  do
  	echo $args
  done
```

### 0x9. Shell自动登录expect
```
# 需要安装expect
#!/usr/bin/expect
spawn ssh root@192.168.22.194
expect "*password:"
send "123\r"
expect "*#"
interact
```

### 0xa. for循环读取文件作为变量
```
for line in `cat filename`
do
  echo $line
done
```
可配合上面的使用，变量单独存在文件中

### 0xb. 查找包含某字符的所有文件
```
grep -rn "word to find" *
# * 表示当前目录所有文件，支持通配符
# -r 递归查找
# -n 显示行号
# -R 查找当前目录的所有子目录， 深递归
# -i 忽略大小写
```
### 0xc. 显示n到m行的文件内容，或n到结尾的内容
```
sed -n '100, 150p' path/to/filename

sed '100,$!d' path/to/filename
#或
sed '100,$!p' path/to/filename
```
### 0xd. shell中调用另外一个shell脚本

[fork exec source](http://blog.csdn.net/l_nan/article/details/21874679)

fork方式为直接执行，直接写被调用脚本的路径即可。在子命令执行完后再执行父级命令。子级的环境变量不会影响到父级。

exec path/to/call/script.sh 执行子级的命令后，不再执行父级命令

source path/to/call/sciprt.sh 执行子级命令后继续执行父级命令，同时子级设置的环境变量会影响到父级的环境变量。

### 0xe. awk基本使用

[详解](http://www.cnblogs.com/ggjucheng/archive/2013/01/13/2858470.html)

读入有'\n'换行符分割的一条记录，然后将记录按指定的域分隔符划分域，填充域，$0则表示所有域,$1表示第一个域,$n表示第n个域。默认域分隔符是"空白键" 或 "[tab]键"

可以使用 -F 指定分割符合

```
awk -F: '{print $7}' /etc/passwd
# root:x:0:0:root:/root:/bin/bash
/bin/bash
# 以:为分隔符，打印第7个，所有输出 /bin/bash

```

### 0xf. du -h 查看文件具体大小

[Linux下查看文件和文件夹大小](https://www.cnblogs.com/benio/archive/2010/10/13/1849946.html)

df命令可以显示目前所有文件系统的可用空间及使用情形

du：查询文件或文件夹的磁盘使用空间
```
test@pp:/data/itestin-m# du -h --max-depth=1 .
4.0K	./auto_update
386M	./tools
861M	./data
1.6G	./logs
18G	./download
1.3G	./temp
4.0K	./user
340K	./config
91M	./ycdh_lib
4.0K	./script
4.0K	./tmpp
4.0K	./report
6.2G	./_logs
29G	.
```

### grep 之后只显示部分行数 grep '' | head -200

### 处理 Binary file xxx.log matches
grep认为这是二进制文件，解决方案：grep -a
```
cat log-20171218074617.txt |grep -a c33a1478fa479153124f938541022398
```

### dig查看域名解析

```
dig qq.com +nostats +nocomments +nocmd

; <<>> DiG 9.9.7-P3 <<>> qq.com +nostats +nocomments +nocmd
;; global options: +cmd
;qq.com.				IN	A
qq.com.			600	IN	A	125.39.240.113
qq.com.			600	IN	A	61.135.157.156
qq.com.			17879	IN	NS	ns4.qq.com.
qq.com.			17879	IN	NS	ns2.qq.com.
qq.com.			17879	IN	NS	ns1.qq.com.
qq.com.			17879	IN	NS	ns3.qq.com.
ns1.qq.com.		3706	IN	A	101.226.68.138
ns1.qq.com.		3706	IN	A	14.17.19.139
ns2.qq.com.		18704	IN	A	101.227.169.106
ns3.qq.com.		106736	IN	A	182.140.177.149
ns3.qq.com.		106736	IN	A	182.140.167.157
ns4.qq.com.		106736	IN	A	123.151.178.115
ns4.qq.com.		106736	IN	A	125.39.247.247
ns4.qq.com.		106736	IN	A	184.105.206.124
ns4.qq.com.		106736	IN	A	203.205.144.156
```

### 统计当前文件夹下文件的个数

```
#1、 统计当前文件夹下文件的个数
ls -l |grep "^-"|wc -l

#2、 统计当前文件夹下目录的个数
ls -l |grep "^d"|wc -l

#3、统计当前文件夹下文件的个数，包括子文件夹里的 
ls -lR|grep "^-"|wc -l

#4、统计文件夹下目录的个数，包括子文件夹里的
ls -lR|grep "^d"|wc -l
```

### Argument list too long， use xargs or exec
传递给ls, rm等命令的参数过长。
```
#递归查找所有pdf文件并删除
find . -name "*.pdf" -print0 | xargs -0 rm

# xargs 命令是给其他命令传递参数的一个过滤器，-i会将xargs的内容赋值给{}。
find . -name "*.jpeg" | xargs -i rm {}

# 非递归，指定查找深度
find . -maxdepth 1 -name "*.jpeg" -print0 | xargs -0 rm

## 问题：要拷贝test文件夹下以jpg结尾的文件到train目录。
#命令1为：
find test/ -name "*.jpg" | xargs -i cp {} train

#命令2为：
find test/ -name "*.jpg" -exec cp {} train \;

```

### linux修改文件所有者和文件所在组

[give user write access to folder](https://askubuntu.com/questions/402980/give-user-write-access-to-folder)

```
# change group
chgrp -R  用户名    文件名 
# need to run with root 
chown -R 用户名   文件名  -R

#     r： 4（读权限）
#     w： 2（写权限）
#     x： 1（执行权限）
# 将这些数字相加，就得到每一组的权限值，例如
-rw-r--r--  1 myy groupa 0 Sep 26 06:07 filed
第一组（user）：rw- = 4+2+0 = 6
第二组（group）：r-- = 4+0+0 = 4
第三组（others）：r-- = 4+0+0 = 4

# example: This will make the user & group testuser the owner of the file of /var/www/test/public_html
sudo chown -R testuser:testuser /var/www/test/public_html

```

### zsh 安装使用

[zsh安装使用](https://blog.csdn.net/gatieme/article/details/52741221)
```
sudo apt-get install zsh
#进入zsh
zsh

#安装oh-my-zsh
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"1

```

### 命令行直接打开指定文件目录
```
# 安装 
sudo apt install libgnome2-bin 

#使用 gnome-open path
gnome-open path/to/destination
```

### 命令行复制粘贴 Ctrl + Shift + C / V 