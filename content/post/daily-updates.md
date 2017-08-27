+++
title = "Daily Playground"
description = "good good study, day day up. "
tags = [
    "shell",
    "work"
]
date = "2017-08-23"
topics = [
    "work"
]
+++


收集整理记录日常工作中用到的技术点。

<!--more-->



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

## Git命令使用
可配合GUI工具和命令行工具参考。

0x0. 命令行单独对不同的文件进行commit操作
```
#理解暂存区的概念，多个文件即可分别处理
git add new.file/modified.file
#然后进行commit，即完成分别进行commit的动作
git commit 

#-a参数对所有工作区的文件都有效，未添加到跟踪区的文件除外
git commit -a 
```

0x1. 配置github账号
注意事项：[source](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)

1. 生成rsa文件时可以指定不同的名称，以保证可以生成使用多个
2. 需要输入passphrase，否则系统认定不够安全，不允许添加到ssh-agent中
3. 配置config 文件，[支持多个git账号](https://my.oschina.net/csensix/blog/184434)
4. 如果不进行config的配置，在添加ssh key之后，将统一使用global的设置

```
# use git config --help to get the details
#Default Git
Host github.com
 HostName github.com
 AddKeysToAgent yes
 User gebitang
 UseKeychain yes
 IdentityFile ~/.ssh/id_rsa_github

Host office
 HostName IP address
 AddKeysToAgent yes
 User userName
 UseKeychain yes
 IdentityFile ~/.ssh/id_rsa
```

[^1]: the footnote text.
[^2]: the footnote text 2.

## Powered by
- [Hugo](//gohugo.io/)
- [Pure CSS](//purecss.io/)
- [blackburn](https://themes.gohugo.io/theme/blackburn/) by [YY](https://github.com/yoshiharuyamashita)

## Built and Deployed with Wercker

[![wercker status](https://app.wercker.com/status/e0d626037da0e4c7f1bee4dcdda350e5/m/master "wercker status")](https://app.wercker.com/project/byKey/e0d626037da0e4c7f1bee4dcdda350e5)