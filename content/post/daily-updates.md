+++
title = "Daily Workground"
description = "good good study, day day up. "
tags = [
    "shell",
    "work"
]
date = "2017-08-24 10:41:50"
topics = [
    "work"
]
+++

## Markdown 语法练习
This is a footnote.[^1]

[Supported Content Formats aka Markdown](https://gohugo.io/content-management/formats/) 

[markdown](https://daringfireball.net/projects/markdown/syntax) 

[Markdown 语法手册](https://www.zybuluo.com/EncyKe/note/120103)


This is a footnote.[^2]

## linux 环境

16.04root密码

1. 默认root密码是随机的，即每次开机都有一个新的root密码
2. 在终端输命令 sudo passwd，然后输入当前用户的密码
3. 输入新的密码并确认，此时的密码就是root新密码
4. 修改成功后，输入命令 su root，再输入新的密码

## Shell commands

任何场景下，使用
```
man command
``` 
查看命令详情 

0x0. 启动terminal 快捷键  
```
Ctrl+Alt+T
```


0x1. 等待
	
    sleep 1 #睡眠1秒 默认
	sleep 1s #睡眠1秒
	sleep 1m #睡眠1分
	sleep 1h #睡眠1小时
	

0x2. 日期
```
#%s表示当前时间的秒数，而%N表示当前时间的纳秒部分，即1秒以下的那部分
joechin@ubuntu:~$ date +%Y%m%d%H%M%S%N
20170824115956519750498
joechin@ubuntu:~$ date +%Y%m%d%H%M%S
20170824115957

```

[^1]: the footnote text.
[^2]: the footnote text 2.
