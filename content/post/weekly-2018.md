+++
title = "Weekly 2018"
description = "Vision Without Execution Is Hallucination"
tags = [
    "life",
    "work"
]
date = "2018-01-01"
topics = [
    "life",
    "work"
]
toc = true
+++

收集日常内容，记录灵感碎片。[2017年部分](../weekly/)

{{< fluid_imgs
  "pure-u-1-1|https://upload.wikimedia.org/wikipedia/commons/f/f6/Challenge_vs_skill.svg|FLOW|https://en.wikipedia.org/wiki/Mihaly_Csikszentmihalyi"
>}}

<!--more-->

## week01

### 2018-01-01
- 思考数据模块可能的实现方式
- 思考，快与慢购买、阅读到41页
- 跟LP一起挑一挑挑战

假期愉快，新年、新月、新周、新日。

### 2018-01-02
1. 观察上线情况
2. 准备数据模块独立

- 曾鸣书院听课vison
- 工程初始化 jetty, mysql, log4j2
- 问题定位：多次出现的fatalException，修复

### 2018-01-03
1. Vision --> 5Steps
2. 框架搭建：API接收数据 ~~并存入数据库~~


- [A Guide to Making This Your Best Year Ever](https://zenhabits.net/best-year/)
- 框架搭建只完成了Log部分的实验。

>掌握的不够熟练，另外一个项目里已经使用过，但只是使用而已。Log部分有这样的问题：第一次配置好后，基本就不会继续调整

- DM集成问题修复

>talk is cheap. 即使确认后，最后代码还是不会说谎的。talk认为观察模式可以干预；code显示观察模式无法干预。最后重新定了方案，跟踪了调用的部分，但调用后的使用还是在验证过程中才发现其中的问题。

- 更新Vision(About)和PFMMR(Contact)
- 睡前回顾明确Vision和PFMMR的重要性和可操作性

### 2018-01-04
1. 独立文章跟踪Five Modules
2. 框架进度 10/100
3. 表格式嵌入分享

- 看十年第三模块，51信用卡创始过程音频
- 协助OCR定位
- [Use Dashboard](https://zenhabits.net/dashboard/) to [Go Deeper, Not Wider](http://www.raptitude.com/2017/12/go-deeper-not-wider/) 
- 定位问题，Log监测遇到启动就Die的场景
- 定位无效任务
- DC支持下载服务

### 2018-01-05

- 无效沟通
- 整理日常使用的命令行工具
- 工作居住证复印材料提交——准备的非复印材料需要留到下周四
- 优化接口输出，JSON格式
- 定位空指针问题：参数类型错误
- Log4j 2!配置 - done

沟通事实需求，注意情绪。

### week01 weekend
洗衣，读书。似乎忘记了记录。现在(下周一)回忆居然有点模糊。


- 两天应该都读了一点书。
- 但tech内容没有学习，需要更大的动力？
- 倒是准备了两个主题：上当充值、自动部署
>>然后发现自动部署写一篇详细的说明文档还挺复杂的。重新读了相关的连接文章，最后还是没有写出来。

## week02
1. 指定年度计划的月度计划详情

Mon.(Monday) Tues.(Tuesday) Wed.(Wednesday) Thu.(Thursday) Fri.(Friday) Sat.(Saturday) Sun.(Sunday)

### 2018-01-08 Mon.
需要明确早上做的事情，尽快进入学习模式，尤其是周一的时候。

- 讨论设计重启场景
- 初步实现功能

>>在UIA端实现了完整的场景，验证时发现又遇到了之前遇到过的问题——**APK无权限执行shell命令**。浪费了设计，只能从头再来，这次该长记性了吧:(

- 晚上看了锤子科技三人组在陌陌的2017年度好物推荐

### 2018-01-09 Tue.
1. 完成功能实现集成

- from [How to Make (and Keep) a New Year's Resolution](https://www.nytimes.com/guides/smarterliving/resolution-ideas) find out [History of SMART Objectives](https://rapidbi.com/history-of-smart-objectives/) and [A BRIEF HISTORY OF SMART GOALS](https://www.projectsmart.co.uk/brief-history-of-smart-goals.php)
- [backUp HowTOMake](https://www.evernote.com/shard/s225/sh/8011fc29-fa30-4536-a0b9-fec308b56a5e/dcc5f3b77c4dfcb64cb3c202911acdcf)
- [goal Vs. objective](https://rapidbi.com/the-difference-between-goals-objectives/)
- 定位问题log：ZT&YT
- 调试问题——stop调用
- review process for RecorderTool, RBTools
- 集成YT+UIA重启逻辑-almost done

今天效率不错，都忘记去打球了:) 

### 2018-01-10 Wed.
1. 重启流程处理

- reading <<Think, Fast and Slow>>
- go flow control module.
- 重启流程完成
- 定位FML问题

### 2018-01-11 Thu.

- 继续定位FML问题，果然最后是一个极容易忽略的问题导致的
- 重启流程正式收尾提交，等待发布
- 居住证提交、汇款咨询
- update zt install tar包
- update zt imsi file

- 遇到反馈的诡异问题：卡死后设备页面全部空白——设备位置丢失

### 2018-01-12 Fri.

- 排查问题
- 修复掉线引起的上报问题
- 远程更新支持
- 新建分支、cherry pick、自动编译、增量升级、验证、推送

### week02 weekend
洗了一堆衣服、（周六下午在干什么？）Go的练习、看了一场好球、蓝色星期二、KDW晒吃相（一篇计划中的文章再次没有完成）

Think Fast and Slow周末两天有继续阅读，每次读都有新的认识——这本书应该常读常新，先读完第一遍吧

第二周，还是没有持续持续记录。跟LP卧谈是也提到，周末的时间应该更好地利用一下。

Go语言编程 from 多抓鱼

## week03
每天工作内容完成后及时记录。
Mon.(Monday) Tues.(Tuesday) Wed.(Wednesday) Thu.(Thursday) Fri.(Friday) Sat.(Saturday) Sun.(Sunday)

### 2018-01-15 Mon.
- 点线面体的分析惊出一身冷汗
- busybox支持8.0设备
- 奥利奥远程真机问题定位——minicap配套


### 2018-01-16 Tue.
- 处理批量升级
- 部署升级新工位
- go 练习
- VS code 配置使用

### 2018-01-17 Wed.

- Go method 指针这块需要加深理解
- 查询启动失败问题

>> UIA优化启动逻辑</br>
>> 问题定位sql

- 掮客工作：日志解析（更新脚本）、结果上报（前端做兼容）
- 批量更新yt、zt
- usb识别问题
- 码云gitee使用
- 问题复现——任务调整、自动点击

“触发式”工作内容

### 2018-01-18 Thu.

- 为避免cmd窗口过多，期望使用Powershell，希望powershell可以双击执行，但受限于执行策略，搜索半天没有实现 10:30
- 系统维护偏差——重新配置机器 12:25
- 设备升级诡异现象定位——更换PC，yt依然使用了旧版本
- Go study——interface
- worker project update
- 截图问题定位：端口重复导致

加入区块链战友，第一次分享后大家都很有热情。——可应用“场景”论分析。

更重要的是后续一步一步的发展，主动参与、主动输出。——对外输出能力很重要。

### 2018-01-19 Fri.
1. 定位短信问题。
2. 修复yt执行问题

- 短信问题数据查询 -done
- 定位yt执行问题 -done
- 重复截图问题修复验证


### 2018-01-20 Sat.
- 集体定位了一天的问题，最后就改了一行代码

### 2018-01-21 Sun.
- 群里讨论的很兴奋，听了近3小时的在线课程blcokchain
- 了解EOS周边：创世团队、steemit上的、youtube上的

从技术入门难度比较高，迎难而上吧

## week04
每天工作内容完成后及时记录。
Mon.(Monday) Tues.(Tuesday) Wed.(Wednesday) Thu.(Thursday) Fri.(Friday) Sat.(Saturday) Sun.(Sunday)

### 2018-01-22 Mon.

- 完成ZT升级版本
- 完成YT升级版本
- 新思路解决Copy Fail问题——顽疾解决
- [在线阅读epub](http://www.neat-reader.com/app#/epubreader)

>>微信提示升级，但直接在线升级时多次下载失败。</br>
>>Chrome下载时根据语言设置自动判断推送的英文版本，从Firefox下载中文新版</br>
>>然后自主升级又成功了。

升级内容：只描述3点内容，web版本给出了每一点的说明和图示；应用启动后会提示，只显示内容，无提示。

中午听到的产品思维课刚好讲的微信的“迭代”方式：

- 前一版是后一版的准备动作
- 第一版只是核心，发布后“自然生长”


### 2018-01-23 Tue.

- Blockchain address
- 复制提醒功能
- 讨论独立短信模块服务
- debug安装问题

### 2018-01-24 Wed.
1. minicap单张截图支持8.0设备

- [用户体验要素](https://github.com/willianjusten/hangouts/blob/master/ux/The%20Elements%20of%20User%20Experience%20-%20Jesse%20James%20Garrett.pdf)
- 问题修复。饶了一圈，最后还是一个极简单的场景
- tiger+evernote笔记练习——应该增加这方面的训练。

重新捡起来minicap相关内容，NKD编译新版本，准备测试环境、验证、再验证。最后发现是so文件的问题。

### 2018-01-25 Thu.
1. apk获取短信

- 产品思维之用户体验地图 VS. 服务蓝图
- review and design for new feature and newbie
- hot-fix bug 需求实现的偏差，定义清楚行为
- update accounts in personal project
- apk获取短信，请求server发送短信

### 2018-01-26 Fri.

- SharedPreference保存信息
- 动态获取读取短信权限 -pending
- 集成测试
- 代码提交、发布版本

### 2018-01-27 Sat.
- 思考、快与慢阅读
- 球赛、游戏183
- 洗衣
- 然后就睡觉+万万没想到

### 2018-01-28 Sun.
- 精通比特币初版阅读
- wercker文章

## week05
每天工作内容完成后及时记录。
Mon.(Monday) Tues.(Tuesday) Wed.(Wednesday) Thu.(Thursday) Fri.(Friday) Sat.(Saturday) Sun.(Sunday)

### 2018-01-29 Mon.
- 问题修复：安装失败后没有自动结束任务
- 干预接口增加参数
- 重启脚本解析
- 手机自动断线现象
- socket协议区分

### 2018-01-30 Tue.

- 升级结果检查
- keybase基础使用，channel创建
- 点、线、面培训业务场景、流程、步骤
- 日志问题定位——重启logcat导致
- George Lakoff books
- 重启脚本
- 解析日期已经掉线处理

可以分析对应log是否有异常信息

### 2018-01-31 Wen.

- 升级环境统一检查，更新安装配置手册
- 完善短信平台设计——主动性，结果不错。
- go runtine学习
- 以太坊MetaMask

老友来电&知识付费的观点。

### 2018-02-01 Tue.
- 定位执行问题
- 修复重启bug
- 短信本地培训


## week06 
每天工作内容完成后及时记录。
Mon.(Monday) Tues.(Tuesday) Wed.(Wednesday) Thu.(Thursday) Fri.(Friday) Sat.(Saturday) Sun.(Sunday)

### 2018-02-05 Mon.
- getmonero info
- DC add one api

### 2018-02-06 Tue.
1. 双工位部署更新方案
2. 邮件替代方案
3. 短信验证

- 双工位方案-done

```
tar -czvf ycyt.tar.gz ycdh.jar config/custom.ini ycdh_lib tools log4j.properties

#解压到升级文件夹
tar -xvf ycyt.tar.gz
# 执行远程更新
```
- 邮件替代方案完成上线
- 短信问题为手机特殊情况-pending
- 修复重启逻辑bug
- 配置dispatch方法


### 2018-02-07 Wen.
1. 升级zt

- 查询monitor record集成
- 升级zt——done
- 海外任务联调
- 配置不同的启动方式
- 问题定位，短信平台环境更新

### 2018-02-08 Thu.
idea：map映射上报的类型；

- 短信定位，更新配置、测试验证
- 场景调用定义