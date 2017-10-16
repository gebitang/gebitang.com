+++
title = "Learn Vim Progressively"
description = "practice makes perfect "
tags = [
    "vim",
    "work"
]
date = "2017-10-16"
topics = [
    "vim"
]
toc = true
+++

鼓足了多次勇气，一直没有拿下这个神器。今天(2017-10-16)立个Flag：每天晚饭后至少练习30分钟，两个月后熟练使用VIM。

**Level Up**

>- 一、1st Level – Survive
- 二、2nd Level – Feel comfortable
- 三、3rd Level – Better. Stronger. Faster.
- 四、4th Level – Vim Superpowers

现有水平：一
<!--more-->

## 练习记录

### day1 2017-10-16

稍加练习其实就可以达到水平二。
目前开始使用VIM进行编写这篇文章，这也是练习的过程。


```
set nu #显示行号
set nonu #隐藏行号
# input operation
a -->在光标后插入，使用最普遍的
o -->字母o，在当前行后插入一个新行
O -->大写的字母O，在当前行前插入一个新行
cw --> 替换从光标所在位置后到一个单词结尾的字符，修改文件时用。 
# 移动光标 
0 --> 数字0，到行头
^ --> 到行头
$ --> 到本行行尾
/patter --> 搜索模式，按n到下一个

# copy past
p --> for paste， p 在当前位置之后，常用的；P 大写则表示在当前位置之前
yy --> 拷贝当前行
dd --> 删除当前行

# Undo Redo 
u --> 撤销
Ctrl+r --> 前进

# 文件操作
:e path/to/file 
:w save, but not quit
:wq save and quit
:saveas path/to/file, save to file
:x = ZZ == :wq 
:q! 
:open path/to/file
:bn next open file
:bp previous open file 
```

## 练习材料
[Learn Vim Progressively](http://yannesposito.com/Scratch/en/blog/Learn-Vim-Progressively/)

[LVP中文](https://coolshell.cn/articles/5426.html)

