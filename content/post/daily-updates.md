+++
title = "Daily Playground"
description = "good good study, day day up. "
tags = [
    "shell",
    "work"
]
date = "2017-08-24"
topics = [
    "work"
]
toc = true
+++

收集整理记录日常工作中用到的技术点。

{{< fluid_imgs
  "pure-u-1-1|https://cdn.80000hours.org/wp-content/uploads/2016/03/80K_challengegraph_V1.png|The sweet spot|https://80000hours.org/career-guide/job-satisfaction/#dont-aim-for-low-stress"
>}}

<!--more-->

## javacv 依赖的 com.googlecode.javacpp.Loader无法找到问题
这个依赖包实际上依赖了javacpp这个jar包，但可能无法自动导入依赖。
```
<dependency>
    <groupId>opencv</groupId>
    <artifactId>javacv</artifactId>
</dependency>

<dependencies>
    <dependency>
      <groupId>com.googlecode.javacpp</groupId>
      <artifactId>javacpp</artifactId>
      <version>0.5</version>
    </dependency>
  </dependencies>

  <repositories>
    <repository>
      <id>javacpp</id>
      <name>JavaCPP</name>
      <url>http://maven2.javacpp.googlecode.com/git/</url>
    </repository>
  </repositories>
```

可以 [clean后重新 import](https://blog.csdn.net/nmcurd/article/details/80430779)，或者手动将依赖包的依赖添加到第一级依赖中

## jetty log

[jetty log](https://www.eclipse.org/jetty/documentation/current/default-logging-with-stderrlog.html)

Once activated, you can find the properties file at `${jetty.base}/resources/jetty-logging.properties`. By default, the following parameters are defined. To change them, un-comment the line and substitute your naming scheme and configuration choices.
```
## Force jetty logging implementation
#org.eclipse.jetty.util.log.class=org.eclipse.jetty.util.log.StdErrLog

## Set logging levels from: ALL, DEBUG, INFO, WARN, OFF
#org.eclipse.jetty.LEVEL=INFO
#com.example.LEVEL=INFO

## Hide stacks traces in logs?
#com.example.STACKS=false

## Show the source file of a log location?
#com.example.SOURCE=false
```
## Java Video

[javacv lib](https://github.com/bytedeco/javacv) and [example](https://github.com/bytedeco/javacv/blob/1.3.2/samples/JavaFxPlayVideoAndAudio.java)
and usage [Examle to play mp4 in JavaFx](https://github.com/teocci/Streaming-Player-Java)

## update-alternatives 配置

可以用来管理多个版本的应用，如python、java、javac。

`sudo update-alternatives --config java`

可使用 `man update-alternatives`查看使用方法。

## Python

### 修改系统默认的python的版本

[更改Ubuntu默认python版本的方法](https://www.cnblogs.com/yifugui/p/8649864.html)

修改当前用户的：编辑当前用户的bash config文件，增加一行 `alias python='/usr/bin/python3'`

使用 update-alternatives 为整个系统更改 Python 版本
```
➜  ~ su 
Password: 
lee# update-alternatives --list python
update-alternatives: error: no alternatives for python
lee# ls /usr/bin/python* 
/usr/bin/python   /usr/bin/python2.7	     /usr/bin/python2-config  /usr/bin/python3.6	 /usr/bin/python3.6m	     /usr/bin/python3-config  /usr/bin/python3m-config
/usr/bin/python2  /usr/bin/python2.7-config  /usr/bin/python3	      /usr/bin/python3.6-config  /usr/bin/python3.6m-config  /usr/bin/python3m	      /usr/bin/python-config
lee# update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1
update-alternatives: using /usr/bin/python2.7 to provide /usr/bin/python (python) in auto mode

#--install 选项使用了多个参数用于创建符号链接。最后一个参数指定了此选项的优先级
lee# update-alternatives --install /usr/bin/python python /usr/bin/python3.6 2
update-alternatives: using /usr/bin/python3.6 to provide /usr/bin/python (python) in auto mode
lee# python --version
Python 3.6.7

# cofig 
update-alternatives --config python

# remove 
update-alternatives --remove python /usr/bin/python2.7
```


### python2 & 3
```
root@PublicPro:/data# python --version
Python 2.7.6
root@PublicPro:/data# python3 --version
Python 3.4.3

#install python2 in ubuntu18.04
apt-get install -y python2.7; cp /usr/bin/python2.7 /usr/bin/python
```

### python3环境+ shapely 模块
```
#0. 安装libgeos-dev支持CDLL(libgeos_c.so)
sudo apt-get install -y libgeos-dev

#1. python3 安装pip 
apt-get install python3-pip

#2. 安装Shapely
pip3 install Shapely

#4. 验证
python3 dumputil.py 1523520544882_windowinfo
4b9b792b7975f243b0949ae77e2fba56

问题：
## window上安装Shapely依赖 geos_c.dll，需要确认本地的python版本是32位还是64位
Python 3.6.0 (v3.6.0:41df79263a11, Dec 23 2016, 07:18:10) [MSC v.1900 32 bit (Intel)] on win32
Python 2.7.9 (default, Dec 10 2014, 12:24:55) [MSC v.1500 32 bit (Intel)] on win32
安装对应的geos版本
https://trac.osgeo.org/osgeo4w/

目的是得到  geos_c.dll文件；之后复制到python对应的DLLS目录即可 

有可能保存的内容包括：
找不到文件 geoc_c.dll——没有该文件
OSError: [WinError 193] %1 不是有效的 Win32 应用程序。

```

### 解压jar文件 jar xf xxx.jar

将把xxx.jar解压到当前目录下，可使用 man jar进行查看参数。


## HTTP

### get方法传递数组

```
# 只需要使用相同的参数名称即可
http://gebitang.com?key=kvalue&reboot=1&charge_id=1&charge_id=2

# 在 HttpServletRequest中获取
Map<String, String[]> map = request.getParameterMap();
String[] ss = request.getParameterValues("charge_id");
String[] ss1 = request.getParameterValues("reboot");
String r = request.getParameter("key");
```

## VS code

### 折叠代码块快捷键

-  折叠所有区域代码的快捷  
  - 先按下  ctrl 和 K，再按下 ctrl 和 0 (0,1,2,3表示不同的折叠级别)

-  展开所有折叠区域代码的快捷  
  - 先按下  ctrl 和 K，再按下 ctrl 和 J

[ref](https://segmentfault.com/q/1010000008537247)  
`cmd + option + [ `折叠鼠标所在代码段
`cmd + option + ] `展开鼠标所在代码段

全选之后 `cmd + k` 然后 `cmd + num（0, 1， 2， 3， 4）`可以选择折叠级别

windows把cmd换成ctrl

`cmd + shift + p`输入`折叠`查看所有和折叠相关的功能以及快捷键


类似sublime，`ctrl+alt+P` 打开管理，

### 支持go语言
安装插件 `go`， 在extention中搜索go

### 支持CPP

[C/C++ for VS Code](https://code.visualstudio.com/docs/languages/cpp)，[官方文档](https://github.com/Microsoft/vscode-cpptools/blob/master/Documentation/Getting%20started.md)

You can also generate or edit a c_cpp_properties.json file with the **C/Cpp: Edit Configurations** command from the Command Palette (**Ctrl+Shift+P**).

[Visual Studio Code如何编写运行C、C++](https://www.zhihu.com/question/30315894/answer/154979413)

### 查看内存占用

1. 可使用 `"window.titleBarStyle": "native"` 切換为舊式界面。
2. 電腦卡頓的時候可以用 `Developer: Open Process Explorer` 查看各插件的內存使用。


## RBT工具使用

### rbtools提交、更新
[RBT工具使用](https://www.zybuluo.com/Jazka/note/173956#code-review)

- 首次提交review 

一般建议在独立的 task 分支进行开发, 在合并到 release 或者 master 等分支之前, 需要提交 review.

```
# 使用 -d 参数打印更多信息
rbt post -d --tracking-branch=my.local.debug.branch
```

- 追加修改

```
# 需要在对应的目录下执行
rbt  post -d -r 2919 --parent=base.changed.commit
```

### rbtools 提示错误ascii codec can t decode byte 0xe0
[参考](http://jingyan.baidu.com/article/eae07827a6f5a11fec5485f4.html)</br>
[立即停止在 Python 中使用 setdefaultencoding('utf-8')， 以及为什么](http://blog.ernest.me/post/python-setdefaultencoding-unicode-bytes)

- 这是Python 2 mimetypes的bug
- 需要将Python2.7/lib/mimetypes.py文件中如下片段注释或删除：

```
try:
    ctype = ctype.encode(default_encoding) # omit in 3.x!
except UnicodeEncodeError:
    pass
```

补充其它解决办法

解决办法： 在报错的页面添加代码： 
```
import sys 
reload(sys) 
sys.setdefaultencoding('utf8')
```

执行 Python ez_setup.py，报错：
```
UnicodeDecodeError: 'utf8' codec can't decode byte 0xb0 in position 35: invalid start byte
```
解决办法：
在报错的页面添加代码： 
```
import sys 
reload(sys) 
sys.setdefaultencoding('gb18030')
```

## Grails - Groovy - Gradle

### grails
[officail doc](http://docs.grails.org/snapshot/guide/single.html)</br>
[documentation](http://grails.org/documentation.html)</br>
[grails 2.5.6 doc](https://grails.github.io/grails2-doc/2.5.6/guide/single.html)

```
# 启动、指定端口、编码
grails -Dfile.encoding=UTF-8 -Dserver.port=8090 run-app

```

### 打包部署

```
#查看帮助 grails help
# 打包 
grails war
```

### groovy
[offical doc](http://groovy-lang.org/documentation.html#gettingstarted)</br>
[single page](http://groovy-lang.org/single-page-documentation.html)</br>
[下载离线文档](http://groovy-lang.org/download.html)

[Groovy 基础](https://www.w3cschool.cn/groovy/groovy_basic_syntax.html)</br>
[用 Groovy 服务器页面（GSP）改变视图](https://www.ibm.com/developerworks/cn/java/j-grails03118/index.html)</br>
[GSP 学习笔记(1)-- GRAILS开发](http://blog.csdn.net/netdevgirl/article/details/3720890)


### gradle
[official doc](https://docs.gradle.org/current/userguide/userguide_single.html)</br>
[中文手册1.5版本](https://dongchuan.gitbooks.io/gradle-user-guide-/java_quickstart/the_java_plugin.html)</br>
[极客学院版](http://wiki.jikexueyuan.com/project/gradle/java-quickstart.html)</br>
[official guides](https://gradle.org/guides/)</br>
下载完整版本安装后，在安装目录下的`docs/userguide/userguide.html`下有完整的手册

#### Gradle via scoop
window下的包管理工具 [`scoop`](https://github.com/lukesampson/scoop/wiki)：默认安装在用户目录下，然后再将由scoop安装的包安装到自己的apps目录下。达到不需要提供用户权限选项的目的。

例如，使用scoop安装gradle（也是gradle官方推荐的安装方法）。`scoop install gradle`

```
C:\Users\joechin>scoop install gradle
Installing 'gradle' (4.9) [64bit]
gradle-4.9-all.zip (108.2 MB) [===============================================================================] 100%
Checking hash of gradle-4.9-all.zip ... ok.
Extracting gradle-4.9-all.zip ... done.
Linking ~\scoop\apps\gradle\current => ~\scoop\apps\gradle\4.9
Creating shim for 'gradle'.
'gradle' (4.9) was installed successfully!
'gradle' suggests installing 'java/oraclejdk' or 'java/openjdk'.

```

#### Gradle projects 

```
D:\openSources\VocabHunter>gradle -q projects

------------------------------------------------------------
Root project
------------------------------------------------------------

Root project 'vocabhunter'
+--- Project ':command-line'
+--- Project ':core'
+--- Project ':gui'
+--- Project ':osx'
\--- Project ':package'

To see a list of the tasks of a project, run gradle <project-path>:tasks
For example, try running gradle :command-line:tasks
D:\openSources\VocabHunter>gradle :gui:tasks

> Task :gui:tasks

------------------------------------------------------------
All tasks runnable from project :gui
------------------------------------------------------------

Application tasks
-----------------
run - Runs this project as a JVM application

Build tasks
-----------
assemble - Assembles the outputs of this project.
build - Assembles and tests this project.
buildDependents - Assembles and tests this project and all projects that depend on it.
buildNeeded - Assembles and tests this project and all projects it depends on.
classes - Assembles main classes.
clean - Deletes the build directory.
jar - Assembles a jar archive containing the main classes.
testClasses - Assembles test classes.

Distribution tasks
------------------
assembleDist - Assembles the main distributions
distTar - Bundles the project as a distribution.
distZip - Bundles the project as a distribution.
installDist - Installs the project as a distribution as-is.

Documentation tasks
-------------------
javadoc - Generates Javadoc API documentation for the main source code.

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in project ':gui'.
components - Displays the components produced by project ':gui'. [incubating]
dependencies - Displays all dependencies declared in project ':gui'.
dependencyInsight - Displays the insight into a specific dependency in project ':gui'.
dependencyUpdates - Displays the dependency updates for the project.
dependentComponents - Displays the dependent components of components in project ':gui'. [incubating]
help - Displays a help message.
model - Displays the configuration model of project ':gui'. [incubating]
projects - Displays the sub-projects of project ':gui'.
properties - Displays the properties of project ':gui'.
tasks - Displays the tasks runnable from project ':gui'.

Verification tasks
------------------
check - Runs all checks.
jacocoTestCoverageVerification - Verifies code coverage metrics based on specified rules for the test task.
jacocoTestReport - Generates code coverage report for the test task.
test - Runs the unit tests.

Rules
-----
Pattern: clean<TaskName>: Cleans the output files of a task.
Pattern: build<ConfigurationName>: Assembles the artifacts of a configuration.
Pattern: upload<ConfigurationName>: Assembles and uploads the artifacts belonging to a configuration.

To see all tasks and more detail, run gradle tasks --all

To see more detail about a task, run gradle help --task <task>

BUILD SUCCESSFUL in 0s
1 actionable task: 1 executed
D:\openSources\VocabHunter>

```

#### gradle tasks

```
D:\openSources\VocabHunter>gradle -q help --task :gui:run
Detailed task information for :gui:run

Path
     :gui:run

Type
     JavaExec (org.gradle.api.tasks.JavaExec)

Options
     --args     Command line arguments passed to the main class. [INCUBATING]

     --debug-jvm     Enable debugging for the process. The process is started suspended and listening on port 5005. [INCUBATING]

Description
     Runs this project as a JVM application

Group
     application
D:\openSources\VocabHunter>

```

#### idea 不显示gradle工具栏

[gradle tool window missing](https://intellij-support.jetbrains.com/hc/en-us/community/posts/205449130-gradle-tool-window-missing)， 先手动创建一个gradle类型的工程后，gradle工具栏会显示出来，之后再导入gradle工程。

#### gradle 设置代理

[Gradle proxy configuration](https://stackoverflow.com/questions/5991194/gradle-proxy-configuration)

use gradlew to set or update gradl.properties file directly.

```
#use gradlew 
gradlew -Dhttp.proxyHost=127.0.0.1 -Dhttp.proxyPort=9527
gradlew -Dhttps.proxyHost=127.0.0.1 -Dhttps.proxyPort=9527

#gradl.properties
systemProp.http.proxyHost=127.0.0.1
systemProp.http.proxyPort=9527
systemProp.https.proxyHost=127.0.0.1
systemProp.https.proxyPort=9527
```


## iTerm2 ssh登录后无法显示中文

[Mac用iTerm2连接到Linux上,不能输入中文](https://blog.fazero.me/2015/09/04/Mac-iTerm2--chinese/)

服务器是ubuntu，用Mac的iterm2 ssh连上去，终端显示中文乱码，也不能输入中文，然而本地终端可以显示和输入。

**终端和服务器的字符集不匹配**。

因为我在本地和服务器都用zsh替代了bash，而且使用了oh-my-zsh，而默认的.zshrc没有设置为utf-8编码，所以本地和服务器端都要在.zshrc设置，步骤如下，bash对应.bash_profile或.bashrc文件。

在对应的配置文件末端添加
```
export LC_ALL=en_US.UTF-8  
export LANG=en_US.UTF-8
```

3.如果设置之后出现以下问题
`-bash: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8)`
解决办法：
```
sudo locale-gen en_US.UTF-8
sudo dpkg-reconfigure locales
```


```
➜  ~ locale
LANG=
LC_COLLATE="C"
LC_CTYPE="UTF-8"
LC_MESSAGES="C"
LC_MONETARY="C"
LC_NUMERIC="C"
LC_TIME="C"
LC_ALL=
➜  ~ source ~/.zshrc
➜  ~ locale
LANG="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_CTYPE="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_ALL="en_US.UTF-8"
```

## SSH登录Mac后中文乱码

在Mac OS X命令行下输入set,输出中LANG=zh_CN.UTF-8。
但是SSH登录后，找不到LANG

解决方法：登录后，输入 
```
➜  ~ set |grep LANG
➜  ~ LANG=zh_CN.UTF-8
➜  ~ export LANG
➜  ~ set |grep LANG
LANG=zh_CN.UTF-8

```
## VirtualBox 识别网银UKey

- 在启动前插入UKey
- 在设置页面选择端口操作，选择USB
- 添加过滤，自动识别的UKey


## 命令行FTP操作

完成登录，上传、下载文件
[参考](http://blog.csdn.net/indexman/article/details/46387561)

```
# 1. login 
ftp xx.xxx.xx.ip
# 2. input username and password accrodingly

# show local dir 
!dir 

# change local dir 
lcd /data/

# cd, ls, pwd, dir, works for remote.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxrwxr-x    5 600      600          4096 Feb 24  2017 test1
drwxrwxr-x    5 600      600          4096 Feb 13  2017 test2
226 Directory send OK.
ftp> cd zt/install
250 Directory successfully changed.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxrwxr-x    2 600      600          4096 Jan 21  2016 adb

# upload file to remote.
ftp> put uitest.apk

# dowload file to local
ftp> get uitest.apk
local: uitest.apk remote: uitest.apk
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for uitest.apk (393600 bytes).
226 Transfer complete.
393600 bytes received in 0.22 secs (1719.1 kB/s)


# rename file name
ftp>rename file.original.name file.renamed.name
350 Ready for RNTO.
250 Rename successful.
```

### 提示500 Illegal PORT command. ftp: bind: Address already in use

登录进入FTP后，修改传输模式。执行：`quote pasv`，然后执行 `passive`。
有可能的影响是，传输时不会再提示传输进度。需要主动结束？
```
ftp> quote pasv
227 Entering Passive Mode (xxx).
ftp> passive
Passive mode on.
ftp> ls
227 Entering Passive Mode (xxx).
150 Here comes the directory listing.
drwxrwxr-x    2 653      653          4096 Oct 19  2017 171021
drwxrwxr-x    5 600      600          4096 Feb 24  2017 xxx
drwxrwxr-x    5 600      600          4096 Feb 13  2017 yyy

```

## JVM 
[my question](https://stackoverflow.com/q/52328614/1087122)

### Heap Vs. Stack

[java栈、堆、方法区详解](https://www.cnblogs.com/hqji/p/6582365.html)  

Heap 

存储的全部是对象，每个对象都包含一个与之对应的class的信息。（class的目的是得到操作指令）；  
jvm只有一个heap区，被所有线程共享，不存放基本类型和对象引用，只存放对象本身。

堆的优劣势：堆的优势是可以动态的分配内存大小，生存期也不必事先告诉编译器，java的垃圾收集器会自动收取这些不在使用的数据，但缺点是，由于要在运行时动态分配内存，存取速度慢。

Stack

每一个线程包含一个stack区，只保存基本数据类型的对象和自定义对象的引用（不是对象），对象都存放在共享heap中；  
每个栈中的数据（基本数据类型和对象引用）都是私有的，其他栈不能访问；  
栈分为3部分：基本类型变量区、执行环境上下文、操作指令区（存放操作指令）  

栈的优势劣势：存取速度比堆要快，仅次于直接位于CPU的寄存器，但必须确定的是存在stack中的数据大小与生存期必须是确定的，缺乏灵活性。单个stack的数据可以共享。  
stack：是一个先进后出的数据结构，通常保存方法中的参数，局部变量。  
在java中，所有基本类型和引用类型都在stack中储存，栈中数据的生存空间一般在当前scopes内



```
# https://ask.csdn.net/questions/272621
public class A{
    public int i=1;
    public static void mian(String args[]){
      A a=new A();
    }
}
```

1.加载class文件到class内容区域，加载静态方法和静态变量到静态区（同时加载的）  
2.调用main方法到栈内存  
3.在栈内存中为a变量（A对象的引用）开辟空间  
4.在堆内存为A对象申请空间  
5.给成员变量进行默认初始化（此时 i=0），同时有一个方法标记，在方法区中创建一个A的方法区，将A的方法区的地址0x01给方法标记  
6.给成员变量进行显示初始化（此时 i=1）  
7.将A对象的地址值给变量a  

### 进程卡死

使用[vjtop]()查看进程使用情况，观察到的现象为老年代占满。

```
12:22:24 - PID: 5695 JVM: 1.8.0_192 USER: root UPTIME: 1d08h
 PROCESS: 119.84% cpu(9.99% of 12 core), 433 thread
 MEMORY: 11g rss, 11g peak, 0m swap | DISK: 406B read, 9kB write
 THREAD: 407 live, 64 daemon, 520 peak, 0 new | CLASS: 9444 loaded, 975 unloaded, 2 new
 HEAP: 3276m/3276m eden, 384m/409m sur, 4095m/4096m old
 NON-HEAP: 63m/66m/512m metaspace, 78m/79m/240m codeCache, 6m/7m/504m ccs
 OFF-HEAP: 7m/7m direct(max=NaN), 0m/0m map(count=0), 407m threadStack
 GC: 1/0ms/0ms ygc, 11/629ms fgc | SAFE-POINT: 11 count, 642ms time, 1ms syncTime

    TID NAME                                                      STATE    CPU SYSCPU  TOTAL TOLSYS
     11 QuantumRenderer-0                                       WAITING  7.73%  4.55%  3.37%  2.11%
     14 JavaFX Application Thread                              RUNNABLE  1.61%  0.32%  1.29%  0.25%
  25701 pool-3-thread-3875                                   TIMED_WAIT  1.49%  0.00%  0.42%  0.01%
  23934 pool-3-thread-3530                                   TIMED_WAIT  1.39%  0.00%  0.48%  0.01%
  11350 pool-3-thread-1930                                   TIMED_WAIT  1.38%  0.19%  0.73%  0.02%
   4458 pool-3-thread-811                                    TIMED_WAIT  1.37%  0.18%  1.05%  0.04%
  31111 pool-3-thread-4898                                   TIMED_WAIT  1.37%  0.18%  0.04%  0.00%
  11931 pool-3-thread-2108                                   TIMED_WAIT  1.35%  0.06%  0.70%  0.02%
  26660 pool-3-thread-4122                                   TIMED_WAIT  1.31%  0.02%  0.11%  0.01%
   2799 pool-3-thread-669                                    TIMED_WAIT  1.29%  0.09%  1.07%  0.04%
```

参考[文章](https://blog.csdn.net/zfgogo/article/details/81260172)可触发强制GC  

使用jmap工具可触发FullGC 

jmap -dump:live,format=b,file=heap.bin <pid> 将当前的存活对象dump到文件，此时会触发FullGC

jmap -histo:live <pid> 打印每个class的实例数目,内存占用,类全名信息.live子参数加上后,只统计活的对象数量. 此时会触发FullGC


### classLoader

Class类的getResource方法，GuiCamera.getClass.getResource("/name")，会从GuiCamera类的根目录上去找资源！而GuiCamera.getClass.getResource("name")，会从GuiCamera的类所在路径里去找资源！即 包名+name!

[Do You Really Get Classloaders](https://zeroturnaround.com/rebellabs/rebel-labs-tutorial-do-you-really-get-classloaders/6/)

```
# No class found
Variants

ClassNotFoundException
NoClassDefFoundError

# Helpful
IDE class lookup (Ctrl+Shift+T in Eclipse)
find *.jar -exec jar -tf ‘{}’\; | grep MyClass
URLClassLoader.getUrls() Container specific logs


#Wrong class found
Variants

IncompatibleClassChangeError
AbstractMethodError
NoSuch(Method|Field)Error
ClassCastException, IllegalAccessError

#Helpful
-verbose:class
ClassLoader.getResource() javap -private MyClass
More than one class found
Variants

LinkageError (class loading constraints violated)
ClassCastException, IllegalAccessError

#Helpful
-verbose:class
ClassLoader.getResource()
```

### 使用中的lib库不能直接覆盖更新

复制覆盖更新时，可能导致无法找到对应的class, 

    Caused by: java.lang.ClassNotFoundException: 

### java.rmi.ConnectException: Connection refused to host 

[java.rmi.ConnectException: Connection refused to host: 127.0.1.1](https://stackoverflow.com/q/15685686/1087122)
[same question](https://community.oracle.com/thread/1178074)</br>
[java.rmi.ConnectException: Connection refused to host](https://coderanch.com/t/487650/java/java-rmi-ConnectException-Connection-refused)</br>
[An Overview of RMI Applications](https://docs.oracle.com/javase/tutorial/rmi/running.html)

```
root@publicpro:~/vjtop# ./vjtop.sh 24550
java.rmi.ConnectException: Connection refused to host: 10.32.21.177; nested exception is:
        java.net.ConnectException: Connection timed out (Connection timed out)
        at sun.rmi.transport.tcp.TCPEndpoint.newSocket(TCPEndpoint.java:619)
        at sun.rmi.transport.tcp.TCPChannel.createConnection(TCPChannel.java:216)
        at sun.rmi.transport.tcp.TCPChannel.newConnection(TCPChannel.java:202)
        at sun.rmi.server.UnicastRef.invoke(UnicastRef.java:129)
        at java.rmi.server.RemoteObjectInvocationHandler.invokeRemoteMethod(RemoteObjectInvocationHandler.java:227)
        at java.rmi.server.RemoteObjectInvocationHandler.invoke(RemoteObjectInvocationHandler.java:179)
        at com.sun.proxy.$Proxy0.newClient(Unknown Source)
        at javax.management.remote.rmi.RMIConnector.getConnection(RMIConnector.java:2430)
        at javax.management.remote.rmi.RMIConnector.connect(RMIConnector.java:308)
        at javax.management.remote.JMXConnectorFactory.connect(JMXConnectorFactory.java:270)
        at javax.management.remote.JMXConnectorFactory.connect(JMXConnectorFactory.java:229)
        at com.vip.vjtools.vjtop.data.jmx.JmxClient.connect(JmxClient.java:106)
        at com.vip.vjtools.vjtop.VMInfo.processNewVM(VMInfo.java:129)
        at com.vip.vjtools.vjtop.VJTop.main(VJTop.java:52)
Caused by: java.net.ConnectException: Connection timed out (Connection timed out)
        at java.net.PlainSocketImpl.socketConnect(Native Method)
        at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
        at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
        at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
        at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
        at java.net.Socket.connect(Socket.java:589)
        at java.net.Socket.connect(Socket.java:538)
        at java.net.Socket.<init>(Socket.java:434)
        at java.net.Socket.<init>(Socket.java:211)
        at sun.rmi.transport.proxy.RMIDirectSocketFactory.createSocket(RMIDirectSocketFactory.java:40)
        at sun.rmi.transport.proxy.RMIMasterSocketFactory.createSocket(RMIMasterSocketFactory.java:148)
        at sun.rmi.transport.tcp.TCPEndpoint.newSocket(TCPEndpoint.java:613)
        ... 13 more
```

问题的逻辑搞清楚了： 

1. java应用启动时没有指定 java.rmi.server.hostname ，所以RMI server会使用默认的hostname。 
2. 启动时，识别到系统的 hostname对应的ip是/etc/hosts中定义的ip
3. vjtop去连接时，自然找到就是这个“旧的”“错误的”ip

修改了 /etc/hosts会立即生效，但java应用已经在修改前使用了“旧的”ip。所以会报上面的错误。

验证：

- 不重启机器，先修改 /etc/hosts 再启动 java应用。然后再用 vjtop连接就可以正常获取到数据。
- 即使这时再修改hostname为一个错误的地址，也依然可以正常获取到数据。

关键是java应用启动时jvm使用的hostname对应的是什么值。

### how to investigate hs_err_pid.log?

[Can anybody tell me details about hs_err_pid.log file generated when Tomcat crashes?](https://stackoverflow.com/a/8121186/1087122)</br>
[Troubleshooting Java SE. official doc](http://www.oracle.com/technetwork/java/javase/index-138283.html),and here is the [detail doc](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/index.html), refer to the chapter "5 Troubleshoot System Crashes" and "A Fatal Error Log".

[file format infor](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/felog003.html)

#### JVM Crash due to SIGSEGV
[JVM Crash due to SIGSEGV](https://stackoverflow.com/a/28404197/1087122)</br>
[Troubleshooting Guide for HotSpot VM](https://docs.oracle.com/javase/7/docs/webnotes/tsg/TSG-VM/html/crashes.html)</br>
java应用启动参数使用` -Xcheck:jni`参数
[Other Command-Line Options](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/clopts002.html)

[使用jni参数出现大量无效log](https://community.appdynamics.com/t5/Knowledge-Base/How-to-resolve-JNI-Warnings-in-application-logs/ta-p/26728)，[无效log](https://community.oracle.com/thread/3783234)，关闭使用。

```
Signal Description
SIGSEGV, SIGBUS, SIGFPE, SIGPIPE, SIGILL -- Used in the implementation for implicit null check, and so forth.
SIGQUIT Thread dump support -- To dump Java stack traces at the standard error stream. (Optional.)
SIGTERM, SIGINT, SIGHUP -- Used to support the shutdown hook mechanism (java.lang.Runtime.addShutdownHook) when the VM is terminated abnormally. (Optional.)
SIGUSR1 -- Used in the implementation of the java.lang.Thread.interrupt method. (Configurable.) Not used starting with Solaris 10 OS. Reserved on Linux. SIGUSR2 Used internally. (Configurable.) Not used starting with Solaris 10 OS. SIGABRT The HotSpot VM does not handle this signal. Instead it calls the abort function after fatal error handling. If an application uses this signal then it should terminate the process to preserve the expected semantics.
```


#### Registers

```
数据寄存器分为：
AH&AL=AX(accumulator)：累加寄存器，常用于运算;在乘除等指令中指定用来存放操作数，
另外,所有的I/O指令都使用这一寄存器与外界设备传送数据。
BH&BL=BX(base)：基址寄存器，常用于地址索引
CH&CL=CX(count)：计数寄存器，常用于计数；常用于保存计算值，
如在移位指令,循环(loop)和串处理指令中用作隐含的计数器.
DH&DL=DX(data)：数据寄存器，常用于数据传递。
他们的特点是，这4个16位的寄存器可以分为高8位: AH, BH, CH, DH.
以及低八位：AL,BL,CL,DL。
这2组8位寄存器可以分别寻址，并单独使用。


另一组是指针寄存器和变址寄存器，包括：
SP（Stack Pointer）：堆栈指针，与SS配合使用，可指向目前的堆栈位置
BP（Base Pointer）：基址指针寄存器，可用作SS的一个相对基址位置
SI（Source Index）：源变址寄存器，可用来存放相对于DS段之源变址指针
DI（Destination Index）：目的变址寄存器，可用来存放相对于ES 段之目的变址指针。
这4个16位寄存器只能按16位进行存取操作，
主要用来形成操作数的地址，用于堆栈操作和变址运算中计算操作数的有效地址。


Registers:
RAX=0x00007f76b9117b3f, RBX=0x0000000000000001, RCX=0x00106e4e63fcb4e0, RDX=0x000feed7222f6800
RSP=0x00007f778c6393c0, RBP=0x00007f77b4f2c6c8, RSI=0x0000000000000000, RDI=0x00007f7740000020
R8 =0x0000000000000000, R9 =0x00007f76b9117c20, R10=0x00007f778c639100, R11=0x00007f76d50f3112
R12=0x0000000000000001, R13=0x00007f7741cd4cf0, R14=0x00007f774c16d130, R15=0x00007f778c639428
RIP=0x00007f77b4ba610d, EFLAGS=0x0000000000010216, CSGSFS=0x0000000000000033, ERR=0x0000000000000000
  TRAPNO=0x000000000000000d
```

#### gdb 

[Debugging Under Unix: gdb Tutorial](https://www.cs.cmu.edu/~gilpin/tutorial/)</br>
[Linux software debugging with GDB](https://www.ibm.com/developerworks/library/l-gdb/index.html)</br>
[gdb Debugging Full Example (Tutorial): ncurses](http://www.brendangregg.com/blog/2016-08-09/gdb-example-ncurses.html)
```
# https://stackoverflow.com/q/8305866/1087122
# open core file
gdb ./run.sh core


Failed to read a valid object file image from memory.
Core was generated by `java -Xms8g -Xmx8g -XX:NewRatio=1 -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=5'.
Program terminated with signal SIGABRT, Aborted.
#0  0x00007faa54e99c37 in ?? ()
(gdb) where
Python Exception <class 'gdb.MemoryError'> Cannot access memory at address 0x7fa9f0116448:
#0  0x00007faa54e99c37 in ?? ()
Cannot access memory at address 0x7fa9f0116448
(gdb) bt full
#0  0x00007faa54e99c37 in ?? ()
No symbol table info available.
Cannot access memory at address 0x7fa9f0116448
(gdb)

```

### JMX 

[openJDK The JavaTMManagement Extensions (JMX) API](http://openjdk.java.net/groups/jmx/)</br>
[How to Monitor a Remote JVM running on RHEL](https://johnpfield.wordpress.com/2014/01/29/how-to-monitor-a-remote-jvm-running-on-rhel/)

[Official FAQ](https://www.oracle.com/technetwork/java/hotspotfaq-138619.html)

[关键系统的JVM参数推荐(2018仲夏版) wx](https://mp.weixin.qq.com/s/FHY0MelBfmgdRpT4zWF9dQ)</br>
[关键系统的JVM参数推荐(2018仲夏版) web](http://calvin1978.blogcn.com/articles/jvmoption-7.html)

### java 装箱和拆箱

要理解装箱和拆箱的概念，就要理解Java数据类型—— 对象类型到基本类型相互转换


* 装箱：把基本类型用它们相应的引用类型包装起来，使其具有对象的性质。int包装成Integer、float包装成Float
* 拆箱：和装箱相反，将引用类型的对象简化成值类型的数据

```
//这是自动装箱  (编译器调用的是static Integer valueOf(int i))
Integer a = 100;             
//这是自动拆箱
int     b = new Integer(100);
```

### Java线程池：ThreadPoolExecutor

- [Java线程池实现原理及其在美团业务中的实践](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html) 包含详细的运行流程图

- [更好的使用Java线程池](https://my.oschina.net/andylucc/blog/648127) 同一个作者，顺藤摸瓜，[扒一扒ReentrantLock以及AQS实现原理](https://my.oschina.net/andylucc/blog/651982)以及`并发编程`下的系列文章

- [深入理解Java线程池：ThreadPoolExecutor](http://www.ideabuffer.cn/2017/04/04/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3Java%E7%BA%BF%E7%A8%8B%E6%B1%A0%EF%BC%9AThreadPoolExecutor/) 包括详细的源码注释说明。

- 结合IDE阅读源码的注释内容。IDEA的`Documentation View`可以查看final字段的值，小惊喜——

>COUNT_BITS 就是29，CAPACITY就是1左移29位减1（29个1），这个常量表示workerCount的上限值，大约是5亿。

![](https://upload-images.jianshu.io/upload_images/3296949-2a47c9f118e696c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- [https://docs.oracle.com/javase/8/docs/](https://docs.oracle.com/javase/8/docs/)配合官方文档[Java SE
API](https://docs.oracle.com/javase/8/docs/api/index.html)


### 查看默认参数

[JDK 8: Thread Stack Size Tuning](http://xmlandmore.blogspot.com/2014/09/jdk-8-thread-stack-size-tuning.html)</br>
[Default -Xss value on Windows for JDK 8](https://stackoverflow.com/a/45142459/1087122)

```
# on linux: intx ThreadStackSize     = 1024    {pd product}
java -XX:+PrintFlagsFinal -version

# on windows:
# In JDK 8, HotSpot installation comes with a feature named Native Memory Tracking (default: disabled).
# To enable it, use:

-XX:NativeMemoryTracking=[off|detail|summary]
java -XX:+UnlockDiagnosticVMOptions -XX:NativeMemoryTracking=summary -XX:+PrintNMTStatistics -version


```

### 运行jar包中指定的 main 方法 -cp

```
# -cp 参数后面是类路径，是指定给解释器到哪里找到你的.class文件， 
# 指定类运行所依赖其他类的路径，通常是类库，jar包之类，需要全路径到jar包
# 写法： java -cp .;myClass.jar packname.mainclassname   
#  java -Xmx512m -XX:+UseSerialGC -XX:-TieredCompilation -XX:CICompilerCount=2 -XX:AutoBoxCacheMax=20000 -cp vjtop.jar:/usr/lib/jvm/java-8-openjdk-amd64/lib/tool.jar com.vip.vjtools.vjtop.VJTop 
java -cp XXXX.jar com.smbea.dubbo.bin.Console
```

### 用Jstack把java进程中的堆栈信息输出到文件
```
jstack -l PID >txt.txt
```

### JVM 参数 -Xms -Xmx 

How is the default Java heap size determined?

[oracle Default headp size](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/parallel.html#default_heap_size)

Default Heap Size
Unless the initial and maximum heap sizes are specified on the command line, they are calculated based on the amount of memory on the machine.

**Client JVM Default Initial and Maximum Heap Sizes**

The default maximum heap size is half of the physical memory up to a physical memory size of 192 megabytes (MB) and otherwise one fourth of the physical memory up to a physical memory size of 1 gigabyte (GB).

For example, if your computer has 128 MB of physical memory, then the maximum heap size is 64 MB, and greater than or equal to 1 GB of physical memory results in a maximum heap size of 256 MB.

The maximum heap size is not actually used by the JVM unless your program creates enough objects to require it. A much smaller amount, called the initial heap size, is allocated during JVM initialization. This amount is at least 8 MB and otherwise 1/64th of physical memory up to a physical memory size of 1 GB.

The maximum amount of space allocated to the young generation is one third of the total heap size.

**Server JVM Default Initial and Maximum Heap Sizes**

The default initial and maximum heap sizes work similarly on the server JVM as it does on the client JVM, except that the default values can go higher. On 32-bit JVMs, the default maximum heap size can be up to 1 GB if there is 4 GB or more of physical memory. On 64-bit JVMs, the default maximum heap size can be up to 32 GB if there is 128 GB or more of physical memory. You can always set a higher or lower initial and maximum heap by specifying those values directly; see the next section.

**Specifying Initial and Maximum Heap Sizes**

You can specify the initial and maximum heap sizes using the flags `-Xms (initial heap size)` and `-Xmx (maximum heap size)`. If you know how much heap your application needs to work well, you can set -Xms and -Xmx to the same value. If not, the JVM will start by using the initial heap size and will then grow the Java heap until it finds a balance between heap usage and performance.

Other parameters and options can affect these defaults. To verify your default values, use the -XX:+PrintFlagsFinal option and look for MaxHeapSize in the output. For example, on Linux or Solaris, you can run the following:

`java -XX:+PrintFlagsFinal <GC options> -version | grep MaxHeapSize`

```
#https://stackoverflow.com/a/13871564/1087122
root@ubuntu:# java -XX:+PrintFlagsFinal -version|grep HeapSize
    uintx ErgoHeapSizeLimit                         = 0          {product}
    uintx HeapSizePerGCThread                       = 87241520   {product}
    uintx InitialHeapSize                          := 262144000  {product}
    uintx LargePageHeapSizeThreshold                = 134217728  {product}
    uintx MaxHeapSize                              := 4171235328 {product}
openjdk version "1.8.0_151"
OpenJDK Runtime Environment (build 1.8.0_151-8u151-b12-0ubuntu0.16.04.2-b12)
OpenJDK 64-Bit Server VM (build 25.151-b12, mixed mode)

C:\Users\gebitang>java -XX:+PrintFlagsFinal -version |findstr HeapSize
    uintx ErgoHeapSizeLimit                         = 0         {product}
    uintx HeapSizePerGCThread                       = 87241520  {product}
    uintx InitialHeapSize                          := 268435456 {product}
    uintx LargePageHeapSizeThreshold                = 134217728 {product}
    uintx MaxHeapSize                              := 4280287232{product}
java version "1.8.0_151"
Java(TM) SE Runtime Environment (build 1.8.0_151-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.151-b12, mixed mode)

```

[堆内存分配](https://www.cnblogs.com/happyPawpaw/p/3868363.html)

* JVM初始分配的堆内存由-Xms指定，默认是物理内存的1/64；
* JVM最大分配的堆内存由-Xmx指定，默认是物理内存的1/4。
* 默认空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制；
* 空余堆内存大于70%时，JVM会减少堆直到-Xms的最小限制。

因此服务器一般设置-Xms、-Xmx 相等以避免在每次GC 后调整堆的大小。

### javac编译的步骤：
```
#1.先找出所有需要编译的java文件并保存到文件列表到javaFiles.txt
find src -name \*.java >javaFiles.txt
#2.使用javac进行编译，因为源代码依赖lib里面的库，所以需要指定classpath参数
javac -d bin -cp .:./lib/*  @.javaFiles.txt
# -d 指定输出目录  
# -cp 指定classpath为当前目录和lib目录下面所有的库文件   
# @ 后面指定需要编译的文件列表
```
### 验证java文件编译的版本

从jar包中解压处理任意一个类文件
```
javap -verbose PointImg.class |findstr "version"
```

### proguard混淆
```
java -jar "/home/lib/proguard.jar" @/home/project-java8-linux.pro
```

## Markdown 语法练习
This is a footnote.[^1]

[Supported Content Formats aka Markdown](https://gohugo.io/content-management/formats/) 

[markdown](https://daringfireball.net/projects/markdown/syntax) 

[Markdown 语法手册](https://www.zybuluo.com/EncyKe/note/120103)

When you do want to insert a `<br/>` break tag using Markdown, you end a line with two or more spaces, then type return.

This is a footnote.[^2]

## Git命令使用
可配合GUI工具和命令行工具参考。

### git http clone方式报错：

[error: RPC failed; curl transfer closed with outstanding read data remaining](https://stackoverflow.com/questions/38618885/error-rpc-failed-curl-transfer-closed-with-outstanding-read-data-remaining)

- `$ git clone http://github.com/large-repository --depth 1`
- `cd large-repository`
- `git fetch --unshallow`

### change remote url

```
git remote -v
# View existing remotes

git remote set-url origin https://github.com/user/repo2.git
# Change the 'origin' remote's URL

git remote -v
# Verify new remote URL
```

### use token from command line

[use token from command line](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line)

```
$ git clone https://github.com/username/repo.git
Username: your_username
Password: your_token
```

### fatal: Unable to create '/path/my_project/.git/index.lock': File exists

[sto](https://stackoverflow.com/a/7860765/1087122): `rm -f ./.git/index.lock`

### 修改commit工具 vim

提交说明信息的时候, linux默认是 nano 编辑器  
nano 这个编辑器使用 ctrl + x 来退出


```
# global
git config --global core.editor vim

# project update: .git/config add--
editor=vim
```

### 查找rep中特定用户的提交记录

[official doc](https://git-scm.com/docs/git-log), [from Stackoverflow](https://stackoverflow.com/a/2954501 )
```
git log --since=2018-01-01  --pretty=oneline --author=uername --author=gebitang |wc -l
```
### ssh variant 'simple' does not support setting port
update setting for git： `git config --global ssh.variant ssh`

### Git Bash: Could not open a connection to your authentication agent
生成公钥添加到对应的网站后，直接clone时，可能提示无法验证。
```
# need to start ssh-agent first.
eval `ssh-agent -s`
```

### Permission denied (publickey).
添加公钥之后，测试验证时会提示“Permission denied (publickey).”。这是因为还没有将生成的对应的key添加到ssh管理中（默认生成的会自动添加，后续生成的多个rsa文件需要手动添加）
```
# step 1 start agent first.  "ssh-agent bash" or " eval `ssh-agent` "

joechin@Gebitang MINGW64 ~/.ssh
$ ssh-agent bash


joechin@Gebitang MINGW64 ~/.ssh
$ ssh-add xxx_id_rsa
Identity added: xxx_id_rsa (aaaa@xxx.com)


# add the_abs_path_of_the_new_RSA_file, such as ~/.ssh/coding_id_rsa
ssh-add coding_id_rsa

# " Could not open a connection to your authentication agent" show, go to step 1

# then it could be cloned normaly.
```


### Fork and Sync Repo

```
1. git remote add upstream  https://github.com/url/project.git
2. git fetch upstream //download objects and refs from another repository 
3. git merge upstream/master //merge upstream/master to current branch

#meaning of merge
git merge origin master //将origin merge 到 master 上
git merge origin/master //将origin上的master分支 merge 到当前 branch 上 
```
- 从其他人的Repo中fork分支到自己的Repo下
- clone当前工程到本地
- [添加远程跟踪源(需要以http的方式添加)](https://help.github.com/articles/fork-a-repo/)

```
$ git remote add upstream  https://github.com/b3log/pipe.git

#查看添加结果
$ git remote -v
origin  git@github.com:gebitang/pipe.git (fetch)
origin  git@github.com:gebitang/pipe.git (push)
upstream        https://github.com/b3log/pipe.git (fetch)
upstream        https://github.com/b3log/pipe.git (push)

```

- [同步远程的跟踪源并合并到本地分支下](https://help.github.com/articles/syncing-a-fork/)

```
$ git fetch upstream

$ git merge upstream/master
```


### 指定branch clone
` git clone -b dawn-2.x git@github.com:EOSIO/eos.git`

### git checkout 

```
test@test~$git checkout -b upcoming origin/upcoming
正在检出文件: 100% (1718/1718), 完成.
分支 'upcoming' 设置为跟踪来自 'origin' 的远程分支 'upcoming'。
切换到一个新分支 'upcoming'
test@test~$
```

### 0x0. 命令行单独对不同的文件进行commit操作
```
#理解暂存区的概念，多个文件即可分别处理
git add new.file/modified.file
#然后进行commit，即完成分别进行commit的动作
git commit 

#-a参数对所有工作区的文件都有效，未添加到跟踪区的文件除外
git commit -a 
```

### 0x1. 配置github账号
注意事项：[source](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)

1. 生成rsa文件时可以指定不同的名称，以保证可以生成使用多个
2. 需要输入passphrase，否则系统认定不够安全，不允许添加到ssh-agent中
3. 配置config 文件，[支持多个git账号](https://my.oschina.net/csensix/blog/184434)
4. 如果不进行config的配置，在添加ssh key之后，将统一使用global的设置


### ssh config 示例 多个rsa文件时
[SSH CONFIG FILE](https://www.ssh.com/ssh/config/)</br>
[OpenSSH Config File Examples](https://www.cyberciti.biz/faq/create-ssh-config-file-on-linux-unix/)

配置了多个公钥后，需要针对不同的站点进行配置，否则会在git clone等交互操作时依然提示需要输入秘密。

配置了config文件后，idea中会根据这个配置文件使用对应的key[Using Git integration](https://www.jetbrains.com/help/idea/using-git-integration.html#SSH-access)

提交commit时，依然需要至少有**`user.name`**和**`user.email`**信息
```
# 对于ssh的配置
# Defines for which host or hosts the configuration section applies. 
# The section ends with a new Host section or the end of the file. 
# A single * as a pattern can be used to provide global defaults for all hosts.
Host github.com
    # Specifies the real host name to log into. Numeric IP addresses are also permitted.
    HostName github.com
    # Defines the username for the SSH connection.
    User gebitang
    # Specifies a file from which the user’s DSA, ECDSA or DSA authentication identity is read. 
    IdentityFile ~/.ssh/id_rsa
Host prj.testin.cn
    HostName prj.testin.cn
    # Specifies the port number to connect on the remote host.
    Port 8201
    # The default is ~/.ssh/identity for protocol version 1, and ~/.ssh/id_dsa, ~/.ssh/id_ecdsa and ~/.ssh/id_rsa for protocol version 2.
    IdentityFile ~/.ssh/gitlab_id_rsa
Host git.coding.net
    HostName git.coding.net
    IdentityFile ~/.ssh/coding_id_rsa
Host bitbucket
    HostName bitbucket.org
    IdentityFile ~/.ssh/bitbucket_id_rsa

```

- ssh配置

生成ssh-key
```
ssh-keygen -t rsa -b 4096 -C "email@address.com"
```
>添加ssh key（id_rsa_xxx、id_rsa_xxx.pub）到对应的repository，如github、gitlab，即可保证可以从对应的设备上就可以访问对应的repository。</br>

- git配置

>git config 配置完全局的 `--global user.name` 和 `--global  user.email`之后，可以在对应的仓库下，配置当前仓库使用的user.name 和user.email

### 0x2. commit amend修改commit
使用 git commit --amend 参数可以对最近的提交进行修改。</br>
作用相当于新提交了一个commit。
如果修改前的commit已经被同步过，需要先进行拉取 git pull的操作
否则在推送amend之后的commit时会提示:
```
$ git push origin master
To git@github.com:gebitang/gebitang.com.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'git@github.com:gebitang/gebitang.com.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
### 0x3. Permission denied (publickey)问题
[原因：SSH的配置文件ssh_config中的“IdentityFile“ 与实际情况不相符](http://blog.itpub.net/25851087/viewspace-1262468/)</br>
生成的一对秘钥，公钥上传到对应的站点后，使用-vt参数进行调试验证
```
#测试连接情况
# v for verbose mode, V for version
ssh -vT git@git.coding.net 
```

>如果clone时使用了https格式，则再次推送时总是会提示输入用户名密码，可以通过`git config --list`查看，然后修改`remote.origin.url`的值

### 0x4. 合并commits
[参考](http://www.jianshu.com/p/964de879904a)
```
# -i for interactive, 而不是指 其中，-i 的参数是不需要合并的 commit 的 hash 值 
# commitid 可理解为在此commit上进行rebase。即此commit之后的commit都可以进行rebase:)
git rebase -i commitid
```
原理：选择要rebase的基础commit，进入交互模式，根据提示对不同的commit处理为pick或squash

### 0x5. 清理、合并操作

>Your local changes to the following files would be overwritten by merge

提示上述错误信息时，可以使用下面的命令清理
```
git clean -d -fx
```
注意：会导致工程的所有配置都会丢失
```
git merge branch.name
```

### 0x6. 重新添加已经忽略的文件
```
#添加已经忽略的文件 
git add -f ignored/file/name
#忽略已经添加的文件 
git rm --cached file/want/to/be/ignore
```
[参考](http://blog.csdn.net/cyxlzzs/article/details/61422214)

### git rm 

[删除文件](https://www.liaoxuefeng.com/wiki/896043488029600/900002180232448)  
```
# 先本地删除
rm test.txt
# 在从git删除
git rm test.txt 
# 然后commit提交
git commit "delete test.txt file."
# 本地需要的话，可以更新到gitignore文件中
```

### 更新mac后默认自带的git无法使用

[StackOverFlow](https://stackoverflow.com/questions/32661484/os-x-cant-start-git-usr-bin-git-probably-the-path-to-git-executable-is-not)
```
➜  gebitang.com git status
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
➜  gebitang.com sudo xcodebuild -license
Password:
xcode-select: error: tool 'xcodebuild' requires Xcode, but active developer directory '/Library/Developer/CommandLineTools' is a command line tools instance
➜  gebitang.com sudo /usr/bin/git
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
➜  gebitang.com xcode-select --install
xcode-select: note: install requested for command line developer tools
➜  gebitang.com
➜  gebitang.com git:(master) ✗
➜  gebitang.com git:(master) ✗
```

### git log历史

```
git log --oneline # show all logs line by line
git show commit # show the commit detail 

git log commit 　　# 查询commit之前的记录，包含commit
git log commit1 commit2  # 查询commit1与commit2之间的记录，包括commit1和commit2
git log commit1..commit2 # 同上，但是不包括commit1
```

### 项目初始化

- create empty project

```
# from 0 to 1
mkdir gotourgo
cd gotourgo
git init
echo "# gotourgo" >> README.md
git add README.md
git commit -m "first commit"
git remote add origin git@git.coding.net:gebitang/gotourgo.git
git push -u origin master

```

- add existed one 

```
git remote add origin git@git.coding.net:gebitang/gotourgo.git
git push -u origin master
```

### 指定branch复制
```
git clone git@192.168.1.93:Automation/project.git -b next next
```

### 修改、忽略、撤销、删除、更新url

```
# 修改已提交的commit
git commit --amend
#忽略已跟踪的文件
git update-index --assume-unchanged filename
#撤销用：
git update-index --no-assume-unchanged filename
#删除已入仓库的文件夹
git rm -r --cached .idea/
git rm -r --cached 要忽略的文件

#git修改文件名大小写的方法。
#首先，在git命令行里面运行：
git config core.ignorecase false

#更新url
git remote set-url origin new-url

#rebase
git rebase -i base-commit-hash
```
### Basic Command line instructions

```
Git global setup
git config --global user.name "gebitang"
git config --global user.email "gebitang@hotmail.com"

Create a new repository
git clone git@gitlab.com:gebitang/simpleweb.git
cd simpleweb
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master

Existing folder
cd existing_folder
git init
git remote add origin git@gitlab.com:gebitang/simpleweb.git
git add .
git commit -m "Initial commit"
git push -u origin master

Existing Git repository
cd existing_repo
git remote rename origin old-origin
git remote add origin git@gitlab.com:gebitang/simpleweb.git
git push -u origin --all
git push -u origin --tags
```

### 显示分支branch的关系
[STO](https://stackoverflow.com/questions/5298972/relationship-between-n-git-branches)  
```
# branch A B C
git log --graph --decorate --oneline --simplify-by-decoration A B C
# all 
git log --graph --decorate --oneline --simplify-by-decoration --all
```

### git tag 

tag默认是按照字母顺序排列的。[tag操作](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001376951758572072ce1dc172b4178b910d31bc7521ee4000)  
```
# 查看所以tag  -l 注意是字母"L",以列表形式列出所有tag的版本号. -n 显示出每个版本号对应的附加说明.
git tag -l -n
# 显示时间
git log --tags --simplify-by-decoration --pretty="format:%ci %d "
# 一行显示
git log --tags --simplify-by-decoration --pretty="format:%ci %d "
```

### git更新历史commit中的author信息
[https://help.github.com/en/github/using-git/changing-author-info](https://help.github.com/en/github/using-git/changing-author-info)

[https://www.git-tower.com/learn/git/faq/change-author-name-email](https://www.git-tower.com/learn/git/faq/change-author-name-email)


[https://blog.tinned-software.net/rewrite-author-of-entire-git-repository/](https://blog.tinned-software.net/rewrite-author-of-entire-git-repository/)

Rewriting the history is done with “[git filter-branch](https://git-scm.com/docs/git-filter-branch)” by walking through the complete history. For each commit, filters are applied after which the changes are re-committed. The different filters allow modifying different parts of the commit.

The following uses “[git filter-branch](https://git-scm.com/docs/git-filter-branch)” to filter the history. Instead of manipulating the files to be recommitted like explained in [Remove files from git history](https://blog.tinned-software.net/remove-files-from-git-history/), this command uses the “–env-filter” to alter the environment in which the re-committing statement takes place.


单独clone出来当前repo
```
git clone --bare https://github.com/user/repo.git
cd repo.git
```

执行修改动作
```
#!/bin/sh
# -f to force rewrite, such as git filter-branch -f --env-filter
git filter-branch --env-filter '

OLD_EMAIL="your-old-email@example.com"
CORRECT_NAME="Your Correct Name"
CORRECT_EMAIL="your-correct-email@example.com"

if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$CORRECT_NAME"
    export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$CORRECT_NAME"
    export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```

这样修改后，会导致远端与本地所有替换的内容都认为不同，需要重新先 'git pull' 获取一次，当出现'fatal: refusing to merge unrelated histories'的提示时，需要执行'git pull origin master --allow-unrelated-histories'命令，然后修改可能的冲突即可。

更新修改后的repo
```
git push --force --tags origin 'refs/heads/*'
```

最后可以删除临时的repo

## C++ Basic


当 std::condition_variable 对象的某个 wait 函数被调用的时候，它使用 std::unique_lock(通过 std::mutex) 来锁住当前线程。当前线程会一直被阻塞，直到另外一个线程在相同的 std::condition_variable 对象上调用了 notification 函数来唤醒当前线程。
 wait_for 可以指定一个时间段，在当前线程收到通知或者指定的时间 rel_time 超时之前，该线程都会处于阻塞状态。而一旦超时或者收到了其他线程的通知，wait_for 返回

std::condition_variable::notify_one() 介绍
唤醒某个等待(wait)线程。如果当前没有等待线程，则该函数什么也不做，如果同时存在多个等待线程，则唤醒某个线程是不确定的(unspecified)。

unique_lock 对象以独占所有权的方式（ unique owership）管理 mutex 对象的上锁和解锁操作，所谓独占所有权，就是没有其他的 unique_lock 对象同时拥有某个 mutex 对象的所有权。


跟踪头文件中的变量

free与malloc()函数配对使用，释放malloc函数申请的动态内存。

mmap将一个文件或者其它对象映射进内存。文件被映射到多个页上，如果文件的大小不是所有页的大小之和，最后一个页不被使用的空间将会清零。
munmap执行相反的操作，删除特定地址区域的对象映射
```
返回说明：   
成功执行时，mmap()返回被映射区的指针，munmap()返回0。失败时，mmap()返回MAP_FAILED[其值为(void *)-1]，munmap返回-1。errno被设为以下的某个值   

EACCES：访问出错
EAGAIN：文件已被锁定，或者太多的内存已被锁定
EBADF：fd不是有效的文件描述词
EINVAL：一个或者多个参数无效
ENFILE：已达到系统对打开文件的限制
ENODEV：指定文件所在的文件系统不支持内存映射
ENOMEM：内存不足，或者进程已超出最大内存映射数量
EPERM：权能不足，操作不允许
ETXTBSY：已写的方式打开文件，同时指定MAP_DENYWRITE标志
SIGSEGV：试着向只读区写入
SIGBUS：试着访问不属于进程的内存区
```


memset是以字节为单位，初始化内存块。
```void *memset(void *s,int c,size_t n)```
总的作用：将已开辟内存空间 s 的首 n 个字节的值设为值 c。
 
把资源内存（src所指向的内存区域） 拷贝到目标内存（dest所指向的内存区域）；拷贝多少个？有一个size变量控制
```void *memcpy(void *dest, void *src, unsigned int size);```

函数指针：

搞懂C/C++函数指针 [一](http://hipercomer.blog.51cto.com/4415661/792300)、[二](http://hipercomer.blog.51cto.com/4415661/792301)、[三](http://hipercomer.blog.51cto.com/4415661/792302)

[C++ 函数指针 & 类成员函数指针](http://blog.csdn.net/crayondeng/article/details/16868351)


[生产消费模型：C++11 并发指南系列](http://www.cnblogs.com/haippy/p/3284540.html)

[Virtual： 虚函数 基类的虚函数必须在派生类中重写](http://www.cnblogs.com/xd502djj/archive/2010/09/22/1832912.html)

[Template： 实现多态](http://www.cnblogs.com/gw811/archive/2012/10/25/2738929.html)

ioctl是设备驱动程序中对设备的I/O通道进行管理的函数

## Wercker Status

[![wercker status](https://app.wercker.com/status/e0d626037da0e4c7f1bee4dcdda350e5/m/master "wercker status")](https://app.wercker.com/project/byKey/e0d626037da0e4c7f1bee4dcdda350e5)

## 80000
{{< fluid_imgs
  "pure-u-1-1|https://cdn.80000hours.org/wp-content/uploads/2016/05/80K_incomewellnessC_V6.png|High income improves evaluation of life but not emotional well-being|https://80000hours.org/career-guide/job-satisfaction/#money-makes-you-happier-but-only-a-little"
>}}





[^1]: the footnote text.
[^2]: the footnote text 2.
