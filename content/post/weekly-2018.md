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

### 2018-01-15 Mon.
- 点线面体的分析惊出一身冷汗