+++
title = "Weekly"
description = "一周之记"
tags = [
    "life",
    "work"
]
date = "2017-09-04"
topics = [
    "life",
    "work"
]
toc = true
+++

收集日常内容，记录灵感碎片。

<!--more-->

## week1 2017-09-04

### 2017-09-04

* 定位到三个机器上无法启动uia2的问题

```shell
# Segmentation fault
# shell无响应
# 安装失败（可能是锁屏状态下，无法识别确定）Failure [INSTALL_FAILED_CANCELLED_BY_USER]
# 无法通过命令行安装apk文件， 一直处于等待中，失败掉线

#特定端口被tcp/tcp6占用？--bypass
```
* 优化uia2启动逻辑，邮件记录
* 实现参数控制socket server启动
* career guide 阅读

### 2017-09-05

* 优化uia2启动：有版本更新时，确保新版本被安装
* 定位到执行runner的bug问题，超时设置无buffer导致返回空
* 完成career guide系列阅读，整理思路

系列文章，有理论，有方法论。大量自引用

### 2017-09-06

* 继续优化uia2启动；处理(a) 非初始化时的逻辑 (b) 精确匹配进程
* 提供诊断模板——用户角度帮忙收集反馈
* 实现运行中更新控制
* 定位到重启造成的shell卡死现象——走笨办法，全线支持UIA方式。

所有的愤怒都是因为无能
所有的迷茫都是因为懒惰
由养育繁殖的压力想到的
——睡前讨论生养儿女：照顾、上学、工作

### 2017-09-07

- 实现新功能，上传netState状态。[使用Regex的分组功能](http://www.cnblogs.com/javaminer/p/3503389.html)
- 编辑wiki，完成复用jar包生成uitest.apk功能
- 开分支，完成**真·模拟重启**