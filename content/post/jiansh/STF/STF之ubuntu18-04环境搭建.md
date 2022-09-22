+++
title = "STF之ubuntu18-04环境搭建"
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

[link to JianShu](https://www.jianshu.com/p/cc975f72d2d4)


参考：
[stf on ubuntu](https://testerhome.com/articles/17696)
[[centos7][stf] 环境搭建](https://testerhome.com/topics/11055)

一、安装npm
```
# https://github.com/nvm-sh/nvm
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash
# 主动生效
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

二、安装nrm
```
# https://github.com/Pana/nrm
npm install -g nrm
#配置私有源
nrm add name url
# use it
nrm use name
```

三、安装依赖应用
依赖应用：`rethinkdb graphicsmagick zeromq protobuf yasm pkg-config`

1. [rethinkdb](https://www.rethinkdb.com/docs/install/ubuntu/)
```
source /etc/lsb-release && echo "deb http://download.rethinkdb.com/apt $DISTRIB_CODENAME main" | sudo tee /etc/apt/sources.list.d/rethinkdb.list
wget -qO- https://download.rethinkdb.com/apt/pubkey.gpg | sudo apt-key add -
sudo apt-get update
sudo apt-get install rethinkdb
```
官方没有提供18.04系统下的正式版本。参考[issues 6663](https://github.com/rethinkdb/rethinkdb/issues/6663)， 使用[srh v2.3.6.srh.1](https://github.com/srh/rethinkdb/releases/tag/v2.3.6.srh.1)提供的版本：
```
wget https://github.com/srh/rethinkdb/releases/download/v2.3.6.srh.1/rethinkdb_2.3.6.srh.1.0bionic_amd64.deb
# install 
sudo dpkg -i rethinkdb_2.3.6.srh.1.0bionic_amd64.deb
# check version
rethinkdb --version
rethinkdb 2.3.6.srh.1~0bionic (CLANG 6.0.0 (tags/RELEASE_600/final))
```

2. 安装GraphicsMagick
下载
```
wget https://sourceforge.net/projects/graphicsmagick/files/graphicsmagick/1.3.33/GraphicsMagick-1.3.33.tar.gz
tar -zxvf GraphicsMagick-1.3.33.tar.gz
cd GraphicsMagick-1.3.33.tar.gz
./configure
```
编译时提示：`no acceptable C compiler found in $PATH` 安装对应的编译器`sudo apt install build-essential`./

3. 安装zeromq

从官方github进行下载 [zeromq releases](https://github.com/zeromq/libzmq/releases)zeromq-4.3.2.tar.gz复制到对应到目录

```
tar -zxvf zeromq-4.1.4.tar.gz
cd zeromq-4.1.4
./configure
make
sudo make install
sudo ldconfig
```

4. 安装protobuf yasm pkg-config

- protobuf 从[protocolbuffers releases](https://github.com/protocolbuffers/protobuf/releases)下载[protoc-3.9.0-linux-x86_64.zip](https://github.com/protocolbuffers/protobuf/releases/download/v3.9.0/protoc-3.9.0-linux-x86_64.zip)版本，直接解压到目录，然后添加环境变量即可

- yasm  pkg-config 使用 `sudo apt install -y yasm pkg-config`安装即可

5.  安装adb工具，识别android设备
[platform-tools](https://developer.android.com/studio/releases/platform-tools) 中有下载连接[https://dl.google.com/android/repository/platform-tools-latest-linux.zip](https://dl.google.com/android/repository/platform-tools-latest-linux.zip)
解压后配置环境变量即可。

[platfort-tools-latest-windows](https://dl.google.com/android/repository/platform-tools-latest-windows.zip)

### 内网环境问题解决
因为是内网访问，机器接在了一台路由器后面，默认的网关无法解析内网域名。 
[Ubuntu 18.04 LTS正确设置DNS服务器](https://www.feiqy.com/ubuntu-dns/)

关闭systemd-resolvd服务
```
systemctl stop systemd-resolvd
systemctl disable systemd-resolvd
```
这样对/etc/resolv.conf做出的修改都能保存下来。

修改systemd-resolv的设置
打开` /etc/systemd/resolved.conf`，修改为
```
[Resolve]
DNS=1.1.1.1 1.0.0.1 10.8.8.8
#FallbackDNS=
#Domains=
LLMNR=no
#MulticastDNS=no
#DNSSEC=no
#Cache=yes
#DNSStubListener=yes
```
再重新启动访问 `sudo systemctl start systemd-resolved.service`

最后，如果`npm install`提示有错误，多试几次吧，github访问不稳定。
