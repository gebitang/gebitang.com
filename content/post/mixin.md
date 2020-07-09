+++
title = "Go Mixin Go"
description = "supergroup.mixin.one"
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

## 第一周

### 第一天
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

### 第二天

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

## 第二周 

执行数据库事务。这种实现可以理解业务逻辑。

### func closure 

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

