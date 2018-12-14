+++
title = "Go Tech"
description = "study go and python and etc."
tags = [
    "go",
    "python"
]
date = "2018-01-03"
topics = [
    "go",
    "python"
]
toc = true
+++

Go release 1.9 version on 24 August 2017. I'm the [31,365](https://github.com/golang/go/stargazers) people to start it on github and just get started to learn it :). 

Let's Go. 

    go tool tour

[Go 1.9 is released](https://blog.golang.org/go1.9). [**Here**](https://blog.golang.org/index) are all the blogs about go.

[Go: Ten years and climbing](https://commandcenter.blogspot.co.uk/2017/09/go-ten-years-and-climbing.html)
[Go十年](http://tonybai.com/2017/09/24/go-ten-years-and-climbing/)
[BIG: Blockchain In Go](https://jeiwan.cc/tags/blockchain/)

<!--more-->

VS环境设置：
[Go tools that the Go extension depends on](https://github.com/Microsoft/vscode-go/wiki/Go-tools-that-the-Go-extension-depends-on)

Ctrl + b: 显示/隐藏侧边栏
Ctrl + j: 显示/隐藏Panel控制栏

### update project 

```
# Add Hugo and its package dependencies to your go src directory.
go get -v github.com/gohugoio/hugo
```

$GOPATH 目录约定有三个子目录：

- src 存放源代码（比如：.go .c .h .s等）
- pkg 编译后生成的文件（比如：.a）
- bin 编译后生成的可执行文件（为了方便，可以把此目录加入到 $PATH 变量中，如果有多个gopath，那么使用${GOPATH//://bin:}/bin添加所有的bin目录）

```
$ go env
set GOARCH=amd64
set GOBIN=
set GOEXE=.exe
set GOHOSTARCH=amd64
set GOHOSTOS=windows
set GOOS=windows
set GOPATH=F:\Tools\gopath\
set GORACE=
set GOROOT=F:\Tools\go
set GOTOOLDIR=F:\Tools\go\pkg\tool\windows_amd64
set GCCGO=gccgo
set CC=gcc
set GOGCCFLAGS=-m64 -mthreads -fmessage-length=0 -fdebug-prefix-map=G:\TempFolder\userTemp\go-build565356171=/tmp/go-build -gno-record-gcc-switches
set CXX=g++
set CGO_ENABLED=1
set CGO_CFLAGS=-g -O2
set CGO_CPPFLAGS=
set CGO_CXXFLAGS=-g -O2
set CGO_FFLAGS=-g -O2
set CGO_LDFLAGS=-g -O2
set PKG_CONFIG=pkg-config

```


### go get golang.org/x 包失败解决方法

[方案](http://blog.csdn.net/alexwoo0501/article/details/73409917)由于限制问题，国内使用 go get 安装 golang 官方包可能会失败

```
go get -v golang.org/x/tour/pic
Fetching https://golang.org/x/tour/pic?go-get=1
https fetch failed: Get https://golang.org/x/tour/pic?go-get=1: dial tcp 216.239.37.1:443: i/o timeout
package golang.org/x/tour/pic: unrecognized import path "golang.org/x/tour/pic" (https fetch: Get https://golang.org/x/tour/pic?go-get=1: dial tcp 216.239.37.1:443: i/o timeout)

```

golang 在 github 上建立了一个**[镜像库](https://github.com/golang)**，如 https://github.com/golang/net 即是 https://golang.org/x/net 的镜像库

获取 golang.org/x/net 包，其实只需要以下步骤：
```
mkdir -p $GOPATH/src/golang.org/x
cd $GOPATH/src/golang.org/x
git clone https://github.com/golang/net.git

## example
➜  src mkdir -p golang.org/x
➜  src cd golang.org/x
➜  x git clone https://github.com/golang/tour.git
Cloning into 'tour'...
remote: Counting objects: 2092, done.
remote: Total 2092 (delta 0), reused 0 (delta 0), pack-reused 2092
Receiving objects: 100% (2092/2092), 895.19 KiB | 133.00 KiB/s, done.
Resolving deltas: 100% (1329/1329), done.
## now you can import "golang.org/x/tour/pic" 
```

其它 golang.org/x 下的包获取皆可使用该方法

或者使用软连接的方式代替
```
git clone https://github.com/golang/net.git $GOPATH/src/github.com/golang/net

git clone https://github.com/golang/sys.git $GOPATH/src/github.com/golang/sys

git clone https://github.com/golang/tools.git $GOPATH/src/github.com/golang/tools

ln -s $GOPATH/src/github.com/golang $GOPATH/src/golang.org/x1

```


<img src="data:image/png;base64, iVBORw0KGgoAAAANSUhEUgAAAQAAAAEACAIAAADTED8xAAAE8UlEQVR4nOzX36cQBgOA4XOONjMzMzMzMzMzM0mSJEmSJEmSJEmSJEmSJEmSJEmSJEmSJEmSJEmSJEmSJEmSJEmSJP34/ovvjPe5ey7f23fEwMCXwcGBr74aGBwc+PRpgDnloWEvYB5ODw19+Q/VMP9/PTTsBczD6REjPAB37QG47a+/9gDctQfgtr/5xgNw1x6A2/72Ww/AXXsAbvu77zwAd+0BuO3vv/cA3LUH4LZ/+MEDcNcegNv+8UcPwF17AG77p588AHftAbjtn3/2ANy1B+C2f/nFA3DXHoDb/vVXD8BdewBu+7ffPAB37QG47d9/9wDctQfgtv/4wwNw1x6A2/7zTw/AXXsAbvuvvzwAd+0BuO2///YA3LUH4Lb/+ccDcNcegNv+918PwF17AG575EgPwF17AG571CgPwF17AG579GgPwF17AG57zBgPwF17AG577FgPwF17AG573DgPwF17AG57/HgPwF17AG57wgQPwF17AG574kQPwF17AG570iQPwF17AG578mQPwF17AG57yhQPwF17AG576lQPwF17AG572jQPwF17AG57+nQPwF17AG57xgwPwF17AG575kwPwF17AG571iwPwF17AG579mwPwF17AG57zhwPwF17AG577lwPwF17AG573jwPwF17AG57/nwPwF17AG57wQIPwF17AG574UIPwF17AG570SIPwF17AG578WIPwF17AG57yRIPwF17AG576VIPwF17AG572TIPwF17AG57+XIPwF17AG57xQoPwF17AG575UoPwF17AG571SoPwF17AG579WoPwF17AG57zRoPwF17AG577VoPwF17AG573ToPwF17AG57/XoPwF17AG57wwYPwF17AG5740YPwF17AG570yYPwF17AG5782YPwF17AG57yxYPwF17AG5761YPwF17AG572zYPwF17AG57+3YPwF17AG57xw4PwF17AG57504PwF17AG571y4PwF17AG57924PwF17AG57zx4PwF17AG57714PwF17AG573z4PwF17AG57/34PwF17AG77wAEPwF17AG774EEPwF17AG770CEPwF17AG778GEPwF17AG77yBEPwF17AG776FEPwF17AG772DEPwF17AG77+HEPwF17AG77xAkPwF17AG775EkPwF17AG771CkPwF17AG779GkPwF17AG77zBkPwF17AG777FkPwF17AG773DkPwF17AG77/HkPwF17AG77wgUPwF17AG774kUPwF17AG770iUPwF17AG778mUPwF17AG77yhUPwF17AG776lUPwF17AG772jUPwF17AG77+nUPwF17AG77xg0PwF17AG775k0PwF17AG771i0PwF17AG779m0PwF17AG77zh0PwF17AG777l0PwF17AG773j0PwF17AG77/n0PwF17AG77wQMPwF17AG774UMPwF17AG770SMPwF17AG778WMPwF17AG77yRMPwF17AG776VMPwF17AG772TMPwF17AG77+XMPwF17AG77xQsPwF17AG775UsPwF17AG771SsPwF17AG779WsPwF17AG77zRsPwF17AG777VsPwF17AG773TsPwF17AG77/XsPwF17AG77wwcPwF17AG7740cPwF17AG7782cPwF17AE77fwEAAP//oVuGohAcKHYAAAAASUVORK5CYII=">

As you append to a slice, its capacity [doubles in size](https://stackoverflow.com/a/45423692/1087122) every time it exceeds its current capacity.
