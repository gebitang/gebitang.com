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
- 到时准备了两个主题：上当充值、自动部署
>>然后发现自动部署写一篇详细的说明文档还挺复杂的。重新读了相关的连接文章，最后还是没有写出来。

## week02
1. 指定年度计划的月度计划详情

### 2018-01-08 一
需要明确早上做的事情，尽快进入学习模式，尤其是周一的时候。

- 讨论设计重启场景
- 初步实现功能

>>在UIA端实现了完整的场景，验证时发现又遇到了之前遇到过的问题——**APK无权限执行shell命令**。浪费了设计，只能从头再来，这次该长记性了吧:(
