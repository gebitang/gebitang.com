+++
title = "Selenium---Appium---的八卦历史"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "Appium"
]
toc = true
+++

[link to JianShu](https://www.jianshu.com/p/532b1c5e69d7)

帅大叔放前面～ 
![Jason Huggins](https://upload-images.jianshu.io/upload_images/3296949-1d5136fd81440afe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 前传
1994年Jason Huggins在Notre Dame大学读工商信息管理专业，98年毕业后在PeopleSoft公司做了两年的售前技术支持，2000年开始加入了ThoughtWork公司，干了6年多。 

2004年在ThoughtWork工作时，为测试Plone(基于python的内容管理系统)写了“JavaScriptTestRunner”，这就是后来的Selenium-core的前身。Jason也就成为了Selenium的共同创始人。

此时，Dan Cuellar还在卡耐基梅隆大学学习计算机科学。07年毕业后，Dan在微软做了4年的测试开发工程师之后，与11年加入Zoosk公司做测试Lead。随着iOS产品的测试用例越来越长，他开始调研如果自动化。彼时的苹果还支持UIAutomation。

一番折腾之后，他完成了iOSAuto工具的开发，使用C#语言按照Selenium的语法方式实现了解析JS命令的自动化框架。

### 相逢
2012年Dan被在伦敦召开的Selenium大会选为演讲嘉宾，但演讲主题与他的iOSAuto无关，在演讲中Dan演示了他的iOSAuto框架，出人意料得大受欢迎，有人建议他再做一次Lighting Talk（闪电演讲）进行详细说明。

第二天但闪电演讲正好由Huggins Jason主持，但Dan的演讲却出了故障，幻灯片一直无法加载，尽管最后时刻搞定了，但匆匆忙忙的5分钟演讲效果并不理想。 

话说Huggins在ThoughtWork工作了6年之后，与07年加入了Google，成为当时还是保密状态的Selenium支持小组成员，恰逢Google Doc产品需要大量的Web测试。

基于这些经验，08年Huggins离开google，并作为联合创始人创办了Sauce Labs公司同时担任CTO。12年时，Sauce Labs也支持iOS端的测试。当年的Selenium大会之后又过了几个月，Huggins想到了Dan做的有个iOS测试的闪电演讲，当时iOSAuto还不开源，两人在旧金山的一个酒吧里会面。

### 开始
在Huggins的鼓励下，Dan将iOAAuto开源，当年8月3日，在github上提交了C#版本的[第一版](https://github.com/penguinho/appium-old/commit/3ab56d3a5601897b2790b5256351f9b5af3f9e90)，Huggins建议换个更受欢迎的语言以吸引更多的用户，9月1日，Dan又提交了一个[python版本](https://github.com/penguinho/appium-old/commit/9b891207be0957bf209a77242750da17d3eb8eda)

为推广这个项目，Huggins建议在11月份举行的移动测试峰会（Mobile Testing Summit）上做演示。但需要换个名字。一番套路之后，他们决定使用“AppleCart”这个名字。但一天之后就不得不放弃了。

Huggins仔细阅读了苹果公司的版权和商标相关指南之后，发现苹果公司“你要使用这个商标，我就告你”的列表第一名就是：“Applecart”。他将这个消息告诉Dan之后，两人又头脑风暴了一番。然后Huggins想到了这个绝妙的名字：Appium：Selenium for Apps

### 后续
The rest is history.  Huggins的Sauce Labs公司全面拥抱appium，并成立工程小组推动使用，其中就包括Jonathan Lipps（现在的appium的项目Lead），小组决定完全重新Appium，使用NodeJS重写后端框架。

PS：一直对机械电子感兴趣的Huggins在Sauce Labs公司时就开始折腾自动化机器人玩愤怒哦的小鸟，类似这样——
![](https://upload-images.jianshu.io/upload_images/3296949-cf7a103242f25dd2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

离开Sauce Labs一年之后，2015年他又创建了[Tapser Robotics](https://www.tapster.io)公司，大概卖这样的东西——
![](https://upload-images.jianshu.io/upload_images/3296949-3fd85cca872ae119.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---

Dan大学辅修的专业是音乐工业，现在是苹果Music的工程经理——
![Dan Cuellar](https://upload-images.jianshu.io/upload_images/3296949-659571c1d64ef154.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
