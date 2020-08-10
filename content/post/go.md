+++
title = "Go"
description = "Go Go Go"
tags = [
    "go"
]
date = "2020-06-21"
topics = [
    "go"
]
toc = true
+++

尴尬了，这篇的日期写的是`2018-01-03`，实际上是近三年前的帖子了。Go 1.9发布的时候就信誓旦旦的要进行学习。结果到今天还是无疾而终的样子，半途而废了好久。捞上来，看看这次可以坚持多久。

[Mixin 大群部署完全教程](https://dbarobin.com/2019/05/19/mixin-super-group/) 拿这个练习，前后端一起端。

<!--more-->

### Wire 

[go wire](https://github.com/google/wire) and [wire blog](https://blog.golang.org/wire)

前提：已经将`$GOPATH/bin`目录添加到环境变量`$PATH`
安装`go get github.com/google/wire/cmd/wire`

[wire tutorial](https://github.com/google/wire/blob/master/_tutorial/README.md)

- Goland 提示 `wire.go doesn't match to garget system. File will be ignored by build tool` 

在设置--> Go --> Build Tags & Vendoring中的Custom tags里指定要用的编译tag，例如 `wireinject`。参考[指定编译tag](https://studygolang.com/topics/10193)，[编译go build -tags](https://www.jianshu.com/p/858a0791f618)，[编译官方手册](https://pkg.go.dev/go/build?tab=doc)

- 执行wire命令时提示`go list stderr <<go: finding module for package pattern=. >>` 

```
go list stderr <<go: finding module for package pattern=. >>
wire: packages not found
wire: generate failed
```

[官方issue #125](https://github.com/google/wire/issues/125#issuecomment-462505979)中的解决方案——

```
# 先升级 go/packages，可以添加 -v参数看到升级了哪些packages
go get -u golang.org/x/tools/go/packages
# 再重新安装 wire
go get github.com/google/wire/cmd/wire
```

[解释](https://github.com/google/wire/issues/125#issuecomment-462516570)是这样的——

>So Wire uses [golang.org/x/tools/go/packages](https://godoc.org/golang.org/x/tools/go/packages) behind the scenes in order to gather all of the files to parse. go/packages is specifically designed to be agnostic to source code layout (e.g. modules versus GOPATH workspaces), so it shells out to the go tool to obtain the information that it needs. Since this requires cooperation from both your installed go tool and the go/packages version, an outdated go/packages library can mess up the communication and report bad results to Wire (which is what happened here).
>
>In the Go modules future, your built version of Wire would have used a known-tested version of go/packages and this issue would not have occurred.

执行完之后，可以正常生成`wire_gen.go`文件。可以先执行`wire check`检查是否符合编译条件。

执行完一次`wire`命令之后，再次需要更新`wire_gen.go`文件时，执行`go generate`命令即可

[undefined: InitializeEvent](https://github.com/google/wire/issues/224#issuecomment-558819174)， 使用wire生成注入代码之后，编译时需要带上对应的代码。

执行 `go build`默认使用了上次的配置? `go build main.go wire_gen.go`可生成可执行文件

### fmt string 

[Go by Example: String Formatting](https://gobyexample.com/string-formatting)

```go
package main

import (
	"fmt"
	"os"
)

type point struct {
	x, y int
}

func main() {

	p := point{1, 2}

	// v for verbs? value?
	// {1 2}
	fmt.Printf("%v\n", p)

	// include the struct’s field names.
	// {x:1 y:2}
	fmt.Printf("%+v\n", p)

	//  prints a Go syntax representation of the value
	//  main.point{x:1, y:2}
	fmt.Printf("%#v\n", p)

	// print the type of a value
	// main.point
	fmt.Printf("%T\n", p)

	// Formatting booleans
	// false
	fmt.Printf("%t\n", false)

	//  base-10 formatting.
	// 123
	fmt.Printf("%d\n", 123)

	// binary formatting
	// 1110
	fmt.Printf("%b\n", 14)

	// prints the character corresponding to the given integer.
	// !
	fmt.Printf("%c\n", 33)

	// provides hex encoding.
	// 1c8
	fmt.Printf("%x\n", 456)

	//  formatting options for floats. For basic decimal formatting use %f.
	// 78.900000
	fmt.Printf("%f\n", 78.9)

	// format the float in (slightly different versions of) scientific notation
	// 1.234000e+08
	fmt.Printf("%e\n", 123400000.0)
	// 1.234000E+08
	fmt.Printf("%E\n", 123400000.0)

	// print basic string
	// "string"
	fmt.Printf("%s\n", "\"string\"")

	// double-quote strings
	// "\"string\""
	fmt.Printf("%q\n", "\"string\"")

	// renders the string in base-16, with two output characters per byte of input
	// 6865782074686973
	fmt.Printf("%x\n", "hex this")

	// print a representation of a pointer
	// 0xc00000a210 (address)
	fmt.Printf("%p\n", &p)

	// specify the width of an integer, use a number after the % in the verb. By default the result will be right-justified and padded with spaces.
	// |    12|   345|
	fmt.Printf("|%6d|%6d|\n", 12, 345)

	// restrict the decimal precision at the same time with the width.precision syntax
	// |  1.20|  3.45|
	fmt.Printf("|%6.2f|%6.2f|\n", 1.2, 3.45)

	// left-justify, use the - flag
	// |1.20  |3.45  |
	fmt.Printf("|%-6.2f|%-6.2f|\n", 1.2, 3.45)

	// format string
	// |   foo|     b|
	fmt.Printf("|%6s|%6s|\n", "foo", "b")

	// |foo   |b     |
	fmt.Printf("|%-6s|%-6s|\n", "foo", "b")

	// return string
	s := fmt.Sprintf("a %s", "string")
	fmt.Println(s)

	// format+print to io.Writers other than os.Stdout using Fprintf.
	fmt.Fprintf(os.Stderr, "an %s\n", "error")

}

```

### 闭包函数传参： 

[Passing parameters to function closure](https://stackoverflow.com/a/30183893/1087122)，[Using goroutines on loop iterator variables](https://github.com/golang/go/wiki/CommonMistakes#using-goroutines-on-loop-iterator-variables)

```go
for i := 0; i < 3; i++ {
    go func() {
        fmt.Println(i)
    }()
}
//result: 3 3 3

for i := 0; i < 3; i++ {
    go func(v int) {
        fmt.Println(v)
    }(i)
}
// result: 0, 1, 2
```

### How to write Go code

重新更新了环境，使用`go1.14.4`开始，即使在window环境上，[安装](https://golang.org/doc/install)也只需要解压、更新环境变量（如果使用cgo的话，还需要gcc环境，机器上已经有了，但目前以我这水平应该还用不到）就可以立即上手了。

[How to write Go code](https://golang.org/doc/code.html)实践——

- 创建工作目录，后续在工作目录进行操作
- 使用`go mod init example.com/user/hello`进行初始化，生成`go.mod`文件，包含了包名和使用的版本名
- 使用`go install`进行安装，三种等价操作。`go install .`, `go install example.com/user/hello`
- 在本地定义引用的包信息：目录名作为引入的内容，其中的方法使用大写字母开头自动`exported`，使用`go build`就可以将引用包安装到本地。之后就可以在应用中使用
- 引入remote的包会自动去下载远程的包。通过`go install`、`go build`、`go run`都可以触发下载动作，并会更新`go.mod`文件
- 使用 `go test`进行测试

注意事项：   

1. 可以使用`go env -w GOBIN=/path/to/bin`和`go env -u GOBIN`进行变量声明和修改
2. 在cmd环境下可以临时设置代理，持续到窗口关闭，前提是本地已经有了proxy`set http_proxy=http://127.0.0.1:7890`, `set https_proxy=http://127.0.0.1:7890`
3. 使用`go test ./...`可以自动遍历当前工程所有文件夹下的test文件



要用到`go module`，参考这两篇——[Go Modules 终极入门](https://mp.weixin.qq.com/s/6gJkSyGAFR0v6kow2uVklA)，[干货满满的 Go Modules 和 goproxy.cn](https://mp.weixin.qq.com/s/jpp7vs3Fdg4m15P1SHt1yA)。


### Gocker Docker

[用 Go 从头实现一个迷你 Docker — Gocker](https://mp.weixin.qq.com/s/P9bVFtXeduZ8Lv-0vFaXAg)   
[原文: Containers the hard way: Gocker: A mini Docker written in Go](https://unixism.net/2020/06/containers-the-hard-way-gocker-a-mini-docker-written-in-go/)  
[源码：Gocker](https://github.com/shuveb/containers-the-hard-way)  

这个看起来有点帅啊，而且代码不是很多的样子，结合耗子叔的这几篇一起理解更有帮助——

[DOCKER基础技术：LINUX NAMESPACE（上）](https://coolshell.cn/articles/17010.html)  
[DOCKER基础技术：LINUX NAMESPACE（下）](https://coolshell.cn/articles/17029.html)  
[DOCKER基础技术：LINUX CGROUP](https://coolshell.cn/articles/17049.html)  
[DOCKER基础技术：AUFS](https://coolshell.cn/articles/17061.html)  
[DOCKER基础技术：DEVICEMAPPER](https://coolshell.cn/articles/17200.html)  

### 结构体对应数据绑定

[Go Echo: get started](https://mp.weixin.qq.com/s/DuEGITdOYHYOXk-v-u2V5A), [Go Echo: basic features](https://mp.weixin.qq.com/s/vg9OSO4g0KG7iDQ7GXoUSQ)跟着这个例子写一个简单的登录demo。

趁热打铁，第三篇新鲜出炉[Go Echo: custom Binder](https://mp.weixin.qq.com/s/2wARcsszax6GFbZcjJPoig) 

- param tag 对应路径参数；
- query tag 对应 URL 参数；
- json tag 对应 application/json 方式参数；
- form tag 对应 POST 表单数据；
- xml tag 对应 application/xml 或 text/xml；

```go
// 表述结构体对应数据绑定时，对应的tag和字段。 如，json类型的name字段；form类型的name字段
type User struct {
 Name string `query:"name" form:"name" json:"name" xml:"name"`
 Sex  string `query:"sex" form:"sex" json:"sex" xml:"sex"`
}
```

顺带安装一下很好用的http调试工具[httpie](https://httpie.org/docs#installation)，python写得，依赖python3.6以上版本（本地的环境已经很混乱得啥都有， 使用`py --version`可以调用python3版本），直接使用 `pip install  --upgrade httpie`安装成功了。

没有进行数据绑定时，传递xml类型数据时，被认为是`Content-Type: text/plain;`；指定了绑定规则后可以识别并转换为`Content-Type: application/json; charset=UTF-8`

### 正则限制 

[https://groups.google.com/forum/#!topic/golang-nuts/7qgSDWPIh_E](https://groups.google.com/forum/#!topic/golang-nuts/7qgSDWPIh_E)

> that (?!re) regular expression is not supported neither in re2 nor in Google Go. Is there any chance that its support will be implemented in future releases? (it is supported at least in Ruby, Python, Java, Perl)

[https://www.reddit.com/r/programmingcirclejerk/comments/5ml6yj/golangs_standard_regex_library_doesnt_have/](https://www.reddit.com/r/programmingcirclejerk/comments/5ml6yj/golangs_standard_regex_library_doesnt_have/)

[go-issues:18868 regexp: support lookaheads and lookbehinds](https://github.com/golang/go/issues/18868) 


[Golang doesn't support positive lookbehind assertion?](https://github.com/StefanSchroeder/Golang-Regex-Tutorial/issues/11)

支持的场景： [https://github.com/google/re2/wiki/Syntax](https://github.com/google/re2/wiki/Syntax)要求[`makes a single scan over the input and runs in O(n) time`](https://groups.google.com/d/msg/golang-nuts/7qgSDWPIh_E/OHTAm4wRZL0J)

>The lack of generalized assertions, like the lack of backreferences, is not a statement on our part about regular expression style.  It is a consequence of not knowing how to implement them efficiently.  If you can implement them while preserving the guarantees made by the current package regexp, namely that it makes a single scan over the input and runs in O(n) time, then I would be happy to review and approve that CL.  However, I have pondered how to do this for five years, off and on, and gotten nowhere.

包含在一传字符串中的手机号，类似`abcd13566778888xyz`这样的手机号码前后`不包含数字`的正则在Go语言里是不支持的。因为无法`一次扫描并在O(n)的时间`里完成。

`(?=re)`	before text matching `re` (NOT SUPPORTED)
`(?!re)`	before text not matching `re` (NOT SUPPORTED)
`(?<=re)`	after text matching `re` (NOT SUPPORTED)
`(?<!re)`	after text not matching `re` (NOT SUPPORTED)


### go vendor 依赖

低版本下对于vendor目录的查找逻辑：

- 需要将项目建立在`$GOPATH`目录下的`src`目录（前提）
- 查找当前工程下的vendor目录(vendor tree) （如果项目没有在`$GOPATH`目录下，这一步将会被忽略）[go issue #14566](https://github.com/golang/go/issues/14566)
- 在`$GOROOT`目录下查找(from $GOROOT)
- 在`$GOPATH`目录下查找(from $GOPATH)

--- 

## archive

>Go release 1.9 version on 24 August 2017. I'm the [31,365](https://github.com/golang/go/stargazers) people to start it on github and just get started to learn it :). 
>
>[Go 1.9 is released](https://blog.golang.org/go1.9). [**Here**](https://blog.golang.org/index) are all the blogs about go.
>
>[Go: Ten years and climbing](https://commandcenter.blogspot.co.uk/2017/09/go-ten-years-and-climbing.html)
>[Go十年](http://tonybai.com/2017/09/24/go-ten-years-and-climbing/)
>[BIG: Blockchain In Go](https://jeiwan.cc/tags/blockchain/)


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

### go test 

同事的项目本地执行没有问题，线上跑go test的时候一直无法通过，build 失败。

最终定位原因：   

- 环境需要安装gcc环境  
- 打开golang的环境变量 `CGO_ENABLED="1"`

环境默认是打开了CGO的，但执行go test时会报gcc错误。为了不安装gcc环境，强制修改了这个变量（还尝试了半天修改的方法）

这是因为甚至是`go test ./...`时，有些报会使用到 “C混合编译”，需要注意这俩个关键因素。

### go import 

[Understanding Dependency Management in Go](https://lucasfcosta.com/2017/02/07/Understanding-Go-Dependency-Management.html) Understanding Vendoring：  

In order to be able to fully understand how vendoring works we must understand the algorithm used by Go to resolve import paths, which is the following:

1. Look for the import at the local vendor directory (if any)
2. If we can’t find this package in the local vendor directory we go up to the parent folder and try to find it in the vendor directory there (if any)
3. We repeat step 2 until we reach $GOPATH/src
4. We look for the imported package at $GOROOT
5. If we can’t find this package at $GOROOT we look for it in our $GOPATH/src folder

### go mod

[Go mod 使用](https://segmentfault.com/a/1190000018536993)  
[告别GOPATH，快速使用 go mod（Golang包管理工具）](https://www.jianshu.com/p/bbed916d16ea)  

go.mod文件中定义了当前项目对应的module名称，如`golang.gebitang.com/my/module`。

对于`pkg/util/log`的包，当前项目中使用import时，可以使用下面的方式进行引入。go mod模块会自动进行转换
```
import (
    "golang.gebitang.com/my/module/pkg/util/log"
)
```

go mod将依赖统一放到GOPATH下的pkg下的pkg下面，并且支持不同版本（使用@vMajor.minor.path）的格式管理


<img src="data:image/png;base64, iVBORw0KGgoAAAANSUhEUgAAAQAAAAEACAIAAADTED8xAAAE8UlEQVR4nOzX36cQBgOA4XOONjMzMzMzMzMzM0mSJEmSJEmSJEmSJEmSJEmSJEmSJEmSJEmSJEmSJEmSJEmSJEmSJP34/ovvjPe5ey7f23fEwMCXwcGBr74aGBwc+PRpgDnloWEvYB5ODw19+Q/VMP9/PTTsBczD6REjPAB37QG47a+/9gDctQfgtr/5xgNw1x6A2/72Ww/AXXsAbvu77zwAd+0BuO3vv/cA3LUH4LZ/+MEDcNcegNv+8UcPwF17AG77p588AHftAbjtn3/2ANy1B+C2f/nFA3DXHoDb/vVXD8BdewBu+7ffPAB37QG47d9/9wDctQfgtv/4wwNw1x6A2/7zTw/AXXsAbvuvvzwAd+0BuO2///YA3LUH4Lb/+ccDcNcegNv+918PwF17AG575EgPwF17AG571CgPwF17AG579GgPwF17AG57zBgPwF17AG577FgPwF17AG573DgPwF17AG57/HgPwF17AG57wgQPwF17AG574kQPwF17AG570iQPwF17AG578mQPwF17AG57yhQPwF17AG576lQPwF17AG572jQPwF17AG57+nQPwF17AG57xgwPwF17AG575kwPwF17AG571iwPwF17AG579mwPwF17AG57zhwPwF17AG577lwPwF17AG573jwPwF17AG57/nwPwF17AG57wQIPwF17AG574UIPwF17AG570SIPwF17AG578WIPwF17AG57yRIPwF17AG576VIPwF17AG572TIPwF17AG57+XIPwF17AG57xQoPwF17AG575UoPwF17AG571SoPwF17AG579WoPwF17AG57zRoPwF17AG577VoPwF17AG573ToPwF17AG57/XoPwF17AG57wwYPwF17AG5740YPwF17AG570yYPwF17AG5782YPwF17AG57yxYPwF17AG5761YPwF17AG572zYPwF17AG57+3YPwF17AG57xw4PwF17AG57504PwF17AG571y4PwF17AG57924PwF17AG57zx4PwF17AG57714PwF17AG573z4PwF17AG57/34PwF17AG77wAEPwF17AG774EEPwF17AG770CEPwF17AG778GEPwF17AG77yBEPwF17AG776FEPwF17AG772DEPwF17AG77+HEPwF17AG77xAkPwF17AG775EkPwF17AG771CkPwF17AG779GkPwF17AG77zBkPwF17AG777FkPwF17AG773DkPwF17AG77/HkPwF17AG77wgUPwF17AG774kUPwF17AG770iUPwF17AG778mUPwF17AG77yhUPwF17AG776lUPwF17AG772jUPwF17AG77+nUPwF17AG77xg0PwF17AG775k0PwF17AG771i0PwF17AG779m0PwF17AG77zh0PwF17AG777l0PwF17AG773j0PwF17AG77/n0PwF17AG77wQMPwF17AG774UMPwF17AG770SMPwF17AG778WMPwF17AG77yRMPwF17AG776VMPwF17AG772TMPwF17AG77+XMPwF17AG77xQsPwF17AG775UsPwF17AG771SsPwF17AG779WsPwF17AG77zRsPwF17AG777VsPwF17AG773TsPwF17AG77/XsPwF17AG77wwcPwF17AG7740cPwF17AG7782cPwF17AE77fwEAAP//oVuGohAcKHYAAAAASUVORK5CYII=">

As you append to a slice, its capacity [doubles in size](https://stackoverflow.com/a/45423692/1087122) every time it exceeds its current capacity.
