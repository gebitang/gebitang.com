+++
title = "IDE issues"
description = "ide problems"
tags = [
    "ide",
    "work"
]
date = "2018-06-17"
topics = [
    "work"
]
toc = true
+++

## IDEA

### 自动清除import `ctrl+alt+o`

配置方法: settings-general-auto import-java项，勾选optimize imports on the fly，在当前项目下会自动清除无效的import。

### Desktop Entry

在Application列表中展示

`Tools -> Generate Desktop Entry`

### 全屏显示 view -> enter full screen

### 查找类方法快捷键

默认为 Ctrl+F12，可修改：
settings -> keymap -> Main Menu -> Navigate -> File Structure 

所有快捷键都可以从这里进行对于的修改

### linux环境下提示 “external changes too low”
[Inotify Watches Limit](https://confluence.jetbrains.com/display/IDEADEV/Inotify+Watches+Limit)

```
#1. Add the following line to either /etc/sysctl.conf file or a new *.conf file (e.g. idea.conf) under /etc/sysctl.d/ directory:
fs.inotify.max_user_watches = 524288
#2. Then run this command to apply the change:
sudo sysctl -p --system
```

### 找不到jar包的异常
工程A直接依赖jarA，但jarA依赖jarB，此时如果jarB没有正常引入：
#### jar包依赖
执行主线程将“跑飞”，但不会有任何提示（期待的报错如：java.lang.ClassNotFoundException:xxx 不会显示）
如：runner中对jarchivelib的依赖

#### gui依赖
主线程不会有任何提示。 如果jarA和jarB都没有导入，则会提示java.lang.ClassNotFoundException

如：Gluon Scene Builder打开 引入了`de.jensd.fx.glyphs.fontawesome.FontAwesomeIconView`的fxml文件

需要引入依赖：[Adding a custom component to SceneBuilder 2.0](https://stackoverflow.com/a/30078204/1087122)

PS: 也可以将导入的jar包直接导入GSB的lib目录，通常为 `%HOME_USER%\AppData\Roaming\Scene Builder\Library`


### 修改terminal的显示大小

[Terminal Line Buffer increase](https://intellij-support.jetbrains.com/hc/en-us/community/posts/206381169-Terminal-Line-Buffer-increase)

 Help/Find action, type 'registry' (it's IDE's internal registry (hidden settings, if you wish)), search the registry for `terminal.buffer.max.lines.count`.

### "find usages" not working

方法、变量的调用不再有效，所有方法认为“never used”。 原因：异常关闭导致的工程的index错乱

`File --> Invalidate Caches and restart` 重启后会重新建立index，比较耗时。

### idea import *

```
Setting -- Editor -- Code Style -- Java -- Imports -- Class count to use import with '*':
```

### Error:java: Compilation failed: internal java compiler error
设置里的java complier设置与当前工程的设置冲突造成的

### Log4j生成的log内容时间与系统时间不匹配
启动时指定Java参数 -Duser.timezone=GMT+08

### slice2java ice生成java文件

根据具体的.ice文件，执行命令
```
slice2java --output-dir=output/dir/path xxx.ice
```

### It is indirectly referenced from required .class files

[参考](http://blog.csdn.net/hello_yz/article/details/40113213)

- jdk版本问题
- 如果是在IDEA中，在Project Structure中对几个项目使用的jdk版本做统一处理

### cannot be resolved to a type

[参考](http://blog.csdn.net/testcs_dn/article/details/39643119)

- JDK版本不匹配
- jar包缺失或冲突 

### 设置背景图片

Ctrl+Shift+A (Help -> Find Action) --> Set background image.

### 全屏模式

Ctrl+Alt+S (in Settings) --> Search "full screen" (Toggle Full Screen mode) --> add new keymap.

for me, it's `Ctrl+Shift+G`

### idea 不显示gradle工具栏

[gradle tool window missing](https://intellij-support.jetbrains.com/hc/en-us/community/posts/205449130-gradle-tool-window-missing)， 先手动创建一个gradle类型的工程后，gradle工具栏会显示出来，之后再导入gradle工程。
