+++
title = "App开源测试框架去哪儿了"
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



[link on JianShu](https://www.jianshu.com/p/b0c2af93a4a7)

中文搜索 App开源测试框架，大概率可以看到下面这个文章的变种——
- [移动APP自动化测试框架对比](https://mp.weixin.qq.com/s/pu1TKZhNW2L2BmvpjGzLCw) 2016年文章，现在的内容依然很多来自这篇的变种；

文章保存在微信上，依然可以找的，而TMQ(Tencent Mobile Quality Center http://tmq.qq.com)、GT(http://gt.qq.com)本身似乎本身随着业务调整已经找不到入口了。现在统一为 WeTest平台

站在2019年的年底，看看这些App开源框架都哪儿去了：
>Android自动化框架
>1. Instrumentation 
>2. Robotium 
>3. UIAutomator
>4. Espresso
>5. Calabash
>6. Appium
>7. Selendroid
>8. Robolectric
>9. RoboSpock
>10. Cafe
>11. Athrun

>IOS自动化框架
>1. XCTest
>2. UIAutomation
>3. Frank
>4. KIF
>5. Calabash-ios
>6. Subliminal
>7. Kiwi

---

google官方的instrumentation的框架不支持跨应用，其中的佼佼者[Robotium](https://github.com/RobotiumTech)目前已经无人维护，域名 [www.robotium.org](http://www.robotium.org/)直接调整到了github的项目页面，留给大家缅怀。

google的Espresso也是基于instrumentation的，自然也不能跨应用，国内似乎也少见人使用，google提供了一套各个测试框架的测试样例[https://github.com/android/testing-samples](https://github.com/android/testing-samples)

同样不支持跨应用的[https://github.com/calabash](https://github.com/calabash)也停止了维护——
>After delivering support for the final releases of iOS 11 and Android 8 operating systems, Microsoft will discontinue our contributions to developing Calabash, the open-source mobile app testing tool. We hope that the community will continue to fully adopt and maintain it.

由ebay支持的[https://github.com/selendroid/selendroid](https://github.com/selendroid/selendroid)最新的更新还在2018年11月4日。

[Robolectric](https://github.com/robolectric/robolectric) 4.7k 针对Android代码的单元测试，活跃维护

[RoboSpock](https://github.com/robospock/RoboSpock) 停止维护

[Cafe](http://cafe.baidu.com/#panel1)当初Cafe是百度出品的一个基于Robotium的测试框架，现在已经变成了项目管理工具。

淘宝的Arthrun基于instrument实现，支持Android、iOS，目前这个链接也已经失效[http://code.taobao.org](http://code.taobao.org/)


iOS端8.0版本之前也提供基于Instruments的测试方法，后续版本移除了Instruments，只支持XCTest方式。 iOS端的uiautomation也随风而去。

对应的[Frank](http://www.testingwithfrank.com/)只剩一个blog站点，[Subliminal](https://github.com/inkling/Subliminal/)停止维护

[KIT](https://github.com/kif-framework/KIF)依然活跃，使用私有API。
 Keep It Functional, is an iOS integration test framework. It allows for easy automation of iOS apps by leveraging the accessibility attributes that the OS makes available for those with visual disabilities. 这跟Android端的Uiautomator是一样的思路。

## 剩者为王，留下的只有Appium和Uiautomator

---

不要迷信英文世界的文章，套路也有很多——

[TOP 15 Best Mobile Testing Tools In 2019 For Android & IOS](https://www.softwaretestinghelp.com/best-mobile-testing-tools/)看起来是最近修改于2019年9月8日，但里面提到的工具内容对比一下上面的说明就知道这是个很outdated的内容。另外一家 [14 Best Mobile App Testing Tools for Android & iOS (2019)](https://www.guru99.com/mobile-testing-tools.html)提供的文章在google上也有很高的权重。——这些是SEO做得好，内容不敢恭维。

square的[spoon](https://github.com/square/spoon)17年似乎也不继续维护。

bbc的[hive-ci](https://github.com/bbc/hive-ci)一开始就是个玩具？

testmunk好像16年就已倒闭
