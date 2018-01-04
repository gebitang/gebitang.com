+++
title = "Let's go"
description = ""
tags = [
    "go"
]
date = "2017-08-25"
topics = [
    "go"
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

As you append to a slice, its capacity [doubles in size](https://stackoverflow.com/a/45423692/1087122) every time it exceeds its current capacity.

