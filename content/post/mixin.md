+++
title = "Go Mixin Go"
description = "mixin kernel"
tags = [
    "mixin",
    "go"
]
date = "2020-06-30"
topics = [
    "mixin",
    "go"
]
toc=true
+++

推荐Go语言入门：[Go编程时光](http://golang.iswbm.com/en/latest/index.html)对应的微信公众号系列[Go 语言入门指南](http://mp.weixin.qq.com/mp/homepage?__biz=MzU1NzU1MTM2NA==&hid=1&sn=a016a47829bf9148498adf7195b850b5&scene=18#wechat_redirect)

文章写的通俗易懂，很快就可以过一遍Go语言学习的最小必要知识点。——先“眼到”，至少可以先“读懂”开源项目的代码，然后尝试在开源项目上捣鼓，锻炼“手到”的能力。

<!--more-->

[rum feed](https://github.com/okdaodine/rum-feed)使用的[在线postgresql服务](https://www.elephantsql.com/)和[Quorum open node](https://node.rumsystem.net/)。

## 多版本管理go

```shell
#命令以安装gvm：

bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)

#安装完成后，关闭并重新打开终端，或者运行以下命令以使gvm生效：
source ~/.gvm/scripts/gvm

gvm listall
#选择要安装的Go版本，并执行以下命令来安装指定版本的Go：
gvm install <version>
#例如，要安装Go 1.17版本，可以运行：
gvm install go1.17
# 安装完成后，您可以使用以下命令来切换到已安装的Go版本：
gvm use <version>
#例如，要切换到Go 1.17版本，可以运行：
gvm use go1.17

```

## 多版本管理java

```shell
# 在Linux系统上安装和管理多个Java版本，可以使用工具管理器来简化此过程。以下是使用sdkman的步骤：

curl -s "https://get.sdkman.io" | bash

#安装完成后，关闭并重新打开终端，或者运行以下命令以使sdkman生效：


source "$HOME/.sdkman/bin/sdkman-init.sh"

#列出可用的Java版本：
sdk list java

#选择要安装的Java版本，并执行以下命令以安装指定版本的Java：
sdk install java <version>
#例如，要安装Java 11版本，可以运行：
sdk install java 11.0.12-zulu

#安装完成后，您可以使用以下命令来切换到已安装的Java版本：
sdk use java <version>

#例如，要切换到Java 11版本，可以运行：
sdk use java 11.0.12-zulu

#sdkman只对当前用户有效，因此每个用户可以独立地管理自己的Java版本。

```

## rum节点

转眼两年多了。重新运行rum全节点，文档比较落后，最新源码编译（需要使用go1.19.x版本，安装的1.21版本不支持，所以才使用多版本go环境）的已经使用文档正常启动。单独获取的可执行文件。`api/v1/node`可以get回来node信息。远程连接是提示jwt信息缺失。

需要理由cli命令`.quorum jwt create chain name`先生成jwt token并保存到配置文件`config/peer_options.toml`中。也可以使用help参数一步一步查看使用说明。

可以源码编译了，启动参数有点微调。启动参数需要指定apiHost，否则默认启动的是127.0.0.1，只允许本地连接（netstat -ano |grep port)之后才发现原因（远程连接一直不好使，定位到连接不通的现象）还是看代码最靠谱，根据说明文档启动成功[2024-02-29]


```shell
Usage:
  quorum jwt create chain [flags]

Flags:
  -c, --configdir string    config directory (default "config")
  -p, --peername string     peer name (default "peer")
  -n, --name string         name of the node jwt
  -d, --duration duration   duration of node jwt (default 8760h0m0s)
  -h, --help                help for chain

# 最新的起点命令类似——
# run fullnode of quorum
# keystorepass 要改
# listen/apiport 要改，注意，三个 端口不一样，且 apiport 是 api 专用的
# peer 不要改！

PSW=password
nodepath=owner

./quorum fullnode \
    --keystorepwd=$PSW \
    --keystoredir=keystore/$nodepath \
    --configdir=config/$nodepath \
    --datadir=data/$nodepath \
    --peer=/ip4/101.42.141.118/tcp/62777/p2p/16Uiu2HAm9uziCEHprbzJoBdG9uktUQSYuFY58eW7o5Dz7rKhRn2j \
    --peer=/ip4/94.23.17.189/tcp/62777/p2p/16Uiu2HAm5waftP3s4oE1EzGF2SyWeK726P5B8BSgFJqSiz6xScGz \
    --listen=/ip4/0.0.0.0/tcp/62764 \
    --listen=/ip4/0.0.0.0/tcp/62765/ws \
    --apihost=0.0.0.0 \
    --apiport=62763 \
    --log-compress=true \
    --log-max-age=7 \
    --log-max-backups=100 \
    --log-max-size=10 \
    --logfile=logs/$nodepath/quorum.log \
    --enabledevnetwork=false \
    --loglevel=info >rum.log  2>&1 &

```

运行成功，目前去中心微博的数据大约1.4G。

## apk重签名


- 准备好重签名的文件 

```shell
keytool -genkey -v -keystore your_keystore_name.keystore -alias your_alias_name -keyalg RSA -keysize 2048 -validity 365
```

- 下载[apktool](https://github.com/iBotPeaches/Apktool)，配置好执行环境
- 将apk文件解压`java -jar apktool.jar d -o output_dir name.apk`解压到对应的文件夹
- 编辑对应的文件

```shell
# original\META-INF 文件夹中存放的是原本的签名信息
# apktool.yml 是反编译后生成的文件

# 0. 修改应用名称 res\value\string.xml 文件中的 appName
# 1. AndroidManifest.xml 的 包名package="your.new.name"
# 2. apktool.yml 文件 修改 renameManifestPackage 值为新的包名
# 3. 如果有冲突的privoder，也需要把AndroidManifest文件中对应的provider 进行重命名，例如  provider android:authorities="ccc.li.provider" 
# Failure [INSTALL_FAILED_CONFLICTING_PROVIDER: Scanning Failed.: Can't install because provider name xxx.provider (in package you.new.package) is already used by original.package]
# 
```

- 重新打包为新的apk文件 `apktool b -o new.apk output_dir`
- 重新签名生成的apk文件 `apksigner sign --ks your.keystore --ks-key-alias your_alias_name --ks-pass pass:123456 new.apk`


## 大群

### 前端 

1. 进入web目录
2. 执行`npm install`
3. 执行`npm run serve`进行本地调试

可能遇到的问题——

- 编译报错`typeerror this.getoptions is not a function`

[it seems like `sass-loader@11.0.0` doesn't work with `vue@2.6.12`.](https://stackoverflow.com/a/66087132/1087122)降级之后`"sass-loader": "^10",`之后编译正常

- windows环境下报错`Error: PostCSS received undefined instead of CSS string`

中心编译`node-sass`后可以正常启动了 `npm rebuild node-sass`

### 后端

后端服务分为两个部分，一个是为前端服务的http服务；一个是进行message通信的服务。
需要配置一个postgres数据库，初始化文件为models目录下的`schema.sql`。远程登录问题参考[sonarqube](../how/sonarqube#postgresql远程登录)中的远程登录部分

---

当然入门后，遇到问题问题可以进一步搜索对应的资源，例如关于“切片”的内容——

- [短文介绍区别](https://medium.com/@marty.stepien/arrays-vs-slices-bonanza-in-golang-fa8d32cd2b7c)
- [官方tricks](https://github.com/golang/go/wiki/SliceTricks)
- [官方博客介绍切片](https://blog.golang.org/slices-intro)

- **[官方博客 index](https://blog.golang.org/index)**


读完入门系列，实操了[Mixin大群部署](../2020/0627-mixin-super-group-deploy/)，下一步开始学习[Mixin Supergroup的代码逻辑](https://github.com/MixinNetwork/supergroup.mixin.one)，正好前后端一起来。

计划每天学习内容：

- 源码阅读
- 涉及到的API调用可以独立练手
- 整理出来一段文字记录

## 历史
### 第一周

#### 第一天
fork出来一份代码，新建一个分支，直接在源码上添加注释进行学习。

简单读完入门系列，现在大群的代码至少可以看下去了。配合这张图就比较好理解，部署的时候启动的两个服务：`http`, `message`

{{< fluid_imgs
  "pure-u-1-1|https://cdn.dbarobin.com/loplv1n.png|架构示意图|https://dbarobin.com/2019/05/19/mixin-super-group/"
>}}

- 读取配置文件
- 创建数据库连接
- 根据传递的参数启动两个服务

Message服务启动：使用结构体Hub进行包装：使用Context，并对ctx进行一些列设置，最后启动接口方法 MessageService.Run()。

这个实现里具体的业务逻辑还没完全搞清楚，大概意思是利用数据库做中转：判断具体的群业务？在loop中做消息处理，使用WebSocker的传输方式。

对比起来，Go代码的确看着舒服——这样，就完成了并发下的业务处理逻辑

```go
go distribute(ctx)
go loopInactiveUsers(ctx)
go loopPendingMessages(ctx)
go handlePendingParticipants(ctx)
go handleExpiredPackets(ctx)
go handlePendingRewards(ctx)
go loopPendingSuccessMessages(ctx)
```

知识点：  

[有关包导入](https://mp.weixin.qq.com/s?__biz=MzU1NzU1MTM2NA==&mid=2247483769&idx=1&sn=5ce6c75ecccad89f455f5c43712a6d74&scene=19#wechat_redirect)

- 匿名导入 `_ "net/http/pprof"`，显示导入时不使用会报错，采用匿名导入的方式，默认会执行导入的包的`init`方法。init方法先于main方法执行，并且深度优先。“导入的导入的导入的包”的init方法最新执行

包引用关系：`main→A→B→C`，那么初始化顺序为`C.init→B.init→A.init→main`，编译时，匿名导入的包依然会编译到可执行文件中。

#### 第二天

[Mixin Messenger 机器人接入指南](https://w3c.group/c/1567337919309762)

>APP_BUTTON 的 action 支持 `input:SOMETHING` 。例如当前的 App Button 的 action 是 “input:subscribe” ，当用户点击这个按钮时客户端会自动发送一条 “subscribe” 的消息给机器人，开发者可以任意指定 input 后面的文字。

根据大群的代码以及[API说明](https://developers.mixin.one/api/beta-mixin-message/websocket-messages/)，action也可以为url地址，作为认证首页使用

```
  {
    "id": "UUID",
    "action": "CREATE_MESSAGE",
    "params": {
      "conversation_id": "UUID",
      "category": "APP_BUTTON_GROUP",
      "status": "SENT",
      "message_id": "UUID",
      "data": "Base64 encoded data"
    }
  }

# data format
[{"label": "Mixin Website", "color": "#46B8DA", "action": "https://mixin.one"}, ...]
```

记得签到[答题机器人](https://github.com/yiplee/blockquiz)有对话式式的交互。下载下来读了下源码，一直没搞清楚的关键点是：“提供给用户选项，用户选择后，可以知道用户的选择是什么？”

- 定位到了如何提供用户选项
- 后面用户选择，以及用户选择了什么一直没搞清楚

搜索之后，还是长老的教程里说得明白。参见如上。长老的另外一篇文章[Mixin 公链](https://w3c.group/c/1573118879471104)，传说中的一篇文章搞懂系列——这篇全部理解透彻，算得上Mixin专家了。

### 第二周 

执行数据库事务。这种实现可以理解业务逻辑。

#### func closure 

models目录下的`property.go`中包含以下方法——将函数作为参数传递给函数。这里对bool变量b的赋值操作有点秀？（为什么可以这样操作？）

```go
func ReadProhibitedProperty(ctx context.Context) (bool, error) {
  var b bool
  //传递匿名函数
	err := session.Database(ctx).RunInTransaction(ctx, nil, func(ctx context.Context, tx *sql.Tx) error {
		var err error
		b, err = readPropertyAsBool(ctx, tx, ProhibitedMessage)
		return err
	})
	if err != nil {
		return false, session.TransactionError(ctx, err)
	}
	return b, nil
}

// 实际执行的函数
func readPropertyAsBool(ctx context.Context, tx *sql.Tx, name string) (bool, error) {
	query := fmt.Sprintf("SELECT %s FROM properties WHERE name=$1", strings.Join(propertiesColumns, ","))
	row := tx.QueryRowContext(ctx, query, name)
	property, err := propertyFromRow(row)
	if err != nil || property == nil {
		return false, err
	}
	return property.Value == "true", nil
}
```

被调用的函数定义在durable目录下的`database.go`文件中——

```go
func (d *Database) RunInTransaction(ctx context.Context, opts *sql.TxOptions, fn func(ctx context.Context, tx *sql.Tx) error) error {
	tx, err := d.BeginTx(ctx, opts)
	if err != nil {
		return err
	}
	if err := fn(ctx, tx); err != nil {
		if err := tx.Rollback(); err != nil {
			return err
		}
		return err
	}
	return tx.Commit()
}
```

上面的代码，原理上理解还有点困难，这些资料需要参考一下：

[Can functions be passed as parameters?](https://stackoverflow.com/questions/12655464/can-functions-be-passed-as-parameters)  

[go tour: Function closures](https://tour.golang.org/moretypes/25) 完整的的[go tour](https://tour.golang.org/list) （现在居然对有些国家不提供服务？）

Go functions may be `closures`. A closure is a function value that references variables from outside its body. The function may access and assign to the referenced variables; in this sense the function is "bound" to the variables.

For example, the adder function returns a closure. Each closure is bound to its own `sum` variable.

```go
package main

import "fmt"

func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}
```

## Kernel 

代码只有15K就完成了白皮书的技术实现。可以读文档，结合代码学习一下。

```
140 text files.
140 unique files.
43 files ignored.
--------------------------------------------------------------------------------
Language                      files          blank        comment           code
--------------------------------------------------------------------------------
Go                              113           1839             89          15140
Markdown                          9            327              0           1249
JSON                              2              0              0            196
XML                               4              0              0             81
TOML                              1              4             14             16
Bourne Again Shell                1              4              0             12
Bourne Shell                      1              1              0              4
--------------------------------------------------------------------------------
SUM:                            131           2175            103          16698
--------------------------------------------------------------------------------
```

第一步编译，Mac上没有问题，Windows下提示`exec: “gcc”: executable file not found in %PATH%`，需要安装一下gcc的环境，[参考](https://stackoverflow.com/questions/43580131)——

- 从[mingw-w64](https://sourceforge.net/projects/mingw-w64/files/)这里下载编译好的MinGW-W64 GCC-8.1.0，选择`x86_64`版本的
- 解压后将目前下的bin文件夹添加到环境变量

重新编译后生成mixin.exe文件。可以直接使用——这一点go语言的确有优势，直接打包出来的产出物就可以使用

创建运行测试网络`mixin setuptestnet`会直接在exe所在目录创建对应的目录……一系列逻辑参考源码吧：生成七个地址，创建七个目录。为下一步启动主网做准备。

依次执行`mixin kernel -dir /tmp/mixin-7001 -port 7001` 启动七个节点。可以根据log输出结合代码看看运行逻辑。运行了一会，产生了28G的数据。

主网代币转账的说明应该无法在测试环境下实现。因为没有实际的Domain将资产接入到主网。

昨天的RPC调用一直不成功，今天阅读源码之后，就知道正确的姿势了——本地返回大概是这样滴，线上环境类似，只是数据更多一些。

### getInfo

`mixin -n http://127.0.0.1:8001 getinfo` 端口默认+1000；返回值最后的`node`字段代表自己
```
{
    "epoch":"2021-01-28T16:41:41+08:00",
    "graph":{
        "cache":Object{...},
        "consensus":[
            Object{...},
            Object{...},
            Object{...},
            Object{...},
            Object{...},
            Object{...},
            Object{...}
        ],
        "final":Object{...},
        "sps":0,
        "topology":7
    },
    "mint":{
        "batch":0,
        "pool":"500000.00000000"
    },
    "network":"28f297fa972963a7dde67c02dc5169bb7f5d2227d22b71ab458560c7fd70b8ed",
    "node":"4fd6038742ef3f3c4b8a3a08567e11ba5921171b2e7638b520b8e75f707a04c8",
    "queue":{
        "caches":0,
        "finals":0
    },
    "timestamp":"2021-01-28T16:41:41.000000001+08:00",
    "uptime":"57.732462s",
    "version":"v0.9.9-BUILD_VERSION"
}
```

如何知道主网地址对应的node？

跟踪了一些——CV了一些代码，获取到`IdForNetwork`的值。不知道私钥的情况下是无法将`genesis.json`中的地址跟config中的host对应起来的。这样也启动隐私保护的作用。根据`IdForNetwork`来对应host即可。获取不同host的`getinfo`即可

JSON解析：`cannot unmarshal string into go struct field` 需要让对应的struct实现`MarshalJSON(b []byte) error`和`UnmarshalJSON(b []byte) error`方法，就可以完成自定义的解析，[官方示例](https://golang.org/pkg/encoding/json/)，详细介绍[blog golang json](https://blog.golang.org/json) 



测试网络如何正常关闭？否则重启—— 

```
badger 2021/01/29 20:58:18 INFO: All 0 tables opened in 0s
badger 2021/01/29 20:58:18 INFO: Replaying file id: 0 at offset: 0
badger 2021/01/29 20:58:18 WARNING: Truncate Needed. File /tmp/mixin-7001/snapshots\000000.vlog size: 2147483646 Endoffset: 19566
During db.vlog.open: Value log truncate required to run DB. This might result in data loss
```

[badger issue 744](https://github.com/dgraph-io/badger/issues/744) `Win7 will be prompted, Mac can start normally`开起来是正常现象。启动参数添加`opt.Truncate=true`可解决，[参考](https://github.com/dgraph-io/badger/issues/1353)

### kernel components

可以梳理一些kernel都用到了哪些开源组件。查看项目的[`go.mod`](https://github.com/MixinNetwork/mixin/blob/master/go.mod)文件可以有获取全部信息。

- [Badger](https://github.com/dgraph-io/badger) `Fast key-value DB in Go.` 存储数据，所以需要使用SSD
- [edwards25519](https://github.com/FiloSottile/edwards25519) google内Go团队负责密码和安全的大牛实现的edwards25519椭圆曲线实现，创建地址的基础

### 主网转账

[转账方式](https://github.com/MixinNetwork/developers.mixin.one/blob/main/developers/src/i18n/zh/document/mainnet/tutorials/full-node-join.md) 

- 机器人的授权令牌调用 POST /transactions 转入到主网地址。[transfer-to-mainnet](https://developers.mixin.one/document/wallet/api/transfer-to-mainnet)浏览器默认英文环境，没有这个api的说明（默认中文环境时有），
- 使用官方提供的机器人bot转账，最终也是调用的相同API——调用成功`Mixin transfer success snapshotId: 7cf6d709-c048-4c74-a7da-b822e9d84e09, transaction hash: 6cabed5cff5becd72a3533a318d49ee002ae38d27c18cba9b4710f4c770a624b`。转账成功后，调用任何一个节点的`gettransaction`API都可以查询到结果

Java的api还没调通，看起来加密错误，目前提示：`PIN incorrect.`下一步参考go的实现debug一下。Java下的转账api工作OK的

### 地址格式

- [从群环域到椭圆曲线密码学](https://github.com/AlexiaChen/AlexiaChen.github.io/issues/15)  
- [深入了解Ed25519](https://github.com/AlexiaChen/AlexiaChen.github.io/issues/103)  
- [一场椭圆曲线的寻根问祖之旅](https://www.infoq.cn/article/lbo7imfxawd6um6a6qy8)
- [Ed25519 公钥从私钥哈希 SHA-512(k) 的“左半部分”计算而来](http://aandds.com/blog/eddsa.html)

每个地址/秘钥都是基于`twisted Edwards curve`生成的地址，用来进行非对称加密。

```
-x^2 + y^2 = 1 + -(121665/121666)*x^2*y^2
```

第一步：生成创建地址的随机64位字节的seed：

- 创建随机64位byte作为seed1，对其进行SHA3-256哈希取值获取到32位byte数组hash1
- 将hash1再做一次SHA3-256哈希获得hash2
- 将hash1和hash2合并得到64位的seed2

第二步：生成地址

- 使用seed1生成32位byte长度的Key作为spend Key
- 使用seed2生成32位byte长度的Key作为view Key

view key可以查看对应地址的转账信息；发起转账信息同时需要spend key和view key。

地址包含了一对私钥，公钥可以通过私钥计算得出

```
A Scalar is an integer modulo 
    l = 2^252 + 27742317777372353535851937790883648493
which is the prime order of the edwards25519 group.
This type works similarly to math/big.Int, and all arguments and receivers are allowed to alias.
The zero value is a valid zero element.
```

## TIP

Mixin使用的六位PIM码管理用户私钥的方案正式开源。2000+行代码完成。

引用一下骚总的评论—— 
>晓东是我见过最牛逼程序员～️ 他前几天感冒了，稍微好点的时候，给我们讲了一下Mixin支持原链资产钱包的思路，然后晚上大搞了一下，好像是每人搞了大约12000搞（我们发明的酒精单位，1搞=0.01ml酒精）～ 后面两天也不怎么说话，吃饭也不怎么愿意去了，总去最近的麦当劳，一天到晚坐电脑前，我一度以为他感冒没好，状态不佳～⌨️ 过了两天跟我们说，他已经写完了代码，随时可以上线～✅ 坐在对面的我感到很惭愧，只能安慰自己，每个人擅长的不一样，每个人都有独特的价值～🤦‍♂️ 今天他发布了详细的 6 位数字 PIN 码实现方案开源版本： https://github.com/MixinNetwork/tip

用到了[Boneh-Lynn-Shacham (BLS) signature](https://en.wikipedia.org/wiki/BLS_digital_signature)算法做签。有关BLS介绍参考这个[理解 BLS 签名算法](https://ethfans.org/posts/bls-signatures-better-than-schnorr)，[英文原文: BLS signatures: better than Schnorr](https://medium.com/cryptoadvance/bls-signatures-better-than-schnorr-5a7fe30ea716)以及对于对于Schnorr算法的介绍[How Schnorr signatures may improve Bitcoin](https://medium.com/cryptoadvance/how-schnorr-signatures-may-improve-bitcoin-91655bcb4744)

>BLS 签名算法是一种可以实现签名聚合和密钥聚合的算法（即可以将多个密钥聚合成一把密钥，将多个签名聚合成一个签名）。

这其实是算法发明者的三个字母缩写——三位斯坦福大学的教授+学生Dan Boneh, Ben Lynn, Hovav Shacham于2004年在《Journal of Cryptology》上发表的论文[Short Signatures from the Weil Pairing](https://link.springer.com/chapter/10.1007/3-540-45682-1_30)，下载版本可以从[google scholar](https://scholar.google.com/citations?user=262Twn0AAAAJ&hl=en)上获取

- 第一作者[Dan Boneh](https://crypto.stanford.edu/~dabo/)是为真正的大佬，教密码学课程，开源了一本结束密码学教科书[A Graduate Course in Applied Cryptography](http://toc.cryptobook.us/)，还在coursera开免费课[Cryptography I](https://www.coursera.org/learn/crypto)
- 第二作者[Ben Lynn](https://crypto.stanford.edu/~blynn/)好像是台湾人，还开源了一份git教程[Git Magic](https://crypto.stanford.edu/~blynn/gitmagic/)。大佬比较低调[github地址](https://github.com/blynn)
- 第三作者[Hovav Shacham](https://www.cs.utexas.edu/~hovav/)2018年去了University of Texas at Austin 

刹车刹车，先学习TIP的代码~ 