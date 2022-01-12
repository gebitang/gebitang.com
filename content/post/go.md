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

## 2021 

### quorum 

API文档查看，依赖[swaggo](https://github.com/swaggo/swag)，启动swagger服务依赖swaggo生成的docs包。scripts文件夹下提供了对应的生成脚本，结果我自己研究了半天，只生成了个空的swagger接口文档。

最终在wsl环境下生产，再把文件复制回来即可。

网络状态判断：`/v1/network` API返回`pkg/p2p/network.go`中的`func (node *Node) eventhandler(ctx context.Context) `的状态，订阅了网络状态的事件[Reachability](https://pkg.go.dev/github.com/libp2p/go-libp2p-core/network#Reachability)，默认未知，如果公网可访问则是public状态，否则是private

编译quorum，在服务器端启动：

- 启动需要添加`-ips public.ip.xx.xx`参数
- 远程使用rum-app节点模式连接时，需要在本地做接口映射，否则需要提供jwt信息
- 可以先通过api生成token，则不需要进行端口转发 `curl -k -X POST "https://127.0.0.1:8002/app/api/v1/token/apply" -H "accept: application/json"`

接口转发Fow Windows，使用`netsh`，使用管理员权限执行

`netsh interface portproxy add v4tov4 listenport=8002 listenaddress=127.0.0.1 connectport=8002 connectaddress=public.ip.xx.xx`

删除转发：[参考](https://stackoverflow.com/a/11535395/1087122)

`netsh interface portproxy delete v4tov4 listenport=8002 listenaddress=127.0.0.1`

接口转发Fow MacOS，使用`nmap`，brew安装如果有问题，直接从[官网下载编译好的二进制文件安装](https://nmap.org/download.html#macosx)

`ncat -l 127.0.0.1 8002 --sh-exec "ncat public.ip.xx.xx 8002" --keep-open` 

Mac端开源通过Ctrl+C停止转发

**异常现象**  使用jwt认证后，证书信息不再是必须的(配置为空时会提醒报错，但实际可以随意填写)

>server端：echo: http: TLS handshake error from 82.129.30.127:54584: remote error: tls: unknown certificate如果证书错误，将连续报六次相同的错误；证书正确时，会报错一次(因为使用的是自定义证书的原因？)。
>client端：net::ERR_CERT_AUTHORITY_INVALID 证书错误时，收到此消息；证书正确时建立TLS链接

### TLS 

[TLS握手是如何建立的](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/)

- **[时机]** TCP握手建立后就开始进行TLS握手过程(先建立TCP握手之后有了通道之后才能进行TLS握手) 
- **[内容]** 握手要确定是事情包括：1)使用哪个版本的TLS；2)使用什么密码套件(cipher suites，即使用什么算法)；3)验证服务器的身份(通过服务器的公钥和SSL证书机构的签名)

**RSA key** 握手交换过程具体如下—— 

1. 客户端“hello”信息。客户端发起请求，包括三项内容：TLS版本、支持的密码套件组(供服务端进行选择)、随机字符串(client random，这项内容最后会参与生成秘钥session key)
2. 服务端“hello”信息。服务端回应请求，包括三项内容：服务端的SSL证书、选择的密码套件、随机字符串(server random，作用同上)
3. 验证。客户端验证SSL证书(确认访问的域名是证书所声明的域名)
4. 预密码(premaster secret)。客户端使用收到的SSL证书中的公钥签名一段随机字符串作为预密码(premaster secret)发送给服务端(只有服务端的私钥才能解密)
5. 解密。服务端解密预密码
6. 创建Session key。使用client random、server random、premaster secret创建session key。双方将产生相同的结果
7. 客户端使用session key签名发送“finished”消息
8. 服务端做相同的动作
9. 握手完成。对称加密通道建立


**DH秘钥**握手交换具体如下——(DH秘钥不同之处是Server端和Client端使用不同的DH参数，但双方依然可以获得相同的结果。)

1. 相同
2. 相同+Server端使用私钥对client random, server random和DH参数进行加密生成自身的签名
3. 客户端使用公钥解密收到的签名，验证服务端拥有对应的私钥(解密信息包括client random、server random，后者本身也会被单独收到)
4. 客户端发送自己选择的DH参数
5. 客户端不需要签名生成premaster secret，并发送给服务端。此时双方可以根据DH参数各自独立计算出premaster secret  

后续动作一样。使用client random、server random、premaster secret创建session key……发送finish消息……建立对称加密安全通道

通道建立后，后续每次通信都将使用不同的session key进行签名，同时TLS通过MAC(message authentication code)确保每次发送的信息没有被篡改。[参考how-does-ssl-work](https://www.cloudflare.com/learning/ssl/how-does-ssl-work/)


配合实例——可以看到TLS握手建立的过程。更详细的说明[解刨TLS](https://www.catchpoint.com/blog/wireshark-tls-handshake)，这里面根据wireshark的抓包信息有更详细的解释。

```shell
➜ curl -k -vvv  "https://127.0.0.1:8002/api/v1/groups2" -H "accept: application/json"
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 8002 (#0)
* ALPN, offering h2 # https://www.keycdn.com/support/alpn Application-Layer Protocol Negotiation 可减少round trip
* ALPN, offering http/1.1
* successfully set certificate verify locations: 
*   CAfile: /etc/ssl/cert.pem #证书验证位置
  CApath: none #证书路径
* TLSv1.2 (OUT), TLS handshake, Client hello (1): 
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server accepted to use h2
* Server certificate:
*  subject: O=Acme Co; CN=*
*  start date: Dec 31 10:38:55 2021 GMT
*  expire date: Dec 29 10:38:55 2031 GMT
*  issuer: O=Acme Co; CN=*
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x7fc8b800b600)
> GET /api/v1/groups2 HTTP/2
> Host: 127.0.0.1:8002
> User-Agent: curl/7.64.1
> accept: application/json
>
* Connection state changed (MAX_CONCURRENT_STREAMS == 250)!
< HTTP/2 404
< content-type: application/json; charset=UTF-8
< content-length: 24
< date: Wed, 12 Jan 2022 10:11:28 GMT
<
{"message":"Not Found"}
* Connection #0 to host 127.0.0.1 left intact
* Closing connection 0


➜ curl  -vvv  "https://qq.com/healthcheck" -H "accept: application/json"
*   Trying 58.250.137.36...
* TCP_NODELAY set
* Connected to qq.com (58.250.137.36) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/cert.pem
  CApath: none
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server accepted to use h2
* Server certificate:
*  subject: C=CN; ST=\U5E7F\U4E1C\U7701; L=\U6DF1\U5733\U5E02; O=Shenzhen Tencent Computer Systems Company Limited; CN=qq.com
*  start date: Jul 26 00:00:00 2021 GMT
*  expire date: Jul 26 23:59:59 2022 GMT
*  subjectAltName: host "qq.com" matched cert's "qq.com"
*  issuer: C=US; O=DigiCert Inc; CN=DigiCert Secure Site CN CA G3
*  SSL certificate verify ok.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x7fa424811c00)
> GET /healthcheck HTTP/2
> Host: qq.com
> User-Agent: curl/7.64.1
> accept: application/json
>
* Connection state changed (MAX_CONCURRENT_STREAMS == 128)!
< HTTP/2 302
< server: ias/1.4.2.3_1.17.3
< date: Wed, 12 Jan 2022 10:11:55 GMT
< content-type: text/html
< content-length: 151
< location: https://www.qq.com/healthcheck
<
<html>
<head><title>302 Found</title></head>
<body>
<center><h1>302 Found</h1></center>
<hr><center>ias/1.4.2.3_1.17.3</center>
</body>
</html>
* Connection #0 to host qq.com left intact
* Closing connection 0

```

### pass 

[The Missing Semester of Your CS Education](https://missing.csail.mit.edu/)的[中文版本](https://github.com/missing-semester-cn/missing-semester-cn.github.io)，提到安全相关的应用：

- PGP电子邮件加密，windows平台下可以使用[gpg4win](https://www.gpg4win.org/)，参考[PGP加密电子邮件演示](https://mycyberpunk.com/2020/04/27/pgp-email/)或[利用 PGP 技术对你的邮件进行加密](https://sspai.com/post/35592)，使用非对称加密邮件：将要发送的邮件进行加密，邮件客户端进行解密。公布自己的PGP公钥，这样别人可以使用公钥加密要发送给你的内容，自己的邮件客户端使用私钥进行解密即可
- 密码管理。提到了[pass](https://www.passwordstore.org/)，之前经理[介绍过](https://gitpress.io/@lyric/password-management-with-pass)。项目有很多配套支持，发现使用go语言开发的gui项目已经暂停开发(因为依赖的[qml](https://github.com/go-qml/qml)项目停止维护)，作者重新使用rust语言写了新的客户端。前两天看到[wormhole-gui](https://github.com/jacalz/wormhole-gui)项目，可以依赖[fyne](https://github.com/fyne-io/fyne)项目(Cross platform GUI in Go inspired by Material Design)实现一个兼容的客户端？提供了从其他密码软件导入的功能：通常是导出不同的文件格式，然后进行格式转换即可，例如lasspass可以导出cvs文件，然后解析并导入到pass，提供了[ruby脚本](https://git.zx2c4.com/password-store/tree/contrib/importers/lastpass2pass.rb)


利用git同步密码仓库，全平台覆盖

**基础操作**

- 显示所有内容`pass`
- 显示某一项内容 `pass folder/site.name`
- 新建一项纪录 `pass insert folder/site.name` 手动输入密码
- 自动创建密码 `pass generate folder/site.name 15` 自动生成长度15字符串的密码
- 删除密码项目 `pass rm folder/site.name` 

**pgp秘钥**

- 首先需要生成一个pgp秘钥(用来加密对应的密码) 

```shell
#安装
apt-get install gnupg
#生成证书
gpg --gen-key
#导出证书，pass在不同设备间同步时，解密需要
gpg --output private.pgp --armor --export-secret-key username@email #私钥
gpg --armor --output public.key --export username@email  #公钥
# 更新证书等操作参考： man 手册或官方文档 https://gnupg.org/documentation/manuals/gnupg/
#导入证书， 在另外的linux上重建pass
gpg --import private.pgp 
gpg --import publick.key
```

**安装pass**

`sudo apt-get install pass`

- 生成密码库 `pass init "pgp key id"`，详细步骤[参考](https://git.zx2c4.com/password-store/about/#EXTENDED%20GIT%20EXAMPLE)

```shell
pass init "pgp key id"
pass git init
pass git remote add origin git.store.url
pass git push -u --all 
```

- 正常维护密码库，然后进行同步即可。

**Android同步**

需要安装对应的android app [Android-Password-Store](https://github.com/android-password-store/Android-Password-Store#readme)和 [OpenKeyChain](https://github.com/open-keychain/open-keychain)

前者用来同步git仓库，后者用来导入gpg证书(上一步中使用的证书)。两者配合就可以在android上使用

**浏览器同步**

这样可以在不同的PC、Mac平台使用。

- 安装[gpg4win](https://www.gpg4win.org/)。导入pass使用的gpg证书
- 安装[browserpass-native](https://github.com/browserpass/browserpass-native)，从release页面进行安装
- 安装[browserpass-extension](https://github.com/browserpass/browserpass-extension)，从chrome web store安装
  - 配置插件的`Custom gpg binary`，通常在安装gpg4win同级的目录`GnuPG\bin\gpg.exe`(这里折腾了半天，从命令行里找到了灵感`which gpg`)
  - 配置同步后的git目录

**迁移**

从lastPass导出csv文件，linux下安装ruby环境，执行导入`./lastpass2pass.rb path/to/passwords_file.csv` 如果存储的数据不规范，可能报错，删除对应的数据即可。

>文件夹乱码''$'\020'(编码为\020)，在linxu下可以创建文件夹成功，在window环境下无法创建对应的文件夹，导致git同步失败。  
> 导入需要做一些必要的处理(正好借此机会整理一下密码列表)，因为pass以`文件夹/域名名称`做区分，lastPass下可以在同一个域名下保存多个用户名密码，后者的搜索也更强大   

**pass同步**

条件：pass的git仓库已经存在，需要在另外一台linux机器上进行同步。

- 安装pass，导入相同的公钥私钥
- `git clone git.path.url ~/.password-store ` 这个步骤比较关键，这样才能确保文件目录一致
- 然后执行`pass init fingerprint`进行初始化

**问题**

- 看起来导入私钥有问题，每次编码总进行提醒`There is no assurance this key belongs to the named user`， 需要进行[trust操作](https://stackoverflow.com/a/34132924/1087122)
- `pass edit folder/name`时会使用默认的编辑器，编辑修改默认的编辑器`sudo update-alternatives --config editor`
- `pass -c folder/name`提示: `Can't open display:0`。

[参考](https://stackoverflow.com/questions/61110603/how-to-set-up-working-x11-forwarding-on-wsl2)：

```shell
export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2}'):0
export LIBGL_ALWAYS_INDIRECT=1
```

需要激活X11 server，本地使用了MobaXTerm，提示允许后，可以剪切到内容，但不稳定。对应的[Can't use X-Server in WSL 2](https://github.com/microsoft/WSL/issues/4106)，先pending着再说


```shell
gpg --edit-key <KEY_ID>
gpg> trust
# You will be asked to select the trust level from the following:
1 = I don't know or won't say
2 = I do NOT trust
3 = I trust marginally
4 = I trust fully
5 = I trust ultimately
m = back to the main menu

Your decision? 5
Do you really want to set this key to ultimate trust? (y/N) y

# 可以执行passwd进行密码修改
passwd
# save
save
```

**保存私钥**

还差最后一步，如何保存生成的私钥。

- 使用openssl对称加密对应的私钥和公钥文件 `openssl aes-256-cbc -in public.key -out sslpub.key`将秘钥加密，私钥采用对应的操作
- 将加密后的文件保存为hex格式——方便保存 `xxd -p sslpub.key > hexsslpub.key`
- hexsslpub.key文件的内容就可以公开分发了，例如同步到Rum群组中

**恢复秘钥**

- `xxd -r -p hexsslpub.key sslpub.key`将hex格式的文件重新恢复为加密后的文件
- 使用openssl进行解密`openssl aes-256-cbc -d -in sslpub.key -out publicreverse.key` 
- 验证恢复后的两个文件是否一致，例如检查md5或进行字节比对`cmp public.key publicreverse.key`

整个方案中只需要记住两个密码：一个是pgp的密码，一个是对称加解密的密码。

### clash 

>V2ray + COW实现go build编译。前者提供Sock5代理，后者提供http代理，并使用Sock5做二级代理 

实践上可以直接使用[clash](https://github.com/Dreamacro/clash)，完全使用go语言实现的clash core，[支持proxy providers](https://github.com/Dreamacro/clash/wiki/configuration#proxy-providers)，windows平台下的[clash for windows](https://github.com/Fndroid/clash_for_windows_pkg)，其实就是使用Electron开发的基于Clash内核的GUI版本(`A Windows/macOS/Linux GUI based on Clash and Electron.`)

>- Clash：一个 Go 语言开发的多平台代理客户端，[Github](https://github.com/Dreamacro/clash)
>- ClashX：Clash 的 Mac 图形客户端，[Github](https://github.com/yichengchen/clashX)
>- ClashForAndroid：Clash 的 Android 图形客户端，[Github](https://github.com/Kr328/ClashForAndroid)
>- Clash for Windows：Clash 的 Windows/macOS 图形客户端，[Github](https://github.com/Fndroid/clash_for_windows_pkg)

命令行使用基础：

- 下载对应平台[最新版本](https://github.com/Dreamacro/clash/releases/latest)
- 解压文件`gzip -d clash-linux-amd64-v1.9.0.gz`后，添加可执行权限`chmod +x clash-xxx`
- `cp clash-darwin-amd64-vx.y.z /usr/local/bin/clash`添加到用户可执行文件目录
- 下载配置文件到目录`mkdir /usr/local/etc/clash`
- 执行 `clash -d /usr/local/etc/clash` 即可运行 clash 并打印暴露在本地的代理服务端口，默认为 `127.0.0.1:7890`

### unable to find utility "clang", not a developer tool or in PATH

执行`go run main.go`提示如下错误，找不到clang环境。
```
 go run main.go
# runtime/cgo
sh: line 1: 52743 Bus error: 10           /Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild -sdk /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.14.sdk -find clang 2> /dev/null
clang: error: unable to find utility "clang", not a developer tool or in PATH
```

需要先安装`CommandLineTools`，可以先检查`xcode-select --print-path`安装路径，如果不是`/Library/Developer/CommandLineTools`，则需要先安装`xcode-select --install`，然后修改路径`sudo xcode-select --switch /Library/Developer/CommandLineTools`。正常情况下，此时环境恢复正常。

通过命令行更新xcode版本：检查配套版本：`softwareupdate --list`。参考[How to update Xcode from command line](https://stackoverflow.com/questions/34617452/how-to-update-xcode-from-command-line)老版本可参考此操作。

### go mail 

原生自动的需要指定证书。本地测试会报错`x509: certificate signed by unknown authority`，不指定证书的验证方式，使用[go-simple-mail](https://github.com/xhit/go-simple-mail)完成。

其中自定义了`loginAuth`模式，实现官方的auth接口；指定为`EncryptionNone`模式时，直接建立连接，不使用tls模式——

```go
case EncryptionSSL, EncryptionSSLTLS:
		conn, err = tls.Dial("tcp", address, config)
default:
	conn, err = net.Dial("tcp", address)
}
```

### download excel & go-echarts & plot

excel操作： 阿里巴巴的开发者开源的[excelize](https://github.com/qax-os/excelize)，最低要求1.15版本

[go-echarts](https://github.com/wcharczuk/go-chart)可以方便画出图表，产出物是html文件，需要想办法转换为图片——echarts自身提供了js交互的图片下载，需要在浏览器打开才能操作

[issue 38](https://github.com/go-echarts/go-echarts/issues/38)提到这个问题——

>So the only possibility to achieve this is to:  
- use headless browser that will enter generated page and initiate the download :(
- wkhtmltoimage (or similar package)
- use external APIs that do html -> img as a service

echarts最终还是利用js进行画图，导致使用工具将html文件装换为图片时无法保存图表。最终使用[plot](https://github.com/gonum/plot)，类似与python生态的`matplotlib.pyplot`，直接画图

gonum/plot默认不支持中文字体，需要自己手动加载，参考[这里](https://github.com/gonum/plot/issues/702#issuecomment-858640740)，也可以通过网上下载解压操作，[参考](https://pkg.go.dev/gonum.org/v1/plot@v0.9.0/vg#example-package-AddFont)

```go
ttf, err := os.ReadFile("/System/Library/Fonts/Supplemental/Arial.ttf")
if err != nil {
	panic(err)
}
fontTTF, err := opentype.Parse(ttf)
if err != nil {
	log.Fatal(err)
}

arial := font.Font{Typeface: "Arial"}
font.DefaultCache.Add([]font.Face{
	{
		Font: arial,
		Face: fontTTF,
	},
})

plot.DefaultFont = arial
p := p.New()
```

### go len函数

[关于 len 函数的诡异 Go 面试题解析](https://mp.weixin.qq.com/s/1fAmtwDTc7Gv8sGilKdGTQ)里提到的例子：
```go
package main

func main() {
  var x *struct {
    s [][32]byte
  }
  
  println(len(x.s[99]))
}
```
正确答案是打印32，程序不会panic。尽管作为“一个结构体类型指针变量”x没有进行初始化。因为len为内置函数，根据[标准库文档](https://golang.org/ref/spec#Length_and_capacity)中对于`Length and capacity`的说明——

>The built-in functions `len` and `cap` take arguments of various types and return a result of type int. The implementation **guarantees** that the result always fits into an int.  
>The expression len(s) is constant if s is a string constant. The expressions len(s) and cap(s) are constants if the type of s is an array or pointer to an array and the expression s does not contain channel receives or (non-constant) function calls; in this case s is not evaluated. Otherwise, invocations of len and cap are not constant and s is evaluated.

>如果 v 的类型是数组或指向数组的指针，且表达式 v 没有包含 channel 接收或（非常量）函数调用，则返回值也是一个常量。这种情况下，不会对 v 进行求值（即编译期就能确定）。否则返回值不是常量，且会对 v 进行求值（即得运行时确定）。

因为`x.s[99]` 的类型是 `[32]byte`，以此不会求值，编译器能够在编译阶段分析出 x.s[99] 的类型是 [32]byte，且不需要对 x.s[99] 求值，因此直接返回数组的长度，即 32。具体的编译计算方法，可以参考[Go len() 是怎么计算出来的](https://mp.weixin.qq.com/s/0Twg1EWtatd2CMRAppfukw)

### go 汇编

[GO的编译执行流程](https://zhuanlan.zhihu.com/p/62922404) `go build -n main.go` (-n 不执行地打印流程中用到的命令)

- 创建临时目录，`mkdir -p $WORK/b001/`
- 查找依赖信息，`cat >$WORK/b001/importcfg << ...`
- 执行源代码编译，`/home/geb/go/pkg/tool/linux_amd64//compile ...`
- 收集链接库文件，`cat >$WORK/b001/importcfg.link << ...`
- 生成可执行文件，`/home/geb/go/pkg/tool/linux_amd64/link -o ...`
- 移动可执行文件，`mv $WORK/b001/exe/a.out main`


[GO汇编入门](https://mp.weixin.qq.com/s/pOyN1CM3op_OSeBiNOnLbg)

- AX――累加器（Accumulator），使用频度最高
- BX――基址寄存器（Base Register），常存放存储器地址
- CX――计数器（Count Register），常作为计数器
- DX――数据寄存器（Data Register），存放数据
- SI――源变址寄存器（Source Index），常保存存储单元地址
- DI――目的变址寄存器（Destination Index），常保存存储单元地址
- BP――基址指针寄存器（Base Pointer），表示堆栈区域中的基地址
- SP――堆栈指针寄存器（Stack Pointer），指示堆栈区域的栈顶地址
- IP――指令指针寄存器（Instruction Pointer），指示要执行指令所在存储单元的地址。IP寄存器是一个专用寄存器。

针对`main.go`，获取汇编

方式一：

- 使用 `go build -gcflags "-N -l" main.go` 生成对应的可执行二进制文件 
- 使用 `go tool objdump -s "main\." main` 反编译获取对应的汇编

>反编译时`"main\."` 表示只输出 main 包中相关的汇编；`"main\.main"` 则表示只输出 main 包中 main 方法相关的汇编

方式二：

- 使用 `go tool compile -S -N -l main.go` 这种方式直接输出汇编

方式三：

- 使用 `go build -gcflags="-N -l -S" main.go`直接输出汇编

关键是`gcflags`对应的参数——

> `-l` 禁止内联   
> `-N` 编译时，禁止优化  
> `-S` 输出汇编代码  

### go time format 

[Go 的时间格式化为什么是 2006-01-02 15:04:05？](https://mp.weixin.qq.com/s/f3VWaUGseWX6NCA2KGT-pw)， 参考`src/time/format.go`源码 

```
1: month (January, Jan, 01, etc)
2: day
3: hour (15 is 3pm on a 24 hour clock)
4: minute
5: second
6: year (2006)
7: timezone (GMT-7 is MST)

ANSIC       = "Mon Jan _2 15:04:05 2006"
```
进行数字排序的结果，方便记忆“一月二号三点四分五秒2006年西七区”，英文的书写格式为`Mon, 02 Jan 2006 15:04:05 -0700` ([RFC1123](https://tools.ietf.org/html/rfc1123)带有数字时区的表达方式)


### go mod vendor 

[Vendor dependencies of a module in Go ](https://golangbyexample.com/vendor-dependency-go/)，使用mod模式，生成vendor文件夹`go mod vendor`将项目依赖的所有依赖保存到工程的vendor文件夹目录下，增加新的依赖之后，再次执行此命令，添加新的依赖。执行`go mod vendor -v`查看所有的依赖信息

### http head 

[GoLang: Case-sensitive HTTP Headers with net/http](https://dhdersch.github.io/golang/2016/08/11/golang-case-sensitive-http-headers.html) 
```go
// use this to avoid the key being canonicalized by  textproto.CanonicalMIMEHeaderKey. 
req.Header["kbn-version"] = []string{"6.4.3"}
// the key will be canoicalized as "Content-Type"
req.Header.Set("content-type", "application/json")
```
### 版本升级

[升级原理实现的源码: dl项目](https://github.com/golang/dl)，[升级原理说明](https://mp.weixin.qq.com/s/jEhX5JHAo9L6iD3N54x6aA) 

操作步骤： 

```
go get golang.org/dl/go<version>  // 其中 <version> 替换为你希望安装的 Go 版本
go<version> download   // 和上面一样，<version> 是具体的版本
# 例如
go get golang.org/dl/go1.14.15
go1.14.15 download
```

- `GOPATH`相当于工作目录，即使go mod启用后，也是在`$GOPATH/pkg/mod`目录下维护。上述执行`go get`下载的可执行文件也是在`GOPATH/bin`路径下，所以可以在下一步执行`go1.14.15`。[go环境变量含义官方说明](https://golang.org/cmd/go/#hdr-Environment_variables)
- `GOROOT`相当于go的安装目录

执行`go1.14.15 download`时，相当于执行的`https://github.com/golang/dl/blob/master/go1.14.15/main.go`方法，内部实现为`version.Run("go1.14.15")`，会单独处理`download`命令，其他命令则统一执行`runGo(root)`方法

多版本使用——(windows环境下，就添加环境变量吧)
- 将 ~/sdk/go1.16.4/bin/go 加入 PATH 环境变量（替换原来的）；
- 做一个软连，默认 go 执行 go1.16.4（推荐这种方式），不需要频繁修改 PATH；
- 移动 go1.16.4 替换之前的 go（不推荐）；

```
# 下载tar包，定义文件目录 /usr/local/go
# go 安装&升级
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.17.6.linux-amd64.tar.gz
# 添加到环境变量
export PATH=$PATH:/usr/local/go/bin
```

### 业务梳理一

系统涉及内容：graphql(gqlgen)方式交互；restful api交互(包括中间件使用)；gorm数据库操作。

- UI：业务信息
- 对应`"@/components/Setting/professional"`挂载前befoerMount调用`this.getOrgBusiness();`
- 上述方法声明在`settingModule/GetOrgBusiness`(对应的js graph方法声明`api/modules/settings.js`) 
- 查询的query方法`getOrgBusiness`

进入后端逻辑——

- 所有的query声明都在`query.graphqls`中定义，对应方法`orgBusiness: [OrgBusiness]`
- 使用自动生成的resolver: `query.resolvers.go`(这里是如何知道使用store的get方法就可以完成？)
- 对orgTree的接口方法实现中完成：
  - 查询数据库，解析格式化内容返回格式化数据；
  - 同时defer执行API调用：查询部门接口，解析结果，保存更新记录(只需要保持两条对应记录即可——只涉及到两个部门)

### 文件上传

本地文件上传移植后工作正常(不再维护的SDK还是可以正常工作——具体的原理后续再研究)。

仿现有逻辑(添加URL前缀作为分类；定义处理请求的方法)添加上传文件的API接口——
```
func InitRouter(router *mux.Router) {
	poolRoute := router.PathPrefix("/pool").Subrouter()
	poolRoute.UseEncodedPath()

	poolRoute.HandleFunc("/file/upload", UploadFile)

}
```
[上传文件获取方式](// https://tutorialedge.net/golang/go-file-upload-tutorial/)，对比起来Java看起来是简单不少

```
#获取form文件
file, handler, err := r.FormFile("file")

#获取参数列表
vars := mux.Vars(r)
```

[查看对象类型](https://yourbasic.org/golang/find-type-of-object/)(另外这个站点有很多有用的基础信息)

```
var x interface{} = []int{1, 2, 3}
xType := reflect.TypeOf(x)
xValue := reflect.ValueOf(x)
fmt.Println(xType, xValue) // "[]int [1 2 3]"
```

本身的上传接口设计是“充分”的：本地文件上传，利用反射处理为string格式；如果是字节流，相当于slice类型（一开始传递的参数错误：将`multipart.File`对象直接传递进去，被认为是struct类型）

字符串替换——

```
func nowAsString() string {
	// .000将保留末尾的0值；.999将忽略末尾的0值
	now := time.Now().Format("2006-01-02 15:04:05.000")
	// 两个一组，使用后者替换前者
	replacer := strings.NewReplacer("-", "", ":", "", ".", "", " ", "")
	return replacer.Replace(now)
}
```

### Context

[context](https://golang.org/pkg/context/#example_WithValue)传递数据。应用场景——

进行用户认证，每次API调用都需要验证用户是否登录——通常都会使用SSO的方式，每个请求获取到COOKIE中的token，然后在中间层进行有效性验证。获取用户信息后可以保持在context中。然后继续执行请求处理`next.ServeHTTP(w, r)`

```
func WithValue(parent Context, key, val interface{}) Context
```

>WithValue returns a copy of parent in which the value associated with key is val.
>
>Use context Values only for request-scoped data that transits processes and APIs, not for passing optional parameters to functions.
>
>The provided key must be comparable and should not be of type string or any other built-in type to avoid collisions between packages using context.

- 不建议直接使用string或内置的数据类型作为key——通常重新封装一个自定义的类型


```
type favContextKey string

f := func(ctx context.Context, k favContextKey) {
	if v := ctx.Value(k); v != nil {
		fmt.Println("found value:", v)
		return
	}
	fmt.Println("key not found:", k)
}

k := favContextKey("language")
ctx := context.WithValue(context.Background(), k, "Go")

f(ctx, k)
f(ctx, favContextKey("color"))
```

>Value returns the value associated with this context for key, or nil if no value is associated with key. Successive calls to Value with the same key returns the same result.

- key对应的值通常也会定义为结构体类型，通过key获取值时，使用结构体指针即可，例如`ctx.Value(userKey).(*User)`

```
type User struct {...}

func FromContext(ctx context.Context) (*User, bool) {
	u, ok := ctx.Value(userKey).(*User)
	return u, ok
}
```


### GORM

[gorm](https://github.com/go-gorm/gorm/)，字节[jinzhu](https://github.com/jinzhu)开源的ORM（Object Relational Mapping）项目。

- 使用`Update`方法时需要注意：`WARNING when update with struct, GORM will not update fields that with zero value` 

如果直接使用struct类型(表的映射对象)进行更新时，零值将被忽略。加入字段之前有值，现在需要更新为零值。这是直接使用struct进行更新是无效的。零值被忽略之后，相当于字段还保留有旧值。

这种情况下，需要将struct转换为`map[string]interface{}`之后再更新。使用`structs.Map(*struct)`将struct转换为map格式再调用`Update`方法即可，[参考这里](https://stackoverflow.com/questions/23589564/function-for-converting-a-struct-to-map-in-golang)。这个问题也是类似情况[Update method does not update zero value](https://stackoverflow.com/questions/64330504/update-method-does-not-update-zero-value)


### converting argument $1 type: unsupported type []int, a slice of int

[go sql报错](https://blog.csdn.net/xiaomin1328/article/details/114269917)：

>当…Type做为参数时，本质上函数会把参数转化成一个Type类型的切片，因而在上述代码中，Service层调以可变参数形式传入一个参数，在Exec中的args就已经是[]interface{}类型了，若是直接把args做为func (s *Stmt) Exec args …interface{}) (Result, error)的参数，对于Exec来讲，收到的args就只有一个长度为1的切片，其元素类型为[]interface{}，因而就有了上述的报错，解决办法很简单，就是在一个slice后加上…，这样就能把它拆包成一个可变参数的形式传入函数。

```golang
res, err := mt.Exec(msg...)//正确引用exec
res, err := mt.Exec(msg)//错误引用exec
```

#### Singular Table

[Mapping between Gorm and database (singular and plural) table structures](https://www.programmersought.com/article/53034028334/)

数据库连接设置了以下属性的含义——对应变量在`model_struct.go`文件中的` GetModelStruct()`方法和`TableName(db *DB) string`中应用

```go
db.SingularTable(true)
```

假设定义了——

```go
type User struct {
   Id   int    `json:"id"`
   Name string `json:"name"`
   Age  int    `json:"age"`
}

type Users struct {
   Id   int    `json:"id"`
   Name string `json:"name"`
   Age  int    `json:"age"`
}
```

1. 数据库中没有`users`表也没有`user`表，执行`DB.AutoMigrate(&User{})`或`DB.AutoMigrate(&Users{})`—— 
    - 没有上述设置，则都自动创建`users`表(自动设置为复数形式)；
    - 有上述设置时，则自动分别创建`user`表和`users`(遵守单数形式)


2. 数据库中只有`users`表，没有`db.SingularTable(true)`设置

以下操作会将数据都添加到`users`表；设置之后，第一行将保存提示`user`表不存在
```
DB.Create(&User{Name: "Li", Age: 5})  // 
DB.Create(&Users{Name: "Li", Age: 5}) // 
```

3. 数据库中只有`user`表，没有`db.SingularTable(true)`设置，相同的添加数据的操作都会失败，提示`users`表不存在

```
DB.Create(&User{Name: "Li", Age: 5}) // Table 'users' doesn't exist
DB.Create(&Users{Name: "Li", Age: 5}) // Table 'users' doesn't exist
```

总结：gorm默认使用复数映射，go代码的单数、复数struct形式都匹配到复数表中，创建表、添加数据时都是如此。指定了`db.SingularTable(true)`之后，进行严格匹配。

#### 连表查询

`db.SingularTable(true)`一开始我以为这意味着只能操作单表- -|(如果有这种限制，相当于数据库表彼此没有“关系”了)导致需要连表查询时，只能手动进行数据整合，查出一个集合，再对集合中的每一条记录进行扩展(再查另外一个表获取关系)。耗时自然就无法接受了。总体梳理只有3k左右的数据，做上述操作(相当于先模糊查询一次，然后再连续查询3k次)之后，耗时20s以上

实际上gorm是支持连表查询，类似——相当于`PoolApp`表关联`leftJoin`了`pool_week`表表，然后再定义出要联合查询的字段(`dbSelect`)、查询的条件(`where`)、结果偏移量、查询总数等信息之后。就可以依次返回需要的查询结果。（相当于拆解sql语句，这样才能称为orm撒~）

```
leftJoin := "left join pool_week on pool_app.app_id = pool_week.app_id"
GetDB(ctx).Model(dbmodel.PoolApp{}).Joins(leftJoin).Select(dbSelect).Where(where).Order(order).Offset(offset).Limit(limit).Scan(&res)
```


### 初始化方法

- 最权威的解释来自官方spec: [Program initialization and execution](https://golang.org/ref/spec#Program_initialization_and_execution)  
- 简单解释可参考[effective go: initialization](https://golang.org/doc/effective_go#initialization)

目前理解——

- 安装程序入口进行初始化：按文件代码组织顺序，初始化所有引入的、自定义的文件中的初始化过程(这意味这当前程序用不到的代码不会被初始化——例如过程包括多个main包，每个主程序依赖不同的代码文件)
- 初始化变量声明：常量，变量(根据声明的顺序；但如果变量依赖另外的变量，则先计算依赖的变量内容，而且如果依赖的变量来自多变量声明，则此多变量也会被优先初始化)如果有循环依赖的变量声明，则程序无效
- 执行源码文件中定义的无参数的`init()`方法：可以包含多个的`init()`方法(不同的文件)；一个文件中也可以包含多个`init()`方法；各个init方法的执行顺序不确保有序，所以应当让各个方法做独立的逻辑
- 进入主程序`main`方法的逻辑


### gqlgen 

#### go get and gqlgen downgrade

目前(2021-04-30)使用`go get github.com/99designs/gqlgen`安装的版本是0.13.0；目前项目中使用的版本是0.11.3；新版本会报错`validation failed: packages.Load: /xxx/xx/xx: WithPathContext not declared by package graphql`

`go get`支持指定版本`go get github.com/99designs/gqlgen@v0.11.3`安装指定版本(默认按照到`GOPATH`路径下的bin目录下)。

> go get github.com/99designs/gqlgen@v0.11.3   
> go: cannot use path@version syntax in GOPATH mode

注意需要打开go mod模式： 
- (windows) `set GO111MODULE=on`
- (unix*) `export GO111MODULE=on`

如果安装时的依赖出现`unrecognized import path... connect: connection refused`的错误，是因为本地代理未启用https服务，使用`insecure`模式进行独立安装即可`go get -insecure git.xxx.com/medusa/crd` 

```
unrecognized import path "git.xxx.com/medusa/crd": https fetch: Get "https://git.xxx.com/medusa/crd?go-get=1": dial tcp 10.16.210.58:443: connect: connection refused
```


[Building a GraphQL API in Go using gqlgen](https://medium.com/weareservian/building-a-graphql-api-in-go-using-gqlgen-f7a42eba2193)

依次执行以下命令，则完成了[gqlgen站点的getting started](https://gqlgen.com/getting-started/)的内容。

```
cd mygplprj/src
go mod init gitlab.com/jigar_xyz/mygplprj
cd cmd/go-graphql
go get github.com/99designs/gqlgen
go run github.com/99designs/gqlgen init --verbose
```

执行`go run ./server.go`就可以起到一个GraghQL站点。目录结构说明——

```
├── gqlgen.yml               - The gqlgen config file, knobs for controlling the generated code.
├── graph
│   ├── generated            - A package that only contains the generated runtime
│   │   └── generated.go
│   ├── model                - A package for all your graph models, generated or otherwise
│   │   └── models_gen.go
│   ├── resolver.go          - The root graph resolver type. This file wont get regenerated
│   ├── schema.graphqls      - Some schema. You can split the schema into as many graphql files as you like
│   └── schema.resolvers.go  - the resolver implementation for schema.graphql
└── server.go                - The entry point to your app. Customize it however you see fit
```

- `gqlgen.xml` 配置文件，说明如何自动生成代码。更详细的介绍参考[config](https://gqlgen.com/config/)。结合目前工程中的`gqlgen.xml`可以更好理解
- `generated/generated.go` 根据配置自动生成的服务端文件，相当于GraphQL的运行时。可在上述配置文件中做对应的声明(文件名，文件地址，所属的包)
- `models_gen.go` 对应各个`*.graphqls`文件绑定的model，gqlgen会自己进行匹配，如果找到则忽略(相当于人工写)，否则会根据定义自动生成。[Generated models required to build the graph. Often you will override these with your own models. Still very useful for input types.](https://www.howtographql.com/graphql-go/1-getting-started/)
- `resolver.go` 根类型resolver
- `schema.graphqls` 定义的schema，可以分割到多个不同的文件中
- `schema.resolvers.go` 对定义的实现

[Introducing gqlgen: a GraphQL Server Generator for Go](https://en.99designs.de/blog/engineering/gqlgen-a-graphql-server-generator-for-go/) 项目开发人员对`gqlgen`的介绍

在Windows环境下折腾半天，Mac环境下并不复现- -|| 还没了解原理的情况下，下搁这儿吧。

>windows环境下 执行`gqlgen generate`时，总是报错类似——   
>`/graph/model.CronJob failed: unable to build object definition: unable to find type git.gebtest.com/gebing/testprj`的错误  
>或者`/graph/model.JobConditioned: unable to build object definition: unable to find type gebtest.com/gebing/testprj`  
>配置文件中指定了`autobind`和`models`的配置。但项目中没找到  

奇怪的是即使是windows环境下不能识别对前缀`gebtest.com/gebing/testprj`的定义；`/graph/model.JobConditioned`这个model实际不存在(项目里没这个定义)

关于`unable to find type`的错误，也只有类似官方这个资料：[#issue911: gqlgen Autobind internal package - unable to find type](https://github.com/99designs/gqlgen/issues/911)，看起来跟上面的情况关系不大

#### wsl相关：不能识别autobind的配置

使用wsl环境，从wsl里访问windows环境下的项目，执行`gqlgen generate`时提示`unable to load module_name/graph/model - make sure you're using an import path to a package that exists`，即使修改为绝对路径，类似`/mnt/d/project/prjname/graph/model`也无法识别。源码的[config_test.go](https://github.com/99designs/gqlgen/blob/master/codegen/config/config_test.go)(unable to load ../chat - make sure you're using an import path to a package that exists)文件中有这种场景。 ~~暂时先忽略这个问题，涉及到两个不同操作系统互相访问的问题，识别不到也正常。~~ mac环境也出现这个现象，看来还没定位到真实的原因

在wsl环境里重新clone当前项目——而不是直接访问windows环境下的此项目，可以正常执行。应用层没有问题，上面的异常涉及到了系统层——重新clone的方法也不好使了这次。

使用`export GOFLAGS=-mod=vendor`模式运行，需要确保vendor文件夹下的内容是OK的。重新先执行`go mod vendor`之后再执行`gqlgen generate`，问题消失。但实际上`go mod vendor`并没有改变什么vendor文件夹下的内容啊？只是在不同的平台下，几个文件的换行符不同而已，诡异。这个问题还是没定位到最终原因

如果提示`could not import C (no metadata for C) `，安装gcc即可`sudo apt install gcc`


#### 接口定义与实现

- 先定义接口`interface`，以及接口中可能用到的结构体。例如`pkg/todo_service.go`

```
type ToDoItem struct {
	Id        string
	Text      string
	IsDone    bool
	CreatedOn time.Time
	UpdatedOn *time.Time
}

type ToDo interface {
	Initialise() error
	Create(text string, isDone bool) (*string, error)
	Update(id string, text string, isDone bool) error
	List() ([]ToDoItem, error)
}
```

- 定义接口的实现对象结构体。让此结构体实现上述所有的接口方法。go语言中**任意实现了所有接口中定义的方法的对象，都默认实现了此接口。**例如`pkg/imp/todo_service.go`

```

type ToDoImpl struct {
	DbUserName string
	DbPassword string
	DbURL      string
	DbName     string
}

func (t *ToDoImpl) Initialise() error {...}

func (t *ToDoImpl) Create(text string, isDone bool) (*string, error) {...}

func (t *ToDoImpl) Update(id string, text string, isDone bool) error {...}

func (t *ToDoImpl) List() ([]todo.ToDoItem, error) {...}
```

#### graphql请求与响应

- 通常就两类主要的请求：一种为查询`query`；一种为变更`mutation`。通常在在`schema.graphl`文件中定义

```
schema {
    query: MyQuery
    mutation: MyMutation
}

type MyQuery {
    todos: [Todo!]!
}

type MyMutation {
    createTodo(todo: TodoInput!): Todo!
}
```

- 响应会在对应的`schema.resolver.go`文件中实现。默认有一个`resolver.go`文件，可以一行实现`type Resolver struct{}`。关键的是对请求和响应的声明:

```
// MyMutation returns generated.MyMutationResolver implementation.
func (r *Resolver) MyMutation() generated.MyMutationResolver { return &myMutationResolver{r} }

// MyQuery returns generated.MyQueryResolver implementation.
func (r *Resolver) MyQuery() generated.MyQueryResolver { return &myQueryResolver{r} }

type myMutationResolver struct{ *Resolver }
type myQueryResolver struct{ *Resolver }

```

- 然后在对应的resolver中实现graphql文件中的各种查询和变更的方法，gqlgen默认实现的方法只包含一句`panic(fmt.Errorf("not implemented"))`

```
func (r *myQueryResolver) Todo(ctx context.Context, id string) (*model.Todo, error) {...}
func (r *myMutationResolver) CreateTodo(ctx context.Context, todo model.TodoInput) (*model.Todo, error) {...}
```

在对应resolver具体的方法实现中调用上述“接口方法”。完成对请求的最终响应。

最终对graphql的请求处理，`gqlgen`会最终生成`generated.go`(具体名称有配置文件定义)作为运行时响应

这样就算完成一个完整套路。更多细节学习官方文档：[Introduction to GraphQL](https://graphql.org/learn/)，[GraphQL 入门](https://graphql.cn/learn)

#### 混合使用

根据[How to configure gqlgen using gqlgen.yml](https://gqlgen.com/config/)中的描述，`models`字段的含义如下——

> This section declares type mapping between the GraphQL and go type systems
> 
> The first line in each type will be used as defaults for resolver arguments and
> modelgen, the others will be allowed when binding to fields. Configure them to
> your liking


参考[Create Your First Simple GraphQL Golang Application with GQLGen](https://medium.com/@ktrilaksono/create-your-first-simple-graphql-golang-application-with-go-gqlgen-793e11dc5fc4)中的实践——

```
schema:
- schema.graphql
exec:
  filename: generated.go
models:
  Person:
    model: github.com/antoooks/go-graphql-tutorial/models.Person
  Pet:
    model: github.com/antoooks/go-graphql-tutorial/models.Pet
resolver:
  filename: resolver.go
  type: Resolver
autobind: []
```

会优先使用`models`中定义的映射关系；如果没有找到，则会根据自动匹配规则，创建新的resolver进行实现。

例如，以下查询表示的查询方法为`app`，返回的对象再做“次级选择”(sub-selection)，返回`users`对象的username, role两个字段。这意味着查询方法`app`(这个方法后台定义返回一个`App`对象)返回的对象里**一定**包含一个次级对象查询`users`的实现。

```
query($id: String!){
    app (id: $id) {
      users {
        username
        role
      }
    }
  }
```

验证一下：采用默认的配置文件，在`App.graphqls`文件中添加一个新的查询方法——
```
type App {
    # The app id.
    id: Int
    # The namespace of the app
    namespace: String!
    # The name of the app
    name: String!
    # The users belong to the app
    users: [AppUser]
    # 新增一个方法
    geb: [Gebitang]
}

type Gebitang {
    id: Int!
    name: String!
    grade: String
}
```

如果在配置文件中指定了`App`对应的model——`github.com/antoooks/go-graphql-tutorial/models.App`，需要在`github.com/antoooks/go-graphql-tutorial/models`目录下的`App.go`文件中定义对应的结构体和方法

```
func (a *App) Geb(ctx context.Context) ([]*Gebitang, error) {
 g := make([]*Gebitang, 0)
 ...
 return g, nil
}

Gebitang struct {
  Id    int
  Name  string
  Grade string
}
```

这种情况下执行`gqlgen generate`时，只会更新最终的`generated.go`文件——用于生产运行时；如果没有更新`App.go`的情况执行`gqlgen generate`，则会根据配置文件自动更新——

- `app.resolver.go` 自动生成查询方法`Geb`和`GetInfo`方法，需要手动做进一步的实现，如下。
- `models_gen.go` 中自动生成对`Gebitang`结构体的声明，如下。

```
# in app.resolver.go
func (r *appResolver) Geb(ctx context.Context, obj *App) ([]*Gebitang, error) {
 panic(fmt.Errorf("not implemented"))
}

func (r *appResolver) GetInfo(ctx context.Context, obj *App) ([]*GebInfo, error) {
 panic(fmt.Errorf("not implemented"))
}

# in models_gen.go 
type Gebitang struct {
 ID    int     `json:"id"`
 Name  string  `json:"name"`
 Grade *string `json:"grade"`
}
```

## archive 2

*update@2021-04-30* 又是快一年过去，好像还是没啥进步乜。再来~

尴尬了，这篇的日期写的是`2018-01-03`，实际上是近三年前的帖子了。Go 1.9发布的时候就信誓旦旦的要进行学习。结果到今天还是无疾而终的样子，半途而废了好久。捞上来，看看这次可以坚持多久。

[Mixin 大群部署完全教程](https://dbarobin.com/2019/05/19/mixin-super-group/) 拿这个练习，前后端一起端。

### 静态站点

[Serving Static Sites with Go](https://www.alexedwards.net/blog/serving-static-sites-with-go)

```go
package main

import (
  "log"
  "net/http"
  "os"
)

func main() {
  // fs := http.FileServer(HTMLDir{http.Dir("./static")})
  fs := http.FileServer(http.Dir("./static")) //absolute path if necessary
  http.Handle("/", fs)

  log.Println("Listening on :3000...")
  err := http.ListenAndServe(":3000", nil)
  if err != nil {
    log.Fatal(err)
  }
}

type HTMLDir struct {
	d http.Dir
}

// https://stackoverflow.com/a/57281956/1087122
func (d HTMLDir) Open(name string) (http.File, error) {
	// Try name as supplied
	f, err := d.d.Open(name)
	if os.IsNotExist(err) {
		// Not found, try with .html
		if f, err := d.d.Open(name + ".html"); err == nil {
			return f, nil
		}
	}
	return f, err
}
```

- Hugo生成静态站点
- 使用上面的go脚本(保存为blog.go，编译`go build blog.go`，生成blog可执行文件)即可做为web服务器
- `sudo certbot --nginx`自动更新配置(确保：nginx中`server_name`有对应的真实有效域名，并且域名配置了对应的公网IP（域名解析）)，还自动添加了301跳转
- 欢迎访问备份blog站点[不想注册](https://blog.buxiangzhuce.com)
- 设置为service服务，下面的内容保存为`/etc/systemd/system/blog.service`，执行`systemctl restart blog.service`期待服务
- 执行`journalctl -u blog.service`查看此服务的log信息

```
[Unit]
Description=Blog Daemon
After=network.target

[Service]
Type=simple
ExecStart=/home/geb/blog
Restart=on-failure

[Install]
WantedBy=multi-user.target

```

```
server {

    server_name blog.buxiangzhuce.com;
    index index.html index.htm;
    charset utf-8;

    location / {
        proxy_pass http://127.0.0.1:1313;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Port $server_port;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/blog.buxiangzhuce.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/blog.buxiangzhuce.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}

server {
    if ($host = blog.buxiangzhuce.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

   server_name blog.buxiangzhuce.com;
    listen 80;
    return 404; # managed by Certbot

}
```

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

[wire guide](https://github.com/google/wire/blob/master/docs/guide.md) 

[依赖两个相同的provider？](https://github.com/google/wire/blob/master/docs/faq.md#what-if-my-dependency-graph-has-two-dependencies-of-the-same-type)

### go ldflags

[Using `ldflags` with `go build`](https://www.digitalocean.com/community/tutorials/using-ldflags-to-set-version-information-for-go-applications)

`ld` stands for linker, the program that links together the different pieces of the compiled source code into the final binary. 

`ldflags`, stands for linker flags. It passes a flag to the underlying Go toolchain linker, [cmd/link](https://golang.org/cmd/link), that allows you to change the values of imported packages at build time from the command line.

使用方式为：

```
# 格式
go build -ldflags="-flag"
# 示例
go build -ldflags="-X 'package_path.variable_name=new_value'"
# 实例
go build -ldflags="-X 'main.Version=v1.0.0'"
#复杂实例
go build -v -ldflags="-X 'main.Version=v1.0.0' -X 'app/build.User=$(id -u -n)' -X 'app/build.Time=$(date)'"
```

- 外层使用双引号，确保传递的flag中的内容即使包含空格也不截断命令；
- key-value值使用单引号
- 要改变的变量需要是包级别的string类型变量。不能是const类型
- 变量是否export都可以（大小写开头的变量都支持）

进一步可以使用`nm`工具查找编译文件中的`symbols`。（包名中不能包含非ASCII码，引号`"`和百分号`%`）

在make文件中使用——[](https://gist.github.com/marz619/e91796afa5a951c0aa4595d7e73d78d0)

`main.go` 

```
package main

var (
	version string
	date    string
)

func init() {
	if version == "" {
		version = "no version"
	}
	if date == "" {
		date = "(Mon YYYY)"
	}
}

func main() {
	println(version, date)
}
```

`makefile:`

```
version=0.0.1
date=$(shell date -j "+(%b %Y)")
exec=a.out

.PHONY: all

all:
	@echo " make <cmd>"
	@echo ""
	@echo "commands:"
	@echo " build          - runs go build"
	@echo " build_version  - runs go build with ldflags version=${version} & date=${date}"
	@echo ""

build: clean
	@go build -v -o ${exec}

build_version: check_version
	@go build -v -ldflags '-X "main.version=${version}" -X "main.date=${date}"' -o ${exec}_${version}

clean:
	@rm -f ${exec}

check_version:
	@if [ -a "${exec}_${version}" ]; then \
		echo "${exec}_${version} already exists"; \
		exit 1; \
	fi;
```

[Setting Go variables from the outside](https://blog.cloudflare.com/setting-go-variables-at-compile-time/)

在`go run`命令中也可以直接使用（因为会默认先执行`go build`)  

`go run -ldflags="-X main.who CloudFlare" hello.go` 


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

[usage of `go mod vendor`](https://golang.org/ref/mod#go-mod-vendor)

The `go mod vendor` command constructs a directory named vendor in the main module's root directory that contains copies of all packages needed to support builds and tests of packages in the main module. 


<img src="data:image/png;base64, iVBORw0KGgoAAAANSUhEUgAAAQAAAAEACAIAAADTED8xAAAE8UlEQVR4nOzX36cQBgOA4XOONjMzMzMzMzMzM0mSJEmSJEmSJEmSJEmSJEmSJEmSJEmSJEmSJEmSJEmSJEmSJEmSJP34/ovvjPe5ey7f23fEwMCXwcGBr74aGBwc+PRpgDnloWEvYB5ODw19+Q/VMP9/PTTsBczD6REjPAB37QG47a+/9gDctQfgtr/5xgNw1x6A2/72Ww/AXXsAbvu77zwAd+0BuO3vv/cA3LUH4LZ/+MEDcNcegNv+8UcPwF17AG77p588AHftAbjtn3/2ANy1B+C2f/nFA3DXHoDb/vVXD8BdewBu+7ffPAB37QG47d9/9wDctQfgtv/4wwNw1x6A2/7zTw/AXXsAbvuvvzwAd+0BuO2///YA3LUH4Lb/+ccDcNcegNv+918PwF17AG575EgPwF17AG571CgPwF17AG579GgPwF17AG57zBgPwF17AG577FgPwF17AG573DgPwF17AG57/HgPwF17AG57wgQPwF17AG574kQPwF17AG570iQPwF17AG578mQPwF17AG57yhQPwF17AG576lQPwF17AG572jQPwF17AG57+nQPwF17AG57xgwPwF17AG575kwPwF17AG571iwPwF17AG579mwPwF17AG57zhwPwF17AG577lwPwF17AG573jwPwF17AG57/nwPwF17AG57wQIPwF17AG574UIPwF17AG570SIPwF17AG578WIPwF17AG57yRIPwF17AG576VIPwF17AG572TIPwF17AG57+XIPwF17AG57xQoPwF17AG575UoPwF17AG571SoPwF17AG579WoPwF17AG57zRoPwF17AG577VoPwF17AG573ToPwF17AG57/XoPwF17AG57wwYPwF17AG5740YPwF17AG570yYPwF17AG5782YPwF17AG57yxYPwF17AG5761YPwF17AG572zYPwF17AG57+3YPwF17AG57xw4PwF17AG57504PwF17AG571y4PwF17AG57924PwF17AG57zx4PwF17AG57714PwF17AG573z4PwF17AG57/34PwF17AG77wAEPwF17AG774EEPwF17AG770CEPwF17AG778GEPwF17AG77yBEPwF17AG776FEPwF17AG772DEPwF17AG77+HEPwF17AG77xAkPwF17AG775EkPwF17AG771CkPwF17AG779GkPwF17AG77zBkPwF17AG777FkPwF17AG773DkPwF17AG77/HkPwF17AG77wgUPwF17AG774kUPwF17AG770iUPwF17AG778mUPwF17AG77yhUPwF17AG776lUPwF17AG772jUPwF17AG77+nUPwF17AG77xg0PwF17AG775k0PwF17AG771i0PwF17AG779m0PwF17AG77zh0PwF17AG777l0PwF17AG773j0PwF17AG77/n0PwF17AG77wQMPwF17AG774UMPwF17AG770SMPwF17AG778WMPwF17AG77yRMPwF17AG776VMPwF17AG772TMPwF17AG77+XMPwF17AG77xQsPwF17AG775UsPwF17AG771SsPwF17AG779WsPwF17AG77zRsPwF17AG777VsPwF17AG773TsPwF17AG77/XsPwF17AG77wwcPwF17AG7740cPwF17AG7782cPwF17AE77fwEAAP//oVuGohAcKHYAAAAASUVORK5CYII=">

As you append to a slice, its capacity [doubles in size](https://stackoverflow.com/a/45423692/1087122) every time it exceeds its current capacity.
