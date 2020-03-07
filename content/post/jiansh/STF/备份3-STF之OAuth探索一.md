+++
title = "备份3-STF之OAuth探索一"
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



[link on JianShu](https://www.jianshu.com/p/1d0202cf5551)

~~[STF之OAuth探索一](https://www.jianshu.com/p/b39b8dbf1375)~~删除备份
`2019.07.09 21:56:49 字数 306 阅读 118`


新工作第一项要将STF的ldap认证模式修改为SSO方式。

目前的背景是只在交接时演示了一下本地启动，本身没有Nodejs开发经验。

探索思路：

1\. 熟悉Nodejs基本知识，将[guide](https://links.jianshu.com/go?to=https%3A%2F%2Fnodejs.org%2Fen%2Fdocs%2Fguides%2F)下的Node.js core concepts模块过了一遍

2\. 菜鸟学院的[Node教程](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.runoob.com%2Fnodejs%2Fnodejs-tutorial.html)扫了一遍

完成上面的两部，跟实际的OAuth还没有正面相遇呢。 内网上的SSO相关wiki浏览了一遍——没有统一的组织，好几个地方都有类似内容——初步判断没有完美支持。

根据STF[官方说明](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fopenstf%2Fstf%2Fblob%2Fmaster%2Fdoc%2FDEPLOYMENT.md%23stf-authservice)，又参考了对应的[issue](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fopenstf%2Fstf%2Fissues%2F903)，再申请使用的appKey和appSecret，甚至重新clone了一份最新的STF源码之后——

可以运行起来，并自动跳转到了认证页面，但认证之后但回调url没有到预期但地址:( 

下一步：

1\. 梳理清楚OAuth但认证逻辑，参考[ 《OAuth 2.0 教程》](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2019%2F04%2Foauth_design.html)用github替换内网但SSO方式；

2\. 到这一步再撸代码应该看个七七八八了吧

（未完待续……）
