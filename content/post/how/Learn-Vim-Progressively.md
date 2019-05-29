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

### 打开以"-"开头的文件
```
# 指定相对目录即可，如当前文件夹下的 ./-abc.log
vi ./-abc.log
```

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
### day2 2017-10-17

```
:1, 10d #删除当前光标之后的10行
:E # 打开当前目录，光标选择之后，o for open打开当前文件。
:9, 15 copy 18 # 复制第9行至15行的内容到 18行
```
### day3 2017-10-18

使用VIM编辑content下的内容
练习保存，复制功能

### 查找替换

```
# 用字符串 str2 替换正文中所有出现的字符串 str1
g/str1/s//str2/g

:s/vivian/sky/         #替换当前行第一个 vivian 为 sky
:s/vivian/sky/g     #替换当前行所有 vivian 为 sky
:n,$s/vivian/sky/     #替换第 n 行开始到最后一行中每一行的第一个 vivian 为 sky
:n,$s/vivian/sky/g     #替换第 n 行开始到最后一行中每一行所有 vivian 为 sky
 （n 为数字，若 n 为 .，表示从当前行开始到最后一行）

# 替换每一行的第一个 vivian 为 sky
:%s/vivian/sky/        #（等同于:1,$s/vivian/sky/   :g/vivian/s//sky/） 


```

### 行内操作

w：光标以单词向前移动 nw：光标向前移动n个单词 光标到单词的第一个字母上  
b：与w相反  
e: 光标以单词向前移动 ne：光标向前移动n个单词 光标到单词的最后一个字母上  
ge:与e相反  

## 练习材料
[Learn Vim Progressively](http://yannesposito.com/Scratch/en/blog/Learn-Vim-Progressively/)

[LVP中文](https://coolshell.cn/articles/5426.html)

[vimdoc](http://vimdoc.sourceforge.net/htmldoc/usr_toc.html)

[打造属于自己的Vim神器](https://zilongshanren.com/blog/2014-06-19-make-your-vim-weapon.html)

```
# from shell, vimtutor
vimtutor

# manual 
:help user-manual

# 打开手册后会自动分割出新窗口，ctrl+c+w在不同窗口见切换。
# g+] 在手册的不同章节间切换——需要先选中对应章节
# :q 可以关闭光标所在的窗口。
```

### 跳转到最后一行 G


### E484: Can't open file /usr/share/vim/vim74/syntax/syntax.vim 
缺少对应的文件导致的，可以先进行删除处理

### 删除提示 subprocess installed post-installation script returned error exit status 

执行自动清理和升级动作
```
apt-get autoclean
apt-get autoremove
apt-get update
```
重新安装对应的应用
