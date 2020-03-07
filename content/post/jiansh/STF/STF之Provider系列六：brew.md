+++
title = "STF之Provider系列六：brew"
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



[link on JianShu](https://www.jianshu.com/p/1a52b09203b9)

[STF之Provider系列五](https://www.jianshu.com/p/a358dbb7c05d)支持脚本自动部署。目前国内到网络环境导致使用brew install 各种依赖应用时：1）耗时；2）不稳定。

[STF之Docker for Mac失败](https://www.jianshu.com/p/285868401ca1)得出结论，Mac环境下到Docker无法挂载USB设备。

需要利用brew的本地化安装来提供环境部署——

[How to install a local file in Homebrew](http://mygeekdaddy.net/2014/12/05/how-to-install-a-local-file-in-homebrew/)  
[How to use Homebrew to install local archive](https://apple.stackexchange.com/a/328946)

```
# download your archive
wget https://github.com/zeromq/libzmq/releases/download/v4.3.2/zeromq-4.3.2.tar.gz

# move the archive to your brew --cache
mv zeromq-4.3.2.tar.gz $(brew --cache)

# then install the archive as normal
brew install zeromq
```
使用`brew cat zeromq`查看zeromq的下载地址 

掌握brew的[基础知识](https://brew.sh/)，[使用手册](https://docs.brew.sh/Manpage)，[常见问题](https://docs.brew.sh/FAQ)

基本思路是将STF依赖的各个包`graphicsmagick zeromq protobuf yasm pkg-config`，以及每个包本身的依赖都下载到本地，然后利用本地化安装进行安装。从而降低对网络对依赖

使用`brew deps -n  Formula` 可以查看对应包到依赖，通过 `brew cat foumular`可以查看到可下载到url地址是什么。
```
# 对应provider来说，不需要rethinkdb——
# wget graphicsmagick zeromq protobuf yasm pkg-config
➜  ~ brew deps -n wget
gettext
libunistring
libidn2
openssl

https://ftp.gnu.org/gnu/gettext/gettext-0.20.1.tar.xz
https://ftp.gnu.org/gnu/libunistring/libunistring-0.9.10.tar.xz 
https://ftp.gnu.org/gnu/libidn/libidn2-2.2.0.tar.gz
https://www.openssl.org/source/openssl-1.0.2s.tar.gz 
https://ftp.gnu.org/gnu/wget/wget-1.20.3.tar.gz 

➜  ~ brew deps -n graphicsmagick
libpng
freetype
jpeg
jasper
libtiff
libtool
little-cms2
webp

https://downloads.sourceforge.net/libpng/libpng-1.6.37.tar.xz
https://downloads.sourceforge.net/project/freetype/freetype2/2.10.1/freetype-2.10.1.tar.xz
https://www.ijg.org/files/jpegsrc.v9c.tar.gz
https://github.com/mdadams/jasper/archive/version-2.0.16.tar.gz 
https://download.osgeo.org/libtiff/tiff-4.0.10.tar.gz 
https://ftp.gnu.org/gnu/libtool/libtool-2.4.6.tar.xz
https://downloads.sourceforge.net/project/lcms/lcms/2.9/lcms2-2.9.tar.gz
https://storage.googleapis.com/downloads.webmproject.org/releases/webp/libwebp-1.0.3.tar.gz

https://downloads.sourceforge.net/project/graphicsmagick/graphicsmagick/1.3.33/GraphicsMagick-1.3.33.tar.xz
https://github.com/zeromq/libzmq/releases/download/v4.3.2/zeromq-4.3.2.tar.gz
https://www.tortall.net/projects/yasm/releases/yasm-1.3.0.tar.gz
https://pkgconfig.freedesktop.org/releases/pkg-config-0.29.2.tar.gz
```

其中protobuf比较特殊，cat出来到内容类似——
```
brew cat protobuf

class Protobuf < Formula
  desc "Protocol buffers (Google's data interchange format)"
  homepage "https://github.com/protocolbuffers/protobuf/"
  url "https://github.com/protocolbuffers/protobuf.git",
      :tag      => "v3.7.1",
      :revision => "6973c3a5041636c1d8dc5f7f6c8c1f3c15bc63d6"
  head "https://github.com/protocolbuffers/protobuf.git"

# 如果指定3.6版本，可以看到下载tar包的url
brew cat protobuf@3.6
class ProtobufAT36 < Formula
  desc "Protocol buffers (Google's data interchange format)"
  homepage "https://github.com/protocolbuffers/protobuf/"
  url "https://github.com/protocolbuffers/protobuf/archive/v3.6.1.3.tar.gz"
  sha256 "73fdad358857e120fd0fa19e071a96e15c0f23bb25f85d3f7009abfd4f264a2a"

  bottle do
    cellar :any
    sha256 "110cb92b662880877a8304713fc137353cd464971a97198dc398789405ded66c" => :mojave
    sha256 "2c85fa9006e7bd119e32616aee287ea598189aa031a8b39aeae10380b68ab3ae" => :high_sierra
    sha256 "1d93421add439a0a67a8ff161270de1ea0c7a0e62fb18ababce16bbf267fab0d" => :sierra
  end
```

[gist for install protobuf](https://gist.github.com/wagnerfonseca-luizalabs/becedc8c5aeb6a8a60f8d72bbf140978)

```
$ wget https://github.com/google/protobuf/releases/download/v3.4.1/protobuf-cpp-3.4.1.tar.gz
$ tar -zxvf protobuf-cpp-3.4.1.tar.gz
$ sudo mv protobuf-3.4.1 /usr/local/bin
$ cd /usr/local/bin/protobuf-3.4.1
$ ./configure
$ make 
$ make install
$ protoc --version
```
