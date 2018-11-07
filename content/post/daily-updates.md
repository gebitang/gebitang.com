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

## Python

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


## Mac 相关
### brew 安装应用的地址

```
# brew install hugo
# https://discourse.gohugo.io/t/hugo-homebrew-update-for-osx/82
brew update
brew install hugo

# 安装的路径：/usr/local/Cellar/hugo/

```

### 强制退出程序

使用快捷键：Command+Option+Esc 打开强制退出程序窗口，然后选中你需要退出的程序，再点右下方的“强制退出”即可。

### 解压jar文件 jar xf xxx.jar

将把xxx.jar解压到当前目录下，可使用 man jar进行查看参数。

## Sublime Text 

[Official Doc](http://www.sublimetext.com/docs/3/)</br>
[Eclipse、Sublime快捷键](https://www.zybuluo.com/Gebitang/note/82413)

### Install for Ubuntu
[install doc](http://www.sublimetext.com/docs/3/linux_repositories.html)
Install the GPG key:
```
wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
```
Ensure apt is set up to work with https sources:
```
sudo apt-get install apt-transport-https
```
Select the channel to use:

Stable
```
echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
```
Dev
```
echo "deb https://download.sublimetext.com/ apt/dev/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
```
Update apt sources and install Sublime Text
```
sudo apt-get update
sudo apt-get install sublime-text
```


## curl命令调用post方法

```
curl -i -H "Accept:application/json" -H "Content-Type:application/json" -XPOST "http://sss.com/abc/" -d '{"op":"Device.restartDevice","device":{"imei":"866568023794679"}}'
```

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

类似sublime，`ctrl+alt+P` 打开管理，

### 支持go语言
安装插件 `go`， 在extention中搜索go

### 支持CPP

[C/C++ for VS Code](https://code.visualstudio.com/docs/languages/cpp)，[官方文档](https://github.com/Microsoft/vscode-cpptools/blob/master/Documentation/Getting%20started.md)

You can also generate or edit a c_cpp_properties.json file with the **C/Cpp: Edit Configurations** command from the Command Palette (**Ctrl+Shift+P**).

[Visual Studio Code如何编写运行C、C++](https://www.zhihu.com/question/30315894/answer/154979413)



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

## Mysql

### 查看表结构 
```
desc `tableName`

desc 表名;
show columns from 表名;
describe 表名;
show create table 表名;
```
### 查询数据库版本号 select @@Version

### CLIENT_PLUGIN_AUTH is required 报错

版本过高，降低mysql-connector-java的版本。

```
# 6.0.6 com.mysql.cj.jdbc.Driver
String driver = "com.mysql.jdbc.Driver"; //
String url = "jdbc:mysql://" + DB_ADDRESS + ":" + DB_PORT + "/" + DB_NAME
        + "?connectTimeout=10000&characterEncoding=UTF-8";
String user = DB_USER;
String password = DB_PASS;

try {
    Class.forName(driver);
    DriverManager.setLoginTimeout(10);
    SmsService.conn = DriverManager.getConnection(url, user, password);
} catch (ClassNotFoundException e) {
    e.printStackTrace();
} catch (SQLException e) {
    e.printStackTrace();
}
```

### 运行远程访问
```

# 安装 
apt-get install mysql-server mysql-client

#登录mysql
mysql -u root -p

# 运行远程访问 注意 youpassword
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'youpassword' WITH GRANT OPTION;
FLUSH PRIVILEGES ;

# 需要修改 /etc/mysql/my.cnf的配置
# 修改bind-address=127.0.0.1为bind-address=0.0.0.0

# 如何查看mysql数据库的端口
# 启动，并进入mysql后，输入命令：show global variables like 'port';

# 重启服务 start stop 
service mysqld restart

```

### 备份数据库表结构

命令行下具体用法如下：  mysqldump -u用戶名 -p密码 -d 數據库名 表名 脚本名;

```
	# 1.导出整个数据库  
    # mysqldump -u用户名 -p密码  数据库名 > 导出的文件名 
　　mysqldump -uroot -pmysql sva_rec  > e:\sva_rec.sql 

　　# 2.导出一个表，包括表结构和数据 

　　# mysqldump -u用户名 -p 密码  数据库名 表名> 导出的文件名 
　　mysqldump -uroot -pmysql sva_rec date_rec_drv> e:\date_rec_drv.sql 

　　# 3.导出一个数据库结构 增加 -d 参数
　　mysqldump -uroot -pmysql -d sva_rec > e:\sva_rec.sql 

    # 4.导出一个表，只有表结构 

　　mysqldump -u用户名 -p 密码 -d数据库名  表名> 导出的文件名 
　　mysqldump -uroot -pmysql -d sva_rec date_rec_drv> e:\date_rec_drv.sql 
```
### 导入数据 

[导入\导出表结构或数据](https://www.cnblogs.com/zcw-ios/articles/3319480.html)
[mysql导入导出sql文件](https://www.cnblogs.com/yuwensong/p/3955834.html)
```
#常用source 命令
#进入mysql数据库控制台，如
mysql -u root -p
mysql>use 数据库
# 然后使用source命令，后面参数为脚本文件(如这里用到的.sql)
mysql>source path/to/dbname.sql
```

### 数据库查询时间转换

FROM_UNIXTIME
```
SELECT FROM_UNIXTIME(ii.match_time/1000), ii.* FROM `intervene_info` as ii 
	where ii.adapt_id in('xxx') and ii.tindex>0 and ii.swj_email='intervene.109@xxx.cn';
```

### 修改表名、列名

[添加列，修改列，删除列](https://blog.csdn.net/ws84643557/article/details/6939846)

```
--修改表名 
alter table test rename test1; 

--添加表列 
alter table test add  column name varchar(10); 

--删除表列 
alter table test drop  column name; 

--修改表列类型 
alter table test modify address char(10) 
--修改表列类型
alter table test change address address char(40) 

--修改表列名
alter table test change  column address address1 varchar(30)
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
## Java IDE

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


## Markdown 语法练习
This is a footnote.[^1]

[Supported Content Formats aka Markdown](https://gohugo.io/content-management/formats/) 

[markdown](https://daringfireball.net/projects/markdown/syntax) 

[Markdown 语法手册](https://www.zybuluo.com/EncyKe/note/120103)


This is a footnote.[^2]

## Maven 

### editor不报错，但执行main方法时提示找不到class

可能的情况是：工程配置中的`sources`的路径配置错误或没有指定对应的文件夹。

### 无法获取到dependency的jar包

新系统下一阵慌乱：本地仓库无法修改？dependency无法下载？（恢复默认，重新加载……都不好使）

仔细检查一下：最终结果是——1. 内网搭建的仓库下没有对应的jar包；2. aliyun的仓库居然罕见的正遇上无法访问。

没有黑魔法，知道原理，明白到底发生了什么，就没什么好慌乱的了。

### maven项目默认Java编译版本
Intellij Idea 创建Maven项目时，默认的Java Language是1.5。[修改](https://www.cnblogs.com/liu-s/p/5371289.html)
```
<properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>

# 或
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.1</version>
            <configuration>
                <!-- http://maven.apache.org/plugins/maven-compiler-plugin/ -->
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```


### cannot resolve symbol

dependency中有对应的条目，到提示找不到类


不同的工程打开后使用的Maven配置不同，打开相同工程时，对应的jar包条目存在，但实际的类找不到--（没有下载到）
注意使用不同的Maven设置即可


### 依赖多个模块
依赖模块的Maven需要重新导入 `reimport all maven projects` 之后，依赖模块的maven工程才会利用自己的pom文件生成依赖。

- 下载apache-maven-3.5.0-bin.tar.gz到本地
- 解压：`tar -xvf apache-maven-3.5.0-bin.tar.gz`
- 更新配置文件：

### Maven 本地使用

```
M2_HOME=/Users/gebitang/apache-maven-3.5.0
PATH=$PATH:$JAVA_HOME/bin:/Users/gebitang/libs/:/usr/local/smlnj/bin:$M2_HOME/bin
```

- 如需要，修改conf文件夹下的settings文件中的localRepository

### 安装jar包到本地
```
mvn install:install-file -Dfile=itestin.jar -DgroupId=cn.itestin.cv -DartifactId=itestin -Dversion=170921 -Dpackaging=jar

[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building Maven Stub Project (No POM) 1
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-install-plugin:2.4:install-file (default-cli) @ standalone-pom ---
[INFO] Installing /Users/gebitang/Downloads/itestin.jar to /Users/gebitang/.m2/repository/cn/itestin/cv/itestin/170921/itestin-170921.jar
[INFO] Installing /var/folders/tw/pwg0lcfx4j79s8txc3z10dqm0000gn/T/mvninstall8572123810588485936.pom to /Users/gebitang/.m2/repository/cn/itestin/cv/itestin/170921/itestin-170921.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 0.828 s
[INFO] Finished at: 2017-09-22T10:14:08+08:00
[INFO] Final Memory: 5M/123M
[INFO] ------------------------------------------------------------------------
```

### 红色波浪线 Maven Failed to read artifact descriptor 导致dependency文件无法识别

[参考](https://stackoverflow.com/questions/6642146/maven-failed-to-read-artifact-descriptor)

在对应的根目录下执行 
```
# -U Forces a check for updated releases and snapshots on remote repositories
# run clean and install
mvn -U clean install
``` 
重新下载识别

为完成git merge，执行了`git clean -d -fx`清理后，整个工程中的配置都会消失——因会恢复到git仓库中的内容，当前目前下其他的内容都会被清理。

打开maven窗口，选择重新导入，可能出现pom文件中的dependency文件本地都有，但maven认为dependency都无法识别——显示红色波浪线。

此时执行该命令会强制重新安装。然后根据错误提示——如果本地有的jar包url仓库中没有会报错——进行对应操作即可。

### Failed to read artifact descriptor for xxx

对应的jar包无法获取到。1. 更改repository； 2. 或者不跟踪更高版本的jar包

### Dependencies version

[Reference](http://maven.apache.org/pom.html)
```
1.0: "Soft" requirement on 1.0 (just a recommendation, if it matches all other ranges for the dependency)
[1.0]: "Hard" requirement on 1.0
(,1.0]: x <= 1.0
[1.2,1.3]: 1.2 <= x <= 1.3
[1.0,2.0): 1.0 <= x < 2.0
[1.5,): x >= 1.5
(,1.0],[1.2,): x <= 1.0 or x >= 1.2; multiple sets are comma-separated
(,1.1),(1.1,): this excludes 1.1 (for example if it is known not to work in combination with this library)
```

### pom文件报错

dependency注意空格格式

## linux 环境

### Gtk-Message: 23:27:47.368: Failed to load module "canberra-gtk-module"

1.  先按照对应的lib库；
2. 依然报错后，使用 cat + more( ` cat  log-20181105232733.txt|more`)的方式可以查看。 会先报错，然后可以恢复正常。 

[Failed to load module “canberra-gtk-module” … but already installed](https://askubuntu.com/q/342202)

```
# not work
apt-get install libcanberra-gtk-module:i386
# not work 
apt install libcanberra-gtk-module libcanberra-gtk3-module

##whole log
root@testin-OptiPlex-7050:/home/testin/桌面/xcdir/_logs# apt install libcanberra-gtk
libcanberra-gtk0            libcanberra-gtk3-dev        libcanberra-gtk-common-dev  libcanberra-gtk-module
libcanberra-gtk3-0          libcanberra-gtk3-module     libcanberra-gtk-dev
root@testin-OptiPlex-7050:/home/testin/桌面/xcdir/_logs# apt install libcanberra-gtk
libcanberra-gtk0            libcanberra-gtk3-dev        libcanberra-gtk-common-dev  libcanberra-gtk-module
libcanberra-gtk3-0          libcanberra-gtk3-module     libcanberra-gtk-dev
root@testin-OptiPlex-7050:/home/testin/桌面/xcdir/_logs# apt install libcanberra-gtk-module
正在读取软件包列表... 完成
正在分析软件包的依赖关系树
正在读取状态信息... 完成
libcanberra-gtk-module 已经是最新版 (0.30-5ubuntu1)。
升级了 0 个软件包，新安装了 0 个软件包，要卸载 0 个软件包，有 6 个软件包未被升级。
root@testin-OptiPlex-7050:/home/testin/桌面/xcdir/_logs# sed -n '1,30p' log-20181105232733.txt
Gtk-Message: 23:27:47.368: Failed to load module "canberra-gtk-module"
log4j:ERROR setFile(null,true) call failed.
java.io.FileNotFoundException: /home/testin/桌面/xcdir/logs/debug.log (权限不够)
        at java.io.FileOutputStream.open0(Native Method)
        at java.io.FileOutputStream.open(FileOutputStream.java:270)
        at java.io.FileOutputStream.<init>(FileOutputStream.java:213)
        at java.io.FileOutputStream.<init>(FileOutputStream.java:133)
        at org.apache.log4j.FileAppender.setFile(FileAppender.java:294)
        at org.apache.log4j.FileAppender.activateOptions(FileAppender.java:165)
        at org.apache.log4j.DailyRollingFileAppender.activateOptions(DailyRollingFileAppender.java:223)
        at org.apache.log4j.config.PropertySetter.activate(PropertySetter.java:307)
        at org.apache.log4j.config.PropertySetter.setProperties(PropertySetter.java:172)
        at org.apache.log4j.config.PropertySetter.setProperties(PropertySetter.java:104)
        at org.apache.log4j.PropertyConfigurator.parseAppender(PropertyConfigurator.java:842)
        at org.apache.log4j.PropertyConfigurator.parseCategory(PropertyConfigurator.java:768)
        at org.apache.log4j.PropertyConfigurator.parseCatsAndRenderers(PropertyConfigurator.java:672)

```

### 修改core文件配置

[setting the core dump name schema](http://www.linuxhowtos.org/Tips%20and%20Tricks/coredump.htm)</br>
[specify filename and path for core dumps](https://sigquit.wordpress.com/2009/03/13/the-core-pattern/)</br>
[to permanently edit the core_pattern file](https://askubuntu.com/a/420628)

修改 `/etc/sysctl.conf`文件；或在`/etc/sysctl.d`文件夹下创建一个以`.conf`结尾的文件。内容为`kernel.core_pattern = core.%e.%p.%h` 也可以在这里直接指定目录

```
%e is the filename
%g is the gid the process was running under
%p is the pid of the process
%s is the signal that caused the dump
%t is the time the dump occurred
%u is the uid the process was running under
```


### 查找文件中某字符串的个数

[统计一个文件中特定字符的个数](https://blog.csdn.net/wenwenxiong/article/details/49028223)

统计一个文件中某个字符串的个数，其实就是在在一块沙地里面找石头，有的人看到石头以后，在上面做个标记（grep），然后记住自己做了多少个标记；有的人看到石头以后，把它挖了（tr：匹配不了字符串，只能去匹配单个字符），最后统计自己挖了多少石头；有的人看到石头以后，把它跳过去（awk），然后统计自己跳了多少次。


```
# -c只能统计一行的；-o 即使在一行也会被计算在内
cat /data/itestin-m/config/devices.xml |grep -o "</device>" |wc -l

# -v 去设定一个变量的值，RS是记录的分隔符，默认的是新行(\n)
# 就是说awk按照一行一行读数据，但是现在RS为'</device>'后，就按'</device>'读数据
# NR为已读的记录数，n个记录是被n-1个分隔符分开的，所以就是--NR
awk -v RS="</device>" 'END {print --NR}' config/devices.xml
```

### sed 查找替换冒号

sed如何转义单引号和双引号：如果外面是双引号，里面的双引号就要转义；写成\"，单引号则不用转义，如果外面是单引号就反过来。

```
# 替换 GRUB_CMDLINE_LINUX="" 为 GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0" 
sed -i "s/GRUB_CMDLINE_LINUX=\"\"/GRUB_CMDLINE_LINUX=\"net.ifnames=0 biosdevname=0\"/g" /etc/default/grub

```
### 修改18.04的网卡默认名称
[configure static IP address on Ubuntu 18.04](https://linuxconfig.org/how-to-configure-static-ip-address-on-ubuntu-18-04-bionic-beaver-linux)
[How to Configure Network Static IP Address in Ubuntu 18.04](https://www.tecmint.com/configure-network-static-ip-address-in-ubuntu/)
[Ubuntu 18.04 网卡配置IP](https://blog.csdn.net/peyte1/article/details/80509056)

编辑 `/etc/netplan/`文件夹下的 `/etc/netplan/01-network-manager-all.yaml` (最后的文件名可能不同)。然后执行 `netplan apply`命令立即生效

```
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: yes
    enp0s8:
      dhcp4: no
      dhcp6: no
      # /24意味着掩码为 255.255.255.0
      addresses: [192.168.56.110/24, ]
      gateway4:  192.168.56.1
      nameservers:
              addresses: [8.8.8.8, 8.8.4.4]
```
[ip段/数字，如192.168.0.1/24的意思是什么](https://blog.csdn.net/www3300300/article/details/38680843)

表示掩码用二进制表示时1的位数
比如/24表示11111111.11111111.11111111.00000000,就是255.255.255.0

### 修改16.04的网卡默认名称

[ubuntu 16.04 重命名网卡](https://blog.csdn.net/u013932687/article/details/70344686)
```
#1. ip a 或 ifconig查看网卡
#2. 查看eth名称变化
root@ubuntu:~# dmesg |grep -i eth
[    3.012281] r8169 Gigabit Ethernet driver 2.3LK-NAPI loaded
[    3.014877] r8169 0000:02:00.0 eth0: RTL8168evl/8111evl at 0xffffc90000646000, 74:d4:35:23:0f:2b, XID 0c900800 IRQ 24
[    3.016230] r8169 0000:02:00.0 eth0: jumbo features [frames: 9200 bytes, tx checksumming: ko]
[    3.090803] r8169 0000:02:00.0 enp2s0: renamed from eth0

#3. 编辑文件 etc/default/grub
# 修改 GRUB_CMDLINE_LINUX="" 为 GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"

#4. 生成新grub配置文件（该文件可能已经存在）
sudo grub-mkconfig -o /boot/grub/grub.cfg

#
# DO NOT EDIT THIS FILE
#
# It is automatically generated by grub-mkconfig using templates
# from /etc/grub.d and settings from /etc/default/grub
#

#5.编辑静态ip配置，如果是DHCP设置ip，则可以忽略此步
/etc/network/interfaces


# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto eth0
iface eth0 inet static
    address 10.34.16.93
    netmask 255.255.255.0
    network 10.34.16.0
    broadcast 10.34.16.255
    gateway 10.34.16.1
    # dns-* options are implemented by the resolvconf package, if installed
    dns-nameservers 114.114.114.114

#6. 需要重启生效

#7. 批量处理上述业务的脚本
#!/bin/bash

oldname=$(ifconfig |grep HWaddr |awk '{print $1}')

sed -i "s/GRUB_CMDLINE_LINUX=\"\"/GRUB_CMDLINE_LINUX=\"net.ifnames=0 biosdevname=0\"/g" /etc/default/grub

grub-mkconfig -o /boot/grub/grub.cfg

sed -i "s/$oldname/eth0/g" /etc/network/interfaces
```



### echo 换行 

```
# -e 参数将解析换行符，以及变量值的实际值
root@ubuntu:~# echo -e 'text\ntest'
text
test

# 更新profile
 echo 'export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64'>>/etc/profile; echo 'export JRE_HOME=${JAVA_HOME}/jre'>>/etc/profile;echo 'export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib'>>/etc/profile;echo 'export PATH=${JAVA_HOME}/bin:$PATH'>>/etc/profile

```
### 配置定时任务 crontab

[定时任务Crontab命令详解](https://www.cnblogs.com/intval/p/5763929.html)

minute hour day month week command

其中：

minute： 表示分钟，可以是从0到59之间的任何整数。

hour：表示小时，可以是从0到23之间的任何整数。

day：表示日期，可以是从1到31之间的任何整数。

month：表示月份，可以是从1到12之间的任何整数。

week：表示星期几，可以是从0到7之间的任何整数，这里的0或7代表星期日。

command：要执行的命令，可以是系统命令，也可以是自己编写的脚本文件。

```
星号（*）：代表所有可能的值，
例如month字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。

逗号（,）：可以用逗号隔开的值指定一个列表范围，
例如，“1,2,5,7,8,9”

中杠（-）：可以用整数之间的中杠表示一个整数范围，
例如“2-6”表示“2,3,4,5,6”

正斜线（/）：可以用正斜线指定时间的间隔频率，
例如“0-23/2”表示每两小时执行一次。
同时正斜线可以和星号一起使用，例如*/10，如果用在minute字段，表示每十分钟执行一次。

# crontab -e to edit this file via root user.
 crontab -l
# Edit this file to introduce tasks to be run by cron.
#
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
#
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').#
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
#
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
#
# For more information see the manual pages of crontab(5) and cron(8)
#
# m h  dom mon dow   command

```

### Make shell execute by dobule click
[How to make a shell file execute by double click ](https://askubuntu.com/questions/465531/how-to-make-a-shell-file-execute-by-double-click/465549#465549)

To run your script by double clicking on its icon, you will need to create a .desktop file for it:
```
[Desktop Entry]
Name=My script
Comment=Test hello world script
Exec=/home/user/yourscript.sh
Icon=/home/user/youricon.gif
Terminal=false
Type=Application
```
Save the above as a file on your Desktop with a .desktop extension. Change `/home/user/yourscript.sh` and `/home/user/youricon.gif` to the paths of your script and whichever icon you want it ot have respectively and then you'll be able to launch by double clicking it.

make yourscript.sh executable, and use it to run the actual script. such as:
```
#!/bin/bash
cd /home/user/Desktop/plus/
echo "to start"
exec ./run.sh
```

[参考资料](https://help.ubuntu.com/community/UnityLaunchersAndDesktopFiles)</br>
[不支持相对路径](https://stackoverflow.com/a/3879096/1087122)


### Ubuntu 显示桌面快捷键 ctrl+win+D
### ubuntu 18.04 启动自动的拼音输入法

[ubuntu 18.04 安装自带的中文输入法](https://blog.csdn.net/mint_ying/article/details/80267005)

英文页面进行安装。需要先安装输入法 `sudo apt-get install ibus-pinyin` 

然后在 settings 的 Region & Language 的 Input Sources设置栏中

1. 点击 `Manage Installed Language` ，初次进入会安装些字体等相关信息。重启后使之生效。
2. 点击 + 添加 `Chinese(Intelligent Pinyin)`。重启后使之生效。 


### CentOS Another app is currently holding the yum lock; waiting for it to exit...

```
Another app is currently holding the yum lock; waiting for it to exit...
  The other application is: PackageKit
    Memory :  31 M RSS (348 MB VSZ)
    Started: Wed Feb 14 20:49:26 2018 - 02:02 ago
    State  : Running, pid: 18008
```
[解决方案](https://www.cnblogs.com/lzxianren/p/4254059.html)

- 查找运行yum的pid，kill后再强制删除对应的pid文件 `rm -f /var/run/yum.pid`


### 安装ubuntu
[下载镜像](http://old-releases.ubuntu.com/releases/)，使用[**Rufus**](https://rufus.akeo.ie/)，[创建启动盘](https://tutorials.ubuntu.com/tutorial/tutorial-create-a-usb-stick-on-windows)。

```
1. 识别U盘，
2. 选择MBR分区方案用于BIOS或UEFI模式(MBR partition scheme for UEFI)
3. 文件系统选择FAT32、簇大小8192字节
4. 点击光盘icon，选择镜像文件，创建iso镜像
```

### vmware player安装vmware tools
提示“正在进行简易安装时，无法手动启动 vmware tools 安装”。

解决：

1. 在虚拟机配置中，将CD/DVD中的镜像移除，修改为“使用物理设备”。 然后重新选择安装vmware tools工具。——会将此工具自动挂载到CD中。
2. 进入此目录，将对应的gz包复制到本地工作目录，解压后执行`./vmware-install.pl`根据提示处理即可

提示找不到 ipconfig、gcc等，需要安装对应的工具。

```
sudo apt update
# for ipconfig
sudo apt install net-tools
# for gcc 
sudo apt install build-essential
# check gcc version
gcc --version

# check system version for gcc
cat /proc/version
Linux version 4.15.0-22-generic (buildd@lgw01-amd64-013) (gcc version 7.3.0 (Ubuntu 7.3.0-16ubuntu3)) #24-Ubuntu SMP Wed May 16 12:15:17 UTC 2018
# this means you need gcc 7.3 version

gcc --version
gcc (Ubuntu 7.3.0-16ubuntu3) 7.3.0
Copyright (C) 2017 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

```


### Linux下adb不能识别Android设备


[Linux下adb不能识别Android设备](http://blog.csdn.net/momo0853/article/details/45726583)</br>
[adb连接不上android手机的终极解决方案](http://blog.csdn.net/liuqz2009/article/details/7942569)

利用 `lsusb` 获取usb设备的vendor信息、将此信息添加到 `~/.android/adb_usb.ini` 文件下（文件不存在则手动创建）

但 `lsusb` 本身就获取不到设备就尴尬了
```
root@testin-MS-7788:~/.android# lsusb
Bus 002 Device 039: ID 093a:2510 Pixart Imaging, Inc. Optical Mouse
Bus 002 Device 006: ID 1a40:0201 Terminus Technology Inc. FE 2.1 7-port Hub
Bus 002 Device 004: ID 1a40:0101 Terminus Technology Inc. Hub
Bus 002 Device 051: ID 2b0e:179d
Bus 002 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 051: ID 1a40:0201 Terminus Technology Inc. FE 2.1 7-port Hub
Bus 001 Device 050: ID 1a40:0101 Terminus Technology Inc. Hub
Bus 001 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
root@testin-MS-7788:~/.android#
root@testin-MS-7788:~/.android#
root@testin-MS-7788:~/.android#
root@testin-MS-7788:~/.android# adb devices -l
List of devices attached
LP036878A7130052555[usb#2-1.4]          device usb:2-1.4 product:X7_CN model:LEX651 device:le_x7

root@testin-MS-7788:~/.android# adb devices -l
List of devices attached

root@testin-MS-7788:~/.android# lsusb
Bus 002 Device 039: ID 093a:2510 Pixart Imaging, Inc. Optical Mouse
Bus 002 Device 006: ID 1a40:0201 Terminus Technology Inc. FE 2.1 7-port Hub
Bus 002 Device 004: ID 1a40:0101 Terminus Technology Inc. Hub
Bus 002 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 051: ID 1a40:0201 Terminus Technology Inc. FE 2.1 7-port Hub
Bus 001 Device 050: ID 1a40:0101 Terminus Technology Inc. Hub
Bus 001 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

### tar命令基本操作

```
#打包
tar -czvf ycdh.tar.gz ycdh.jar ycdh_lib tools config log4j.properties build.properties dh.sh noLog.sh updateself.sh

#查看文件，但不解压
tar -tzvf ycdh.tar.gz

#解压
tar -zxvf ycdh.tar.gz;
```

### ubuntu多版本java支持

#### 使用ppa/源方式安装
1. 添加ppa
```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
```

2. 安装oracle-java-installer
```
sudo apt-get install oracle-java6-installer
sudo apt-get install oracle-java7-installer
sudo apt-get install oracle-java8-installer
```

查看java多版本
```
sudo update-java-alternatives -l
```
切换到其它java版本
``` 
sudo update-java-alternatives -s java-8-oracle
```

### 设置时区
```
# 安装ntp，非必要
apt-get install ntp

# tzselect timezone
tzselct 
# then select your timezone
date -R # to check the result.

cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime # make it permenent.
```

ubuntu14.04上提示错误：
```
#ubuntu14.04的一个bug
$ tzselect
/usr/bin/tzselect: line 171:/home/ubuntu/iso3166.tab: No such file or  directory
/usr/bin/tzselect: time zone files are not set up correctly

#解决办法:
vim /usr/bin/tzselect
${TZDIR=`pwd`}
改为
${TZDIR=/usr/share/zoneinfo}
```

### 设置时间 

```
# date命令将日期设置为2014年6月18日
date -s 06/18/14

# 将时间设置为14点20分50秒
date -s 14:20:50

# 将时间设置为2014年6月18日14点16分30秒（MMDDhhmmYYYY.ss）
date 0618141614.30
```

### 查看启动时间

[Linux查看系统开机时间](http://www.cnblogs.com/kerrycode/p/3759395.html)
```
#  查看最后一次系统启动的时间
gebitang@ubuntu# who -b
         system boot  2017-09-19 16:25
gebitang@ubuntu# who -r
         run-level 5  2017-09-19 16:25
gebitang@ubuntu# last reboot
reboot   system boot  4.4.0-62-generic Tue Sep 19 16:25   still running
reboot   system boot  4.4.0-62-generic Tue Sep 19 14:56   still running
reboot   system boot  4.4.0-62-generic Tue Sep 19 20:12   still running
reboot   system boot  4.4.0-62-generic Tue Sep 19 18:50   still running
# top命令的提示信息中也有
```


### 16.04root密码及用户管理

1. 默认root密码是随机的，即每次开机都有一个新的root密码
2. 在终端输命令 sudo passwd，然后输入当前用户的密码
3. 输入新的密码并确认，此时的密码就是root新密码
4. 修改成功后，输入命令 su root，再输入新的密码

#### 用户管理
[Linux查看和剔除当前登录用户](https://www.cnblogs.com/saptechnique/archive/2012/04/10/2441383.html)

root权限，更多信息，[参考](http://www.cnblogs.com/guangbei/archive/2010/04/26/1721163.html)

/etc/group 的内容包括用户组（Group）、用户组口令、GID及该用户组所包含的用户（User），每个用户组一条记录；格式如下：
`group_name:passwd:GID:user_list `
在/etc/group 中的每条记录分四个字段：〔用户组名称〕：〔用户组密码〕：〔GID〕:〔用户列表〕

###useradd 添加用户

[linux下创建用户](http://www.cnblogs.com/ylan2009/articles/2321177.html)

```
（也可以使用adduser）用来创建新的用户帐号。参数：
-d 设置新用户的登陆目录
-e 设置新用户的停止日期，日期格式为MM/DD/YY
-f 帐户过期几日后永久停权。当值为0时帐号则立刻被停权。而当值为-1时则关闭此功能。预设值为-1
-g 使新用户加入群组
-G 使新用户加入一个新组。每个群组使用逗号“，”隔开，不可以夹杂空白字
-s 指定新用户的登陆Shell
-u 设定新用户的ID值
如：useradd getbitang -d /home/gebitang -u 525 -s /bin/bash 默认新用户就在自己同名的组里
成功创建一个新用户以后，在/etc/passwd文件中就会增加一行该用户的信息，其格式如下：
〔用户名〕：〔密码〕：〔UID〕：〔GID〕：〔身份描述〕：〔主目录〕：〔登陆Shell〕
其中个字段被冒号“：”分成7各部分。
由于小于500的UID和GID一般都是系统自己保留，不用做普通用户和组的标志，
新增加的用户和组一般都是UID和GID大于500的。

```
**可以使用`usermod`修改用户的信息，如： `usermod –d/home/user2 –s/bin/bash user2`** 使用`-s /sbin/nologin`表示不用于登录

```
#查看当前登录用户，需要root权限 w
# 第一列是用户名,
# 第二列是连接的终端,tty表示显示器,pts表示远程连接,
# 第三列是登陆时间,
root@localhost:~# w
 10:48:38 up 17 days, 19:06,  1 user,  load average: 0.00, 0.02, 0.05
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    10.32.22.66      10:10    6.00s  0.15s  0.00s w

# 踢掉在线用户
pkill -kill -t pts/0

# 查看登陆用户历史
node8:/ # last
root     pts/0        10.35.0.4        Fri Jan 19 15:20   still logged in
home   pts/0        10.34.18.238     Mon Jan  8 08:08 - 08:28  (00:20)
root     pts/0        10.35.0.4        Thu Jan  4 15:39 - 15:41  (00:01)

# 新建用户 
```

### passwd 修改密码 
新用户需要制定密码才可以登录。 `passwd gebitang`
根据系统的提示信息输入两次密码，系统会显示：

> passwd ：:all authentication tokens updated successfully

删除用户： `userdel -r gebitang` -r将用户目录下的文档一并删除。在其他位置上的文档也将一一找出并删除。
新建群组：`groupadd -g 555 testgroup`
删除群组： `groupdel testgroup`


### 命令行快捷键

```
#光标移到行首， 辅助：Ctrl All必须从头开始
Ctrl + A 
#光标移到行尾， E represents end
Ctrl + E 

#向前移动一个字符 forward；在vim环境——包括man command也是vim环境——中，则表示向下翻页
Ctrl + F 
#向后移动一个字符 backward 在vim环境——包括man command也是vim环境——中，则表示向上翻页
Ctrl + B 

#向前移动一个单词 辅助：Alt意味着alternative取舍，选择。所以是按照有意义的操作进行
Alt + F 
#向后移动一个单词
Alt + B 
```

### 强制清空命令行已有数据 ctlr + c



### 配置DNS服务器
[资料](https://segmentfault.com/a/1190000002387446)

* 修改DNS
* 锁定配置文件

```shell
# 添加 阿里的dns 223.5.5.5与223.6.6.6
sudo vi /etc/resolv.conf

# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
#nameserver 127.0.1.1
nameserver 223.5.5.5
nameserver 223.6.6.6

#  chattr - change file attributes on a Linux file system
sudo chattr +i /etc/resolv.conf

#解锁 man chattr for details.
sudo chattr -i /etc/resolv.conf
```

### 查看系统版本 cat /proc/version 

```
cat /proc/version

root@testin-DH:~# lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.5 LTS
Release:	14.04
Codename:	trusty
root@testin-DH:~# cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=14.04
DISTRIB_CODENAME=trusty
DISTRIB_DESCRIPTION="Ubuntu 14.04.5 LTS"
root@testin-DH:~# cat /etc/issue
Ubuntu 14.04.5 LTS \n \l

root@testin-DH:~# cat /proc/version
Linux version 4.4.0-31-generic (buildd@lgw01-43) (gcc version 4.8.4 (Ubuntu 4.8.4-2ubuntu1~14.04.3) ) #50~14.04.1-Ubuntu SMP Wed Jul 13 01:07:32 UTC 2016
root@testin-DH:~#
```

### 测试I/O数据 dd

```
time dd bs=64k count=4k if=/dev/zero of=test
4096+0 records in
4096+0 records out
268435456 bytes (268 MB, 256 MiB) copied, 1.01641 s, 264 MB/s

real    0m1.025s
user    0m0.000s
sys     0m0.940s

```

### 查找网站ip 

```
## 10086短信被劫持
~#nslookup bjmobliis.cn
Server:         127.0.1.1
Address:        127.0.1.1#53

Non-authoritative answer:
Name:   bjmobliis.cn
Address: 45.124.113.90

# ping 
ping bjmobliis.cn
PING bjmobliis.cn (45.124.113.90) 56(84) bytes of data.
64 bytes from 45.124.113.90: icmp_seq=1 ttl=109 time=120 ms
64 bytes from 45.124.113.90: icmp_seq=2 ttl=109 time=128 ms
64 bytes from 45.124.113.90: icmp_seq=3 ttl=109 time=117 ms
64 bytes from 45.124.113.90: icmp_seq=4 ttl=109 time=120 ms
64 bytes from 45.124.113.90: icmp_seq=5 ttl=109 time=123 ms
64 bytes from 45.124.113.90: icmp_seq=6 ttl=109 time=119 ms
64 bytes from 45.124.113.90: icmp_seq=7 ttl=109 time=127 ms

```


### SSH Server安装
1. 先创建一个root账号，命令：
sudo passwd root
输入新密码 xxxxx

2. 安装ssh server. 如果是刚安装好系统，第一步是先更新一下源。
sudo apt-get update
更新完了后，安装ssh server 


3. 检查ssh server配置，允许root用户登录：

```
sudo gedit /etc/ssh/sshd_config
找到PermitRootLogin = without-password改成PermitRootLogin yes 最后执行一下
```

```
# 测试可用
sudo apt-get install ssh

#或
sudo apt-get install openssh-server --fix-missing
```

启动 sudo service ssh start

### 安装oracel jdk 8
```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
```

```
test@test:~$ sudo add-apt-repository ppa:webupd8team/java
[sudo] testin 的密码：
 Oracle Java (JDK) Installer (automatically downloads and installs Oracle JDK8). There are no actual Java files in this PPA.

Important -> Why Oracle Java 7 And 6 Installers No Longer Work: http://www.webupd8.org/2017/06/why-oracle-java-7-and-6-installers-no.html

Update: Oracle Java 9 has reached end of life: http://www.oracle.com/technetwork/java/javase/downloads/jdk9-downloads-3848520.html

The PPA supports Ubuntu 18.04, 17.10, 16.04, 14.04 and 12.04.

More info (and Ubuntu installation instructions):
- for Oracle Java 8: http://www.webupd8.org/2012/09/install-oracle-java-8-in-ubuntu-via-ppa.html

Debian installation instructions:
- Oracle Java 8: http://www.webupd8.org/2014/03/how-to-install-oracle-java-8-in-debian.html

For Oracle Java 10, see a different PPA: https://www.linuxuprising.com/2018/04/install-oracle-java-10-in-ubuntu-or.html
 更多信息： https://launchpad.net/~webupd8team/+archive/ubuntu/java
按 [ENTER] 继续或 Ctrl-c 取消安装。

命中:1 http://cn.archive.ubuntu.com/ubuntu bionic InRelease
获取:2 http://ppa.launchpad.net/webupd8team/java/ubuntu bionic InRelease [15.4 kB]
获取:3 http://security.ubuntu.com/ubuntu bionic-security InRelease [83.2 kB]
获取:4 http://cn.archive.ubuntu.com/ubuntu bionic-updates InRelease [83.2 kB]
获取:5 http://ppa.launchpad.net/webupd8team/java/ubuntu bionic/main amd64 Packages [1,556 B]
命中:6 http://cn.archive.ubuntu.com/ubuntu bionic-backports InRelease
获取:7 http://cn.archive.ubuntu.com/ubuntu bionic/main Translation-zh_CN [67.7 kB]
获取:8 http://cn.archive.ubuntu.com/ubuntu bionic/restricted Translation-zh_CN [1,188 B]
获取:9 http://cn.archive.ubuntu.com/ubuntu bionic/universe Translation-zh_CN [174 kB]
获取:10 http://security.ubuntu.com/ubuntu bionic-security/main amd64 DEP-11 Metadata [204 B]
获取:11 http://ppa.launchpad.net/webupd8team/java/ubuntu bionic/main i386 Packages [1,556 B]
获取:12 http://cn.archive.ubuntu.com/ubuntu bionic/multiverse Translation-zh_CN [4,768 B]
获取:13 http://cn.archive.ubuntu.com/ubuntu bionic-updates/main amd64 DEP-11 Metadata [12.1 kB]
获取:14 http://cn.archive.ubuntu.com/ubuntu bionic-updates/main DEP-11 48x48 Icons [8,503 B]
获取:15 http://cn.archive.ubuntu.com/ubuntu bionic-updates/main DEP-11 64x64 Icons [13.1 kB]
获取:16 http://cn.archive.ubuntu.com/ubuntu bionic-updates/universe amd64 DEP-11 Metadata [5,716 B]
获取:17 http://cn.archive.ubuntu.com/ubuntu bionic-updates/universe DEP-11 64x64 Icons [14.8 kB]
获取:18 http://security.ubuntu.com/ubuntu bionic-security/universe amd64 DEP-11 Metadata [2,456 B]
获取:19 http://ppa.launchpad.net/webupd8team/java/ubuntu bionic/main Translation-en [928 B]
已下载 491 kB，耗时 3秒 (164 kB/s)
正在读取软件包列表... 完成
testin@testin-T-001:~$ sudo apt-get update
命中:1 http://security.ubuntu.com/ubuntu bionic-security InRelease
命中:2 http://cn.archive.ubuntu.com/ubuntu bionic InRelease
命中:3 http://ppa.launchpad.net/webupd8team/java/ubuntu bionic InRelease
命中:4 http://cn.archive.ubuntu.com/ubuntu bionic-updates InRelease
命中:5 http://cn.archive.ubuntu.com/ubuntu bionic-backports InRelease
正在读取软件包列表... 完成
testin@testin-T-001:~$ sudo apt-get install oracle-java8-
oracle-java8-installer             oracle-java8-set-default           oracle-java8-unlimited-jce-policy
testin@testin-T-001:~$ sudo apt-get install oracle-java8-installer
正在读取软件包列表... 完成
正在分析软件包的依赖关系树
正在读取状态信息... 完成
将会同时安装下列软件：
  gsfonts-x11 java-common oracle-java8-set-default
建议安装：
  binfmt-support visualvm ttf-baekmuk | ttf-unfonts | ttf-unfonts-core ttf-kochi-gothic | ttf-sazanami-gothic
  ttf-kochi-mincho | ttf-sazanami-mincho ttf-arphic-uming
下列【新】软件包将被安装：
  gsfonts-x11 java-common oracle-java8-installer oracle-java8-set-default
升级了 0 个软件包，新安装了 4 个软件包，要卸载 0 个软件包，有 33 个软件包未被升级。
需要下载 54.4 kB 的归档。
解压缩后会消耗 274 kB 的额外空间。
您希望继续执行吗？ [Y/n] Y
获取:1 http://cn.archive.ubuntu.com/ubuntu bionic/main amd64 java-common all 0.63ubuntu1~02 [7,032 B]
获取:2 http://ppa.launchpad.net/webupd8team/java/ubuntu bionic/main amd64 oracle-java8-installer all 8u171-1~webupd8~0 [33.3 kB]
获取:3 http://cn.archive.ubuntu.com/ubuntu bionic/universe amd64 gsfonts-x11 all 0.25 [7,264 B]
获取:4 http://ppa.launchpad.net/webupd8team/java/ubuntu bionic/main amd64 oracle-java8-set-default all 8u171-1~webupd8~0 [6,846 B]
已下载 54.4 kB，耗时 2秒 (23.8 kB/s)
正在预设定软件包 ...
正在选中未选择的软件包 java-common。
(正在读取数据库 ... 系统当前共安装有 130445 个文件和目录。)
正准备解包 .../java-common_0.63ubuntu1~02_all.deb  ...
正在解包 java-common (0.63ubuntu1~02) ...
正在选中未选择的软件包 oracle-java8-installer。
正准备解包 .../oracle-java8-installer_8u171-1~webupd8~0_all.deb  ...
正在解包 oracle-java8-installer (8u171-1~webupd8~0) ...
正在设置 java-common (0.63ubuntu1~02) ...
正在设置 oracle-java8-installer (8u171-1~webupd8~0) ...
No /var/cache/oracle-jdk8-installer/wgetrc file found.
Creating /var/cache/oracle-jdk8-installer/wgetrc and
using default oracle-java8-installer wgetrc settings for it.
Downloading Oracle Java 8...
--2018-05-21 14:57:49--  http://download.oracle.com/otn-pub/java/jdk/8u171-b11/512cd62ec5174c3487ac17c61aaa89e8/jdk-8u171-linux-x64.tar.gz
正在解析主机 download.oracle.com (download.oracle.com)... 104.119.255.30
正在连接 download.oracle.com (download.oracle.com)|104.119.255.30|:80... 已连接。
已发出 HTTP 请求，正在等待回应... 302 Moved Temporarily
位置：https://edelivery.oracle.com/otn-pub/java/jdk/8u171-b11/512cd62ec5174c3487ac17c61aaa89e8/jdk-8u171-linux-x64.tar.gz [跟随至新的 URL]
--2018-05-21 14:57:49--  https://edelivery.oracle.com/otn-pub/java/jdk/8u171-b11/512cd62ec5174c3487ac17c61aaa89e8/jdk-8u171-linux-x64.tar.gz
正在解析主机 edelivery.oracle.com (edelivery.oracle.com)... 184.50.91.58, 2600:1417:76:19a::2d3e, 2600:1417:76:181::2d3e
正在连接 edelivery.oracle.com (edelivery.oracle.com)|184.50.91.58|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 302 Moved Temporarily
位置：http://download.oracle.com/otn-pub/java/jdk/8u171-b11/512cd62ec5174c3487ac17c61aaa89e8/jdk-8u171-linux-x64.tar.gz?AuthParam=1526885989_493b38bfe4ffc433156ef17f0d5e2d7e [跟随至新的 URL]
--2018-05-21 14:57:49--  http://download.oracle.com/otn-pub/java/jdk/8u171-b11/512cd62ec5174c3487ac17c61aaa89e8/jdk-8u171-linux-x64.tar.gz?AuthParam=1526885989_493b38bfe4ffc433156ef17f0d5e2d7e
正在连接 download.oracle.com (download.oracle.com)|104.119.255.30|:80... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度： 190890122 (182M) [application/x-gzip]
正在保存至: “jdk-8u171-linux-x64.tar.gz”

     0K ........ ........ ........ ........ ........ ........  1% 4.19M 43s
  3072K ........ ........ ........ ........ ........ ........  3% 9.83M 30s
  6144K ........ ........ ........ ........ ........ ........  4% 8.88M 26s
  9216K ........ ........ ........ ........ ........ ........  6% 8.76M 24s
 12288K ........ ........ ........ ........ ........ ........  8% 8.86M 23s
 15360K ........ ........ ........ ........ ........ ........  9% 9.32M 22s
 18432K ........ ........ ........ ........ ........ ........ 11% 9.95M 20s
 21504K ........ ........ ........ ........ ........ ........ 13% 9.24M 20s
 24576K ........ ........ ........ ........ ........ ........ 14% 8.24M 19s
 27648K ........ ........ ........ ........ ........ ........ 16% 7.98M 19s
 30720K ........ ........ ........ ........ ........ ........ 18% 8.70M 18s
 33792K ........ ........ ........ ........ ........ ........ 19% 9.55M 18s
 36864K ........ ........ ........ ........ ........ ........ 21% 9.38M 17s
 39936K ........ ........ ........ ........ ........ ........ 23% 6.52M 17s
 43008K ........ ........ ........ ........ ........ ........ 24% 7.57M 17s
 46080K ........ ........ ........ ........ ........ ........ 26% 7.98M 17s
 49152K ........ ........ ........ ........ ........ ........ 28% 8.83M 16s
 52224K ........ ........ ........ ........ ........ ........ 29% 9.33M 16s
 55296K ........ ........ ........ ........ ........ ........ 31% 9.03M 15s
 58368K ........ ........ ........ ........ ........ ........ 32% 8.49M 15s
 61440K ........ ........ ........ ........ ........ ........ 34% 8.85M 14s
 64512K ........ ........ ........ ........ ........ ........ 36% 7.66M 14s
 67584K ........ ........ ........ ........ ........ ........ 37% 8.06M 14s
 70656K ........ ........ ........ ........ ........ ........ 39% 8.51M 13s
 73728K ........ ........ ........ ........ ........ ........ 41% 8.48M 13s
 76800K ........ ........ ........ ........ ........ ........ 42% 8.69M 13s
 79872K ........ ........ ........ ........ ........ ........ 44% 7.73M 12s
 82944K ........ ........ ........ ........ ........ ........ 46% 8.65M 12s
 86016K ........ ........ ........ ........ ........ ........ 47% 9.35M 11s
 89088K ........ ........ ........ ........ ........ ........ 49% 9.55M 11s
 92160K ........ ........ ........ ........ ........ ........ 51% 8.85M 11s
 95232K ........ ........ ........ ........ ........ ........ 52% 7.21M 10s
 98304K ........ ........ ........ ........ ........ ........ 54% 7.69M 10s
101376K ........ ........ ........ ........ ........ ........ 56% 7.50M 10s
104448K ........ ........ ........ ........ ........ ........ 57% 8.09M 9s
107520K ........ ........ ........ ........ ........ ........ 59% 7.24M 9s
110592K ........ ........ ........ ........ ........ ........ 60% 7.33M 9s
113664K ........ ........ ........ ........ ........ ........ 62% 6.81M 8s
116736K ........ ........ ........ ........ ........ ........ 64% 6.65M 8s
119808K ........ ........ ........ ........ ........ ........ 65% 5.40M 8s
122880K ........ ........ ........ ........ ........ ........ 67% 6.92M 7s
125952K ........ ........ ........ ........ ........ ........ 69% 7.58M 7s
129024K ........ ........ ........ ........ ........ ........ 70% 7.91M 7s
132096K ........ ........ ........ ........ ........ ........ 72% 7.46M 6s
135168K ........ ........ ........ ........ ........ ........ 74% 8.61M 6s
138240K ........ ........ ........ ........ ........ ........ 75% 7.38M 6s
141312K ........ ........ ........ ........ ........ ........ 77% 6.23M 5s
144384K ........ ........ ........ ........ ........ ........ 79% 6.80M 5s
147456K ........ ........ ........ ........ ........ ........ 80% 7.13M 4s
150528K ........ ........ ........ ........ ........ ........ 82% 7.23M 4s
153600K ........ ........ ........ ........ ........ ........ 84% 7.10M 4s
156672K ........ ........ ........ ........ ........ ........ 85% 7.39M 3s
159744K ........ ........ ........ ........ ........ ........ 87% 6.93M 3s
162816K ........ ........ ........ ........ ........ ........ 88% 7.48M 3s
165888K ........ ........ ........ ........ ........ ........ 90% 7.64M 2s
168960K ........ ........ ........ ........ ........ ........ 92% 8.30M 2s
172032K ........ ........ ........ ........ ........ ........ 93% 8.98M 1s
175104K ........ ........ ........ ........ ........ ........ 95% 9.00M 1s
178176K ........ ........ ........ ........ ........ ........ 97% 9.14M 1s
181248K ........ ........ ........ ........ ........ ........ 98% 9.89M 0s
184320K ........ ........ ........ ........                  100% 10.2M=23s

2018-05-21 14:58:13 (7.90 MB/s) - 已保存 “jdk-8u171-linux-x64.tar.gz” [190890122/190890122])

Download done.
Removing outdated cached downloads...
update-alternatives: 错误: 无 java 的候选项
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/jre/bin/ControlPanel 来在自动模式中提供 /usr/bin/ControlPanel (ControlPanel)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/jre/bin/java 来在自动模式中提供 /usr/bin/java (java)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/jre/bin/javaws 来在自动模式中提供 /usr/bin/javaws (javaws)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/jre/bin/jcontrol 来在自动模式中提供 /usr/bin/jcontrol (jcontrol)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/jre/bin/jjs 来在自动模式中提供 /usr/bin/jjs (jjs)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/jre/bin/keytool 来在自动模式中提供 /usr/bin/keytool (keytool)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/jre/bin/orbd 来在自动模式中提供 /usr/bin/orbd (orbd)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/jre/bin/pack200 来在自动模式中提供 /usr/bin/pack200 (pack200)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/jre/bin/policytool 来在自动模式中提供 /usr/bin/policytool (policytool)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/jre/bin/rmid 来在自动模式中提供 /usr/bin/rmid (rmid)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/jre/bin/rmiregistry 来在自动模式中提供 /usr/bin/rmiregistry (rmiregistry)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/jre/bin/servertool 来在自动模式中提供 /usr/bin/servertool (servertool)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/jre/bin/tnameserv 来在自动模式中提供 /usr/bin/tnameserv (tnameserv)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/jre/bin/unpack200 来在自动模式中提供 /usr/bin/unpack200 (unpack200)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/jre/lib/jexec 来在自动模式中提供 /usr/bin/jexec (jexec)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/appletviewer 来在自动模式中提供 /usr/bin/appletviewer (appletviewer)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/extcheck 来在自动模式中提供 /usr/bin/extcheck (extcheck)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/idlj 来在自动模式中提供 /usr/bin/idlj (idlj)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/jar 来在自动模式中提供 /usr/bin/jar (jar)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/jarsigner 来在自动模式中提供 /usr/bin/jarsigner (jarsigner)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/javac 来在自动模式中提供 /usr/bin/javac (javac)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/javadoc 来在自动模式中提供 /usr/bin/javadoc (javadoc)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/javafxpackager 来在自动模式中提供 /usr/bin/javafxpackager (javafxpackager)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/javah 来在自动模式中提供 /usr/bin/javah (javah)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/javap 来在自动模式中提供 /usr/bin/javap (javap)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/javapackager 来在自动模式中提供 /usr/bin/javapackager (javapackager)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/jcmd 来在自动模式中提供 /usr/bin/jcmd (jcmd)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/jconsole 来在自动模式中提供 /usr/bin/jconsole (jconsole)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/jdb 来在自动模式中提供 /usr/bin/jdb (jdb)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/jdeps 来在自动模式中提供 /usr/bin/jdeps (jdeps)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/jhat 来在自动模式中提供 /usr/bin/jhat (jhat)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/jinfo 来在自动模式中提供 /usr/bin/jinfo (jinfo)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/jmap 来在自动模式中提供 /usr/bin/jmap (jmap)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/jmc 来在自动模式中提供 /usr/bin/jmc (jmc)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/jps 来在自动模式中提供 /usr/bin/jps (jps)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/jrunscript 来在自动模式中提供 /usr/bin/jrunscript (jrunscript)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/jsadebugd 来在自动模式中提供 /usr/bin/jsadebugd (jsadebugd)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/jstack 来在自动模式中提供 /usr/bin/jstack (jstack)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/jstat 来在自动模式中提供 /usr/bin/jstat (jstat)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/jstatd 来在自动模式中提供 /usr/bin/jstatd (jstatd)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/jvisualvm 来在自动模式中提供 /usr/bin/jvisualvm (jvisualvm)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/native2ascii 来在自动模式中提供 /usr/bin/native2ascii (native2ascii)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/rmic 来在自动模式中提供 /usr/bin/rmic (rmic)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/schemagen 来在自动模式中提供 /usr/bin/schemagen (schemagen)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/serialver 来在自动模式中提供 /usr/bin/serialver (serialver)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/wsgen 来在自动模式中提供 /usr/bin/wsgen (wsgen)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/wsimport 来在自动模式中提供 /usr/bin/wsimport (wsimport)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/bin/xjc 来在自动模式中提供 /usr/bin/xjc (xjc)
update-alternatives: 使用 /usr/lib/jvm/java-8-oracle/jre/lib/amd64/libnpjp2.so 来在自动模式中提供 /usr/lib/mozilla/plugins/libjavaplugin.so (mozilla-javaplugin.so)
Oracle JDK 8 installed

#####Important########
To set Oracle JDK8 as default, install the "oracle-java8-set-default" package.
E.g.: sudo apt install oracle-java8-set-default
On Ubuntu systems, oracle-java8-set-default is most probably installed
automatically with this package.
######################

正在选中未选择的软件包 oracle-java8-set-default。
正在处理用于 desktop-file-utils (0.23-1ubuntu3) 的触发器 ...
(正在读取数据库 ... 系统当前共安装有 130481 个文件和目录。)
正准备解包 .../oracle-java8-set-default_8u171-1~webupd8~0_all.deb  ...
正在解包 oracle-java8-set-default (8u171-1~webupd8~0) ...
正在选中未选择的软件包 gsfonts-x11。
正准备解包 .../gsfonts-x11_0.25_all.deb  ...
正在解包 gsfonts-x11 (0.25) ...
正在设置 gsfonts-x11 (0.25) ...
正在处理用于 mime-support (3.60ubuntu1) 的触发器 ...
正在设置 oracle-java8-set-default (8u171-1~webupd8~0) ...
正在处理用于 man-db (2.8.3-2) 的触发器 ...
正在处理用于 shared-mime-info (1.9-2) 的触发器 ...
正在处理用于 gnome-menus (3.13.3-11ubuntu1) 的触发器 ...
正在处理用于 hicolor-icon-theme (0.17-2) 的触发器 ...
正在处理用于 fontconfig (2.12.6-0ubuntu2) 的触发器 ...
```
### 更新profile JAVA_HOME

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

修改 全局的 profile文件 `vi /etc/profile`，执行 `source /etc/profile`可立即生效

在/etc/profile文件中添加变量【对所有用户生效（永久的）】  
用vim在文件/etc/profile文件中增加变量，该变量将会对Linux下所有用户有效，并且是“永久的”。  
要让刚才的修改马上生效，需要执行以下代码  
`source /etc/profile`
  
方法二：  
在用户目录下的.bash_profile文件中增加变量【对单一用户生效（永久的）】  
用vim在用户目录下的.bash_profile文件中增加变量，改变量仅会对当前用户有效，并且是“永久的”。  
要让刚才的修改马上生效，需要在用户目录下执行以下代码  
`source .bash_profile `

方法三：  
直接运行export命令定义变量【只对当前shell（BASH）有效（临时的）】  
在shell的命令行下直接使用[export  变量名=变量值]定义变量，该变量只在当前的shell（BASH）或其子shell（BASH）下是有效的，shell关闭了，变量也就失效了，再打开新shell时就没有这个变量，需要使用的话还需要重新定义。  

### ulimit用法 

[ulimit 用法](http://man.linuxde.net/ulimit)

```
-a：显示目前资源限制的设定；
-c <core文件上限>：设定core文件的最大值，单位为区块；
-d <数据节区大小>：程序数据节区的最大值，单位为KB；
-f <文件大小>：shell所能建立的最大文件，单位为区块；
-H：设定资源的硬性限制，也就是管理员所设下的限制；
-m <内存大小>：指定可使用内存的上限，单位为KB；
-n <文件数目>：指定同一时间最多可开启的文件数；
-p <缓冲区大小>：指定管道缓冲区的大小，单位512字节；
-s <堆叠大小>：指定堆叠的上限，单位为KB；
-S：设定资源的弹性限制；
-t <CPU时间>：指定CPU使用时间的上限，单位为秒；
-u <程序数目>：用户最多可开启的程序数目；
-v <虚拟内存大小>：指定可使用的虚拟内存上限，单位为KB。


[root@localhost ~]# ulimit -a
core file size          (blocks, -c) 0           #core文件的最大值为100 blocks。
data seg size           (kbytes, -d) unlimited   #进程的数据段可以任意大。
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited   #文件可以任意大。
pending signals                 (-i) 98304       #最多有98304个待处理的信号。
max locked memory       (kbytes, -l) 32          #一个任务锁住的物理内存的最大值为32KB。
max memory size         (kbytes, -m) unlimited   #一个任务的常驻物理内存的最大值。
open files                      (-n) 1024        #一个任务最多可以同时打开1024的文件。
pipe size            (512 bytes, -p) 8           #管道的最大空间为4096字节。
POSIX message queues     (bytes, -q) 819200      #POSIX的消息队列的最大值为819200字节。
real-time priority              (-r) 0
stack size              (kbytes, -s) 10240       #进程的栈的最大值为10240字节。
cpu time               (seconds, -t) unlimited   #进程使用的CPU时间。
max user processes              (-u) 98304       #当前用户同时打开的进程（包括线程）的最大个数为98304。
virtual memory          (kbytes, -v) unlimited   #没有限制进程的最大地址空间。
file locks                      (-x) unlimited   #所能锁住的文件的最大个数没有限制。
```

### lsof 打开文件过多

[检查以及处理](https://blog.csdn.net/roy_70/article/details/78423880)

```
#1. 查看进程 
ps aux |grep java

#2. 查看进程打开的文件
lsof -p pid

#3. 修改ulimit条件
```

## Shell commands

任何场景下，使用
```
man command
``` 
查看命令详情 

### 使用jq格式化json

需要先安装 `sudo apt install -u jq`

```
cat config/devices.txt |jq . 

{
    "brand": "360",
    "displayed": true,
    "idealName": " 10.35.16.20 [360 1703-M01] _1-5 [864848031310516 23] ",
    "imei": "864848031310516",
    "keyID": "10.35.16.20__1-5",
    "model": "1703-M01",
    "online": true,
    "position": {
      "column": 7,
      "row": 2
}
```

### 收集进程的线程详细信息

```
#!/bin/bash
controller=$(ps -aux|grep java |grep -F "$1"|grep -vF grep|awk '{printf $0}' |awk '{print $2}')
top -b -n 1 -H -p $controller |grep Threads

# -b  :Batch-mode operation
#            Starts top in 'Batch' mode, which could be useful for sending output from top to other programs or to a file.  In this mode, top  will
#            not accept input and runs until the iterations limit you've set with the '-n' command-line option or until killed.
# -n  :Number-of-iterations limit as:  -n number

```

### 查看进程的线程信息
```
top -H -p <pid>
root@test:/root# top -H -p 12091
top - 10:33:34 up 12:58,  2 users,  load average: 14.39, 15.31, 16.61
Threads: 106 total,   4 running, 102 sleeping,   0 stopped,   0 zombie
%Cpu(s): 68.4 us,  0.4 sy,  0.0 ni,  6.3 id, 24.7 wa,  0.0 hi,  0.2 si,  0.0 st
KiB Mem : 16285716 total,   891832 free, 10665560 used,  4728324 buff/cache
KiB Swap:  2097148 total,  2097148 free,        0 used.  4740208 avail Mem

进▒ USER      PR  NI    VIRT    RES    SHR ▒ %CPU %MEM     TIME+ COMMAND
13503 root      20   0 14.222g 9.102g  26088 R 99.9 58.6 445:00.85 java
24782 root      20   0 14.222g 9.102g  26088 R 99.9 58.6 315:46.25 java
25118 root      20   0 14.222g 9.102g  26088 R 99.9 58.6 299:22.98 java
26443 root      20   0 14.222g 9.102g  26088 R 99.9 58.6 173:56.59 java
12104 root      20   0 14.222g 9.102g  26088 S 62.2 58.6 125:17.54 java
16155 root      20   0 14.222g 9.102g  26088 S  2.4 58.6   0:02.87 java
16154 root      20   0 14.222g 9.102g  26088 D  1.2 58.6   0:02.53 java
12117 root      20   0 14.222g 9.102g  26088 S  0.4 58.6   1:25.10 java
16136 root      20   0 14.222g 9.102g  26088 D  0.4 58.6   0:08.15 java
16157 root      20   0 14.222g 9.102g  26088 S  0.4 58.6   0:03.89 java
16285 root      20   0 14.222g 9.102g  26088 S  0.4 58.6   0:05.48 java
12091 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:00.00 java
12093 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:02.23 java
12094 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:24.36 java
12095 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:24.78 java
12096 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:23.79 java
12097 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:23.89 java
12098 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:24.28 java
12099 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:24.99 java
12100 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:24.57 java
12101 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:23.62 java
12102 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:33.86 java
12103 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   2:36.90 java
12105 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   1:14.74 java
12106 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:00.96 java
12107 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:00.87 java
12108 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:00.05 java
12109 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:00.00 java
12110 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:37.36 java
12111 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:39.42 java
12112 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:42.24 java
12113 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:15.51 java
12114 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:00.00 java
12115 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:12.55 java
12116 root      20   0 14.222g 9.102g  26088 S  0.0 58.6  12:19.01 java
12118 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   6:45.93 java
12119 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:00.14 java
12120 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:04.27 java
12121 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:00.00 java
12150 root      20   0 14.222g 9.102g  26088 S  0.0 58.6   0:00.00 java

```
### /dev/null 2>&1含义

[What is /dev/null 2>&1?](https://stackoverflow.com/q/10508843/1087122)</br>
[2>&1>/dev/null](https://blog.csdn.net/zhongqi2513/article/details/78613768)</br>
[shell中命令>/dev/null 2>&1含义](http://tang9140.iteye.com/blog/2254020)</br>

标准输出被重定向到空的设备文件，即不显示，标准错误输出被重定向到标准输出，而标准输出已被重定向到空的设备文件，所以标准错误输出也不显示。

标准输入0    从键盘获得输入 /proc/self/fd/0 </br>
标准输出1    输出到屏幕（即控制台） /proc/self/fd/1 </br>
错误输出2    输出到屏幕（即控制台） /proc/self/fd/2 </br>

### split & cat使用


```
-b：值为每一输出档案的大小，单位为 byte。
-C：每一输出档中，单行的最大 byte 数。
-d：使用数字作为后缀。
-l：值为每一输出档的列数大小。

split -b 102400k all.log
~# ls
all.log   xaa  xab  xac  xad

cat xaa xab xac xad  >back.log
~# md5sum back.log
ab9a814d0d7f412555c48689d6037050  back.log
~# md5sum all.log
ab9a814d0d7f412555c48689d6037050  all.log
```


### 使用zsh

1. 安装 `sudo apt-get install zsh` 
2. 配置默认的sh `chsh -s /bin/zsh` 或 `chsh -s 'which zsh'`，切换为旧bash `chsh -s /bin/bash`
3. 安装 oh-my-zsh [github repo](https://github.com/robbyrussell/oh-my-zsh)

```
# curl方式
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

# wget 方式
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"

```

### 端口占用
利用netstat 或fuser 命令查找
```
netstat -ano |grep 9988
# -k 参数可以kill相应的进程
fuser 9988/tcp
##java可利用ManagementFactory.getRuntimeMXBean().getName()获取进程
```

0x0. 启动terminal 快捷键  
```
Ctrl+Alt+T
```

### 0x1. 等待
	
    sleep 1 #睡眠1秒 默认
	sleep 1s #睡眠1秒
	sleep 1m #睡眠1分
	sleep 1h #睡眠1小时
	

### 0x2. 日期
```
#%s表示当前时间的秒数，而%N表示当前时间的纳秒部分，即1秒以下的那部分
joechin@ubuntu:~$ date +%Y%m%d%H%M%S%N
20170824115956519750498
joechin@ubuntu:~$ date +%Y%m%d%H%M%S
20170824115957

```

### 0x3. 判断文件夹、进程是否存在

```
#!/bin/bash

# Create dir logs if it doesn't exist
if [ ! -d "./_logs" ]
then
    mkdir ./_logs
fi

# 判断是否存在文件 
# ls所有以.txt为后缀的文件，如果不存在，将标准错误重定向到标准输出，
# 2>&1 的意思就是将标准错误也输出到标准输出当中。
# 重定向中 0-标准输出，1-标准输出，2-标准错误，而No such file or directory是一个标准错误。
if ls *.txt >/dev/null 2>&1; then
　　echo "file exist."
else
    echo "no txt file"
fi

controller=$(ps -aux|grep -F "ycdh"|grep -vF grep|awk '{printf $0}' |awk '{print $2}')

# 中括号前后必须有空格，否则报 [: missing `]' 错误
# 需要两个中括号，因为当变量为空时，会报 "[: =: unary operator expected" 错误
if [[ $controller -ne 0 ]] ; then
   echo "try to kill $controller"
   kill -9 $controller
else
  echo "ycdh not running."
fi

```

```
#!/bin/sh  
  
myPath="/var/log/httpd/"  
myFile="/var /log/httpd/access.log"  
  
#这里的-x 参数判断$myPath是否存在并且是否具有可执行权限  
if [ ! -x "$myPath"]; then  
　　mkdir "$myPath"  
fi  

#这里的-d 参数判断$myPath是否存在  
if [ ! -d "$myPath"]; then  
　　mkdir "$myPath"  
fi  
  
#这里的-f参数判断$myFile是否存在  
if [ ! -f "$myFile" ]; then  
　　touch "$myFile"  
fi  
  
#其他参数还有-n,-n是判断一个变量是否是否有值  
if [ ! -n "$myVar" ]; then  
　　echo "$myVar is empty"  
　　exit 0  
fi  
  
#两个变量判断是否相等  
if [ "$var1" = "$var2" ]; then  
　　echo '$var1 eq $var2'  
else  
　　echo '$var1 not eq $var2'  
fi  


-e filename  如果 filename存在，则为真  [ -e /var/log/syslog ]
-d filename  如果 filename为目录，则为真  [ -d /tmp/mydir ]
-f filename  如果 filename为常规文件，则为真  [ -f /usr/bin/grep ]
-L filename  如果 filename为符号链接，则为真  [ -L /usr/bin/grep ]
-r filename  如果 filename可读，则为真  [ -r /var/log/syslog ]
-w filename  如果 filename可写，则为真  [ -w /var/mytmp.txt ]
-x filename  如果 filename可执行，则为真  [ -L /usr/bin/grep ]
filename1-nt filename2  如果 filename1比 filename2新，则为真 
 [ /tmp/install/etc/services -nt /etc/services ]
filename1-ot filename2  如果 filename1比 filename2旧，则为真  
 [ /boot/bzImage -ot arch/i386/boot/bzImage ]

```

### 0x4. 线程控制批量执行
- 循环体的命令用&符号放入后台运行实现多线程效果
- 利用有名管道特性实现线程控制

```
#!/bin/bash
# must /bin/bash add method not work while using /bin/sh

# http://www.cnblogs.com/chenjiahe/p/6268853.html
#定义脚本运行的开始时间
start_time=`date +%s`
# need to make sure root user can remote login
passwd='passwordForRoot'
#创建有名管道
[ -e /tmp/fd1 ] || mkfifo /tmp/fd1 
#创建文件描述符，以可读（<）可写（>）的方式关联管道文件，文件描述符9就有了有名管道文件的所有特性
#has to be number?
exec 9<>/tmp/fd1                   
#关联后的文件描述符拥有管道文件的所有特性,所以这时候管道文件可以删除，留下文件描述符即可
rm -rf /tmp/fd1                    
for ((i=1;i<=5;i++))
do
#&9代表引用文件描述符9，这条命令代表往管道里面放入了一个"令牌"
    echo >&9                   
done

for ip in 10.34.16.25 10.34.16.27 10.34.7.200
do
#代表从管道中读取一个令牌
read -u9
{
	if ping -c 1 -W 5 $ip > /dev/null;then
			#http://www.cnblogs.com/autopenguin/p/5918717.html
			/usr/bin/expect <<-EOF
			set timeout -1

			spawn ssh root@$ip "cd /data/itestin-m/; bash /data/itestin-m/dh.sh"
			expect {
					"*yes/no" { send "yes\r"; exp_continue }
					"*password:" { send "$passwd\r" }
			}
			expect eof

			EOF
			echo -e "~~~~good $ip~~~\n"
	else
			echo -e "=====can't connect@@@@$ip==\n\n"
	fi
	#代表命令执行到最后，把令牌放回管道
	echo >&9   
}&                           
done
wait

stop_time=`date +%s`  #定义脚本运行的结束时间
 
echo "TIME:`expr $stop_time - $start_time`"
#关闭文件描述符的读，必须分开写
exec 9<&-
#关闭文件描述符的写           
exec 9>&-

```

### 0x5. 过滤logcat输出并保存到文件
```shell
#使用 --line-buffered 参数 https://stackoverflow.com/a/18146279/1087122
adb logcat -v time | grep --line-buffered UIA > log.log
```

### 0x6. 查看端口占有
```
# 查看已经连接的服务端口（ESTABLISHED）
netstat -a

#查看所有的服务端口（LISTEN，ESTABLISHED）
netstat -ap

# 查看具体哪个进程占有对应的端口，端口号为特指
lsof -i:9988
```

### 0x7. apt-get安装卸载
```
# 查看已安装应用 dpkg -l配合grep
dpkg -l |grep 'deja-dup'

# 卸载应用 -q for quiet， 可以通过 man apt-get 搜索 -qq查看更多信息
apt-get -q remove deja-dup
```


### 0x8. Shell接受参数

接收来自命令行传入的参数，第一个参数用$1表示，第二个参数$2表示，。。。以此类推。注意：$0表示脚本文件名。另外一个在shell编程中经常用到 的是“$@”这个代表所有的参数，。你可以用一个循环来遍历这个参数。如果用java来类比的话，可以把$@看作是man函数中定义的那个数组
```
#!/bin/bash
for args in $@
  do
  	echo $args
  done
```

### 0x9. Shell自动登录expect
```
# 需要安装expect
#!/usr/bin/expect
spawn ssh root@192.168.22.194
expect "*password:"
send "123\r"
expect "*#"
interact
```

### 0xa. for循环读取文件作为变量
```
for line in `cat filename`
do
  echo $line
done
```
可配合上面的使用，变量单独存在文件中

### 0xb. 查找包含某字符的所有文件
```
grep -rn "word to find" *
# * 表示当前目录所有文件，支持通配符
# -r 递归查找
# -n 显示行号
# -R 查找当前目录的所有子目录， 深递归
# -i 忽略大小写
```
### 0xc. 显示n到m行的文件内容，或n到结尾的内容
```
sed -n '100, 150p' path/to/filename

sed '100,$!d' path/to/filename
#或
sed '100,$!p' path/to/filename
```
### 0xd. shell中调用另外一个shell脚本

[fork exec source](http://blog.csdn.net/l_nan/article/details/21874679)

fork方式为直接执行，直接写被调用脚本的路径即可。在子命令执行完后再执行父级命令。子级的环境变量不会影响到父级。

exec path/to/call/script.sh 执行子级的命令后，不再执行父级命令

source path/to/call/sciprt.sh 执行子级命令后继续执行父级命令，同时子级设置的环境变量会影响到父级的环境变量。

### 0xe. awk基本使用

[详解](http://www.cnblogs.com/ggjucheng/archive/2013/01/13/2858470.html)

读入有'\n'换行符分割的一条记录，然后将记录按指定的域分隔符划分域，填充域，$0则表示所有域,$1表示第一个域,$n表示第n个域。默认域分隔符是"空白键" 或 "[tab]键"

可以使用 -F 指定分割符合

```
awk -F: '{print $7}' /etc/passwd
# root:x:0:0:root:/root:/bin/bash
/bin/bash
# 以:为分隔符，打印第7个，所有输出 /bin/bash

```

### 0xf. du -h 查看文件具体大小

[Linux下查看文件和文件夹大小](https://www.cnblogs.com/benio/archive/2010/10/13/1849946.html)

df命令可以显示目前所有文件系统的可用空间及使用情形

du：查询文件或文件夹的磁盘使用空间
```
test@pp:/data/itestin-m# du -h --max-depth=1 .
4.0K	./auto_update
386M	./tools
861M	./data
1.6G	./logs
18G	./download
1.3G	./temp
4.0K	./user
340K	./config
91M	./ycdh_lib
4.0K	./script
4.0K	./tmpp
4.0K	./report
6.2G	./_logs
29G	.
```

### grep 之后只显示部分行数 grep '' | head -200

### 处理 Binary file xxx.log matches
grep认为这是二进制文件，解决方案：grep -a
```
cat log-20171218074617.txt |grep -a c33a1478fa479153124f938541022398
```

### dig查看域名解析

```
dig qq.com +nostats +nocomments +nocmd

; <<>> DiG 9.9.7-P3 <<>> qq.com +nostats +nocomments +nocmd
;; global options: +cmd
;qq.com.				IN	A
qq.com.			600	IN	A	125.39.240.113
qq.com.			600	IN	A	61.135.157.156
qq.com.			17879	IN	NS	ns4.qq.com.
qq.com.			17879	IN	NS	ns2.qq.com.
qq.com.			17879	IN	NS	ns1.qq.com.
qq.com.			17879	IN	NS	ns3.qq.com.
ns1.qq.com.		3706	IN	A	101.226.68.138
ns1.qq.com.		3706	IN	A	14.17.19.139
ns2.qq.com.		18704	IN	A	101.227.169.106
ns3.qq.com.		106736	IN	A	182.140.177.149
ns3.qq.com.		106736	IN	A	182.140.167.157
ns4.qq.com.		106736	IN	A	123.151.178.115
ns4.qq.com.		106736	IN	A	125.39.247.247
ns4.qq.com.		106736	IN	A	184.105.206.124
ns4.qq.com.		106736	IN	A	203.205.144.156
```

### 统计当前文件夹下文件的个数

```
#1、 统计当前文件夹下文件的个数
ls -l |grep "^-"|wc -l

#2、 统计当前文件夹下目录的个数
ls -l |grep "^d"|wc -l

#3、统计当前文件夹下文件的个数，包括子文件夹里的 
ls -lR|grep "^-"|wc -l

#4、统计文件夹下目录的个数，包括子文件夹里的
ls -lR|grep "^d"|wc -l
```

### Argument list too long， use xargs or exec
传递给ls, rm等命令的参数过长。
```
#递归查找所有pdf文件并删除
find . -name "*.pdf" -print0 | xargs -0 rm

# xargs 命令是给其他命令传递参数的一个过滤器，-i会将xargs的内容赋值给{}。
find . -name "*.jpeg" | xargs -i rm {}

# 非递归，指定查找深度
find . -maxdepth 1 -name "*.jpeg" -print0 | xargs -0 rm

## 问题：要拷贝test文件夹下以jpg结尾的文件到train目录。
#命令1为：
find test/ -name "*.jpg" | xargs -i cp {} train

#命令2为：
find test/ -name "*.jpg" -exec cp {} train \;

```

### linux修改文件所有者和文件所在组
 
```
# change group
chgrp -R  用户名    文件名 
# need to run with root 
chown -R 用户名   文件名  -R

#     r： 4（读权限）
#     w： 2（写权限）
#     x： 1（执行权限）
# 将这些数字相加，就得到每一组的权限值，例如
-rw-r--r--  1 myy groupa 0 Sep 26 06:07 filed
第一组（user）：rw- = 4+2+0 = 6
第二组（group）：r-- = 4+0+0 = 4
第三组（others）：r-- = 4+0+0 = 4
```

###zsh 安装使用

[zsh安装使用](https://blog.csdn.net/gatieme/article/details/52741221)
```
sudo apt-get install zsh
#进入zsh
zsh

#安装oh-my-zsh
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"1

```

### 命令行直接打开指定文件目录
```
# 安装 
sudo apt install libgnome2-bin 

#使用 gnome-open path
gnome-open path/to/destination
```

###命令行复制粘贴 Ctrl + Shift + C / V 

## Git命令使用
可配合GUI工具和命令行工具参考。

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
# add the_abs_path_of_the_new_RSA_file, such as ~/.ssh/coding_id_rsa
ssh-add coding_id_rsa

# then it could be cloned normaly.
```


### Fork and Sync Repo

```
1. git remote add upstream  https://github.com/url/project.git
2. git fetch upstream
3. git merge upstream/master
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


### ssh config 示例
[SSH CONFIG FILE](https://www.ssh.com/ssh/config/)</br>
[OpenSSH Config File Examples](https://www.cyberciti.biz/faq/create-ssh-config-file-on-linux-unix/)

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
ssh-keygen -t rsa -C "email@address.com"
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

## Windows命令

### 修改用户名


[How to Change Your Account Name in Windows 10](https://www.groovypost.com/howto/change-account-name-windows-10/)</br>
[How to Find Security Identifier (SID) of User in Windows](https://www.tenforums.com/tutorials/84467-find-security-identifier-sid-user-windows.html)</br>
[Security identifiers official doc](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/security-identifiers)

查看SID值：使用管理员打开cmd目录，执行`wmic useraccount`会展示所有用户的列表信息

```
PS D:\> whoami /user

用户信息
----------------

用户名           SID
================ ==============================================
gebitang\joechin S-1-5-21-3801529287-1954174758-2671260994-1001
PS D:\>
```

基本流程：</br>

1. 新建一个管理员账号并使用这个账号登陆
2. 修改原用户的名称
3. 修改注册码`regedit `修改`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Profilelist `下原用户名的security identifier (SID)的ProfileImagePath的值。
4. 重启切换到原用户登陆


上面的操作可以实际修改了用户名，但在系统设置-->环境变量中的UI显示还是修改之前的名称。可以使用`netplwiz`修改显示的用户名

Press Windows key + R, type: `netplwiz` or `control userpasswords2` then hit Enter. Select the account then click Properties.

### win10 鼠标一直显示转圈
[windows 10 鼠标光标经常出现蓝色转圈](https://answers.microsoft.com/zh-hans/windows/forum/windows_10-performance/windows-10/6f0f590c-997b-4daf-9b7b-7280dcdb080b)

通常为有应用一直在占用资源。我遇到的是 小米wifi网口一直未启动导致的（拔掉小米wifi后恢复正常），在设备管理器-->网络适配器中的adapter有个向下的箭头，右键可以选择启用设备。启用设备后，转圈提示消失。

### 系统自检wifi热点（需要无线网卡支持）

推荐使用小米wifi：即可以提供wifi热点给其他设备使用；又可以作为无线网卡连接其他wifi上网。

```
#0. 使用管理员权限运行cmd命令行
#1. 检查是否包含无线网卡 （没有无线网卡的就不可用）
netsh wlan show driver

接口名称: WLAN

    驱动程序                  : Broadcom BCM943228HMB 802.11abgn 2x2 Wi-Fi Adapter
    供应商                    : Broadcom
    提供程序                  : Broadcom
    日期                      : 2014/6/9
    版本                      : 6.30.223.245
    INF 文件                  : C:\windows\INF\oem46.inf
    文件                      : 4 total
                                C:\windows\system32\DRIVERS\BCMWL63a.SYS
                                C:\windows\system32\bcmihvsrv64.dll
                                C:\windows\system32\bcmihvui64.dll
                                C:\windows\system32\drivers\vwifibus.sys
    类型                      : 本机 WLAN 驱动程序
    支持的无线电类型          : 802.11n 802.11a 802.11g 802.11b
    支持 FIPS 140-2 模式: 是
    支持 802.11w 管理帧保护 : 是
    支持的承载网络  : 是
    基础结构模式中支持的身份验证和密码:
                                开放式             无
                                开放式             WEP
                                WPA2 - 企业       TKIP
                                WPA2 - 个人       TKIP
                                WPA2 - 企业       CCMP
                                WPA2 - 个人       CCMP
                                供应商定义的          供应商定义的
                                供应商定义的          供应商定义的
                                WPA - 企业        TKIP
                                WPA - 个人        TKIP
                                WPA - 企业        CCMP
                                WPA - 个人        CCMP
    临时模式中支持的身份验证和密码:
                                WPA2 - 个人       CCMP
                                开放式             无
                                开放式             WEP
    是否存在 IHV 服务         : 是
    IHV 适配器 OUI            : [00 10 18]，类型: [00]
    IHV 扩展 DLL 路径         : C:\windows\System32\bcmihvsrv64.dll
    IHV UI 扩展 ClSID         : {aaa6dee9-31b9-4f18-ab39-82ef9b06eb73}
    IHV 诊断 CLSID            : {00000000-0000-0000-0000-000000000000}

#2. 创建无线热点 网络连接中会多出一个网卡“Microsoft Virtual WiFi Miniport Adapter”
# mode：是否启用虚拟WIFI网卡，改为disallow则为禁用。
# ssid：无线网名称，最好用英文（以cai为例）。
# key：无线网密码，
netsh set hostednetwork mode=allow ssid=nameOfwifi key=pwdForssid
承载网络模式已设置为允许。
已成功更改承载网络的 SSID。
已成功更改托管网络的用户密钥密码。

 
#3. 启动/停止此连接 netsh wlan start/stop hostednetwork
C:\windows\system32>netsh wlan start hostednetwork
已启动承载网络。

#4. 设置本地的有线连接为共享模式
在“网络连接”窗口中，右键单击已连接到Internet的网络连接，
选择“属性”→“共享”，勾上“允许其他······连接（N）”并选择新创建的wifi连接点

```
netsh wlan set hostednetwork mode=allow ssid=cai key=12345678

### 开机自启动

```
#1. 编辑注册表 Ctrl+R --> regedit 
计算机\HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run

#2. 右键新建 "字符串"" 类型，自定义名字
#3. 知道该名称的数据值，即要启动应用的全目录，如 F:\Tools\ss\Shadowsocks.exe

# win10环境可以直接启动powershell类型脚本。
# a). 新建脚本保存为.ps1的后缀; b). 修改默认打开应用，选择 powershell
#（默认路径在 %SystemRoot%\system32\WindowsPowerShell\v1.0\powershell.exe）
# 即 %HOMEDRIVE%%HOMEPATH% = C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

### 获取时间戳
[获取系统日期、时间戳记](http://atgfss.iteye.com/blog/354054)</br>
[get current time in windows command line](https://stackoverflow.com/questions/203090/how-do-i-get-current-datetime-on-the-windows-command-line-in-a-suitable-format/)


格式： %date:~x,y%以及%time:~x,y% </br>
说明： x是开始位置，y是取得字符数 </br>
```
# 获取完整的日期和时间， 
格式： %date:~0,4%%date:~5,2%%date:~8,2%%time:~0,2%%time:~3,2%%time:~6,2% 
结果： 20180316175313 
```

### bat注释 
[bat 的注释方法](https://blog.csdn.net/zhangmiaoping23/article/details/56839106)

在批处理中，段注释有一种比较常用的方法：

     goto start
      = 可以是多行文本，可以是命令
      = 可以包含重定向符号和其他特殊字符
      = 只要不包含 :start 这一行，就都是注释
     :start

这样会跳过之间的三行，也就相当于注释


另外，还有其他各种注释形式，比如：

1. :: 注释内容（第一个冒号后也可以跟任何一个非字母数字的字符）
2. rem 注释内容（不能出现重定向符号和管道符号）
3. echo 注释内容（不能出现重定向符号和管道符号）〉nul
4. if not exist nul 注释内容（不能出现重定向符号和管道符号）
5. :注释内容（注释文本不能与已有标签重名）
6. %注释内容%（可以用作行间注释，不能出现重定向符号和管道符号）
7. goto 标签 注释内容（可以用作说明goto的条件和执行内容）
8. :标签 注释内容（可以用作标签下方段的执行内容）

### CMD重定向

[cmd命令的重定向输出](https://www.cnblogs.com/dewin/archive/2009/09/19/1569869.html)
```
# 2>&1
java -Dshow.debug.info=true -jar ./App.jar >> log-%date:~0,4%%date:~5,2%%date:~8,2%%time:~0,2%%time:~3,2%%time:~6,2%.txt 2>&1
```

### 杀死进程
```
#命令行杀死进程
#查看进程 
tasklist
   
#杀死进程 查看帮助 taskkill /?
taskkill /PID 11112
# 强制kill
taskkill /F /PID 4684
taskkill /IM notepad.exe
```

### bat目录提示‘此处不该有xxx’
```
@echo off
set port=%1
set EXISTS_FLAG=false
for /f "tokens=2" %%i in ('netstat -ano ^|findstr ":%port%"') do echo %%i|findstr /E ":%port%" && set EXISTS_FLAG=true && exit
```
for/f中的命令如果有特殊字符需要加转义字符^，您的批处理改成这样就行了。

### 提高CMD显示行数

修改属性、布局、屏幕缓冲区大小、高度。

如果命令行输出被截断，进行重定向吧:(

mode命令显示当前窗口设置状态
```
F:\>mode

设备状态 CON:
---------
    行:　       3000
    列:　　     80
    键盘速度:   31
    键盘延迟:　 1
    代码页:     936

```

### 窗口保护色 #c7edcc

[在线RGB取色器](http://xiaohudie.net/RGB.html)

[修改窗口背景颜色为护眼颜色](https://blog.csdn.net/u010739973/article/details/45675515)，需要重启生效。

```
202 234 206 #caeace
199 237 204 #c7edcc

# HKEY_CURRENT_USER\Control Panel\Colors\下的Window的值默认为255 255 255（白色）， 设置为199 237 204（浅绿）
# HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Themes\DefaultColors\Standard下的Window的数值默认 ffffff，修改为
```

### CMD here for win10

[How to return the 'Open command window here' option to Windows 10's context menu](https://www.windowscentral.com/add-open-command-window-here-back-context-menu-windows-10)这个链接操作了半天还是不好使。直接修改注册表吧，不能迷信外语资料:)

报错下面的内容到文本文件，重命名为.reg，双击执行即可。 

说明：修改不同的注册表值。
```

```
给以下注册表添加了新内容：
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\background\shell]
增加一个条目cmd_here：指定名称、icon和执行的命令

[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Folder\shell]
增加一个条目cmdPrompt：指定名称、icon和执行的命令

[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\shell]
增加一个条目cmd_here：指定名称、icon和执行的命令
```
Windows Registry Editor Version 5.00
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\background\shell\cmd_here]
@="cmd here"
"Icon"="cmd.exe"
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\background\shell\cmd_here\command]
@="\"C:\\Windows\\System32\\cmd.exe\""
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Folder\shell\cmdPrompt]
@="cmd here"
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Folder\shell\cmdPrompt\command]
@="\"C:\\Windows\\System32\\cmd.exe\" \"cd %1\""
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\shell\cmd_here]
@="cmd here"
"Icon"="cmd.exe"
[HKEY_LOCAL_MACHINE\SOFTWARE\Classes\Directory\shell\cmd_here\command]
@="\"C:\\Windows\\System32\\cmd.exe\""

```


### Notepad++ 插件管理

默认的安装是不带PluginManager的，官方下载，指向到[github地址](https://github.com/bruderstein/nppPluginManager/releases)。 32为的选择后缀为UNI的，下载解压到对应的安装目录plugin、update文件夹下。

通常用的三个插件：compare、JSON Viewer、XML tools。通过PM安装、重启即可。

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
