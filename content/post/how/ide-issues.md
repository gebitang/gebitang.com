+++
title = "IDEA issues"
description = "ideA problems"
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

### 全局设置Maven

- Mac： `File`--`New Projects Setup` -- `Preferences for New Projects`
- Win:  `File`--`New Projects Setup` -- `Settings for New Projects`

### ignored pom.xml

[Maven创建module出现Ignored pom.xml文件](https://blog.csdn.net/weixin_43901865/article/details/112596443)

module创建后被删除，再次创建**同名**module时提示对应的pom.xml被忽略。在 `settings-->Building-->Build Tools-->Maven`查看菜单下的ignored files的列表。从列表中移除即可

### 异常断电导致无法启动

重启PC后报错`Internal error. Please refer to https://jb.gg/ide/critical-startup-errors`，`Caused by: java.net.BindException: Address already in use: bind`，`java.util.concurrent.CompletionException: java.net.BindException: Address already in use: bind`，详细报错——

```
Internal error. Please refer to https://jb.gg/ide/critical-startup-errors

java.util.concurrent.CompletionException: java.net.BindException: Address already in use: bind
    at java.base/java.util.concurrent.CompletableFuture.encodeThrowable(CompletableFuture.java:314)
    at java.base/java.util.concurrent.CompletableFuture.completeThrowable(CompletableFuture.java:319)
    at java.base/java.util.concurrent.CompletableFuture$AsyncSupply.run(CompletableFuture.java:1702)
    at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
    at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
    at java.base/java.util.concurrent.Executors$PrivilegedThreadFactory$1$1.run(Executors.java:668)
    at java.base/java.util.concurrent.Executors$PrivilegedThreadFactory$1$1.run(Executors.java:665)
    at java.base/java.security.AccessController.doPrivileged(Native Method)
    at java.base/java.util.concurrent.Executors$PrivilegedThreadFactory$1.run(Executors.java:665)
    at java.base/java.lang.Thread.run(Thread.java:829)
Caused by: java.net.BindException: Address already in use: bind
    at java.base/sun.nio.ch.Net.bind0(Native Method)
    at java.base/sun.nio.ch.Net.bind(Net.java:455)
    at java.base/sun.nio.ch.Net.bind(Net.java:447)
    at java.base/sun.nio.ch.ServerSocketChannelImpl.bind(ServerSocketChannelImpl.java:227)
    at io.netty.channel.socket.nio.NioServerSocketChannel.doBind(NioServerSocketChannel.java:134)
    at io.netty.channel.AbstractChannel$AbstractUnsafe.bind(AbstractChannel.java:550)
    at io.netty.channel.DefaultChannelPipeline$HeadContext.bind(DefaultChannelPipeline.java:1334)
    at io.netty.channel.AbstractChannelHandlerContext.invokeBind(AbstractChannelHandlerContext.java:506)
    at io.netty.channel.AbstractChannelHandlerContext.bind(AbstractChannelHandlerContext.java:491)
    at io.netty.channel.DefaultChannelPipeline.bind(DefaultChannelPipeline.java:973)
    at io.netty.channel.AbstractChannel.bind(AbstractChannel.java:248)
    at io.netty.bootstrap.AbstractBootstrap$2.run(AbstractBootstrap.java:356)
    at io.netty.util.concurrent.AbstractEventExecutor.safeExecute(AbstractEventExecutor.java:164)
    at io.netty.util.concurrent.SingleThreadEventExecutor.runAllTasks(SingleThreadEventExecutor.java:472)
    at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:500)
    at io.netty.util.concurrent.SingleThreadEventExecutor$4.run(SingleThreadEventExecutor.java:989)
    at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74)
    at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
    ... 1 more

-----
Your JRE: 11.0.11+9-b1341.57 amd64 (JetBrains s.r.o.)
C:\Program Files\JetBrains\IntelliJ IDEA 2019.2.3\jbr
```

[Start Failed, Internal error](https://intellij-support.jetbrains.com/hc/en-us/articles/360007568559)，官方答复临时[解决方案](https://youtrack.jetbrains.com/issue/IDEA-238995)

原因是因为本地端口都被占用了，这种情况下，异常断电导致系统没有释放上次依然在使用中的端口。暴力处理，管理员权限cmd命令执行`net stop winnat`关闭nat，重启nat`net start winnat`

因为ide启动时会试图使用这些端口：`If all the 50 ports between 6942 and 6991 are reserved, taken by the other apps or firewall doesn't allow IDE to bind on them, startup fails with the following exception:`，如有已经被占用，重启这些端口服务。

*另外，终极重启大法也好使*

### 折叠代码

[Code folding](https://www.jetbrains.com/help/idea/working-with-source-code.html#code_folding)  

- 折叠鼠标所在代码块：`Ctrl+NumPad +`或者 `Ctrl + plus`；反向操作`Ctrol + NumPad -`或`Ctrl + minus`
- 折叠当前类/文件所有代码块：`Ctrl + Shife + NumPad +` 依次类推


### 相对路径问题

默认的工作路径不同导致相同代码在Eclipse上通过，但IDEA下执行失败。 找不到路径

查看当前路径—— 
```
File file = new File(".");
for(String fileNames : file.list())
    System.out.println(fileNames);
```

对应多模块项目来说—— 

- Eclipse默认是当前模块所在的路径
- idea默认的是当前项目所在的路径。可以在`run -> edit configurations -> working directory`配置下修改工作目录

### 使用本地文件作为依赖的源码

`Project Settings` --> `Libraries` --> 选中对应的jar包，右侧添加`Source`选中本地文件夹(项目的src文件夹)

### 个人正版支持多个设备使用

[Can I use my personal license on multiple machines?](https://sales.jetbrains.com/hc/en-gb/articles/206544319-Can-I-use-my-personal-license-on-multiple-machines-)

趁着[1024节日活动](https://www.jetbrains.com/zh-cn/lp/programmers-day/)，购买了第一份license for IDEA Ultimate

### 将IDE以及工程目录添加到信任区

[Antivirus Impact on Build Speed](https://intellij-support.jetbrains.com/hc/en-us/articles/360006298560)，这篇文章里可以链接到：社区版本的源码库、

将对应的文件夹、exe文件添加到信任区。

[Changing IDE default directories used for config, plugins, and caches storage](https://intellij-support.jetbrains.com/hc/en-us/articles/207240985)修改默认的目录地址。

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

[goland: Override console cycle buffer size](https://www.jetbrains.com/help/go/settings-console-folding.html)

### "find usages" not working

import异常"cannot resolve symbol"；方法、变量的调用不再有效；所有方法认为“never used”。 原因：异常关闭导致的工程的index错乱

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

### lombok 注解不生效

前提需要安装lombok插件：`ctrl+alt+s`（或者`file->settings`）搜索lombok，安装后重启生效

### 其他IDEA时提示端口占用问题 

Start Failed, Internal error: recovering IDE to the working state after the critical [startup error](https://intellij-support.jetbrains.com/hc/en-us/articles/360007568559) 

这里的解决方案并未生效，差点就重装了。重启电脑后可以正常启动，怀疑是上次有server在运行中，强制关机导致的。端口一直没有释放，需要使用 `netstat -ano | findstr 28000` + `taskkill /F /PID 3223(端口号)`配合检查一下是否有端口真正被占用。

```
Address already in use
...
---
JRE 11.0.4+10-b304.69 amd64 by JetBrains s.r.o C:\Program Files\JetBrains\IntelliJ IDEA 2019.2.3\jbr
```

### cannot resolve symbol 

代码中的类明明在，却找不到。看起来是个[普遍问题](https://stackoverflow.com/questions/5905896/intellij-inspection-gives-cannot-resolve-symbol-but-still-compiles-code) 

尝试操作包括：  
- File -- Settings -- Invalid Cache and Restart 
- Re-Import all Maven projects
- manage projects：删除当前有问题的项目，重新导入，导入时选择删除已有的项目重新导入 （work for me this way）

###  打开uiautomator类型的源码项目

针对project settings：

- Project
- Modules
- Libraries
- Facets
- Artifacts

关注前三个——

**Project**： 

配置对应的SDK、语言级别、编译输出的目录

**Modules**：

选择Sources对应的目录（src），否则打开源码时无法识别调用关系

**Libraries**：

依赖的lib通常直接在工程目录下，需要手动新建lib（添加，选择Java类型，选中对应的目录，如libs。）

--- 

关键是对于Modules和Libraries的设置，否则IDE无法识别工程。