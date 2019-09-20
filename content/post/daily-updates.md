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


## Mac 相关

### Chrome全屏 Command+Shift +F
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

# 查看内存占用

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

## Mysql

### install on Ubuntu 18.04

[How to Install MySQL on Ubuntu 18.04
This tutorial is also available for:](https://linuxize.com/post/how-to-install-mysql-on-ubuntu-18-04/)  

```
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost' IDENTIFIED BY 'very_strong_password';
```

### 重置auto_increment初始值

[TRUNCATE TABLE 语法](https://blog.csdn.net/weixin_36210698/article/details/70176652)
```
-- 1. 将数据全部删除，而且重新定位自增的字段
truncate table 你的表名

-- 2. 直接重置autoIncrement的值
ALTER TABLE table_name AUTO_INCREMENT = 1;
-- 可以先删除表内容 
delete from table_name;
```

### 复制表结构

[复制表结构](https://www.cnblogs.com/chenmh/p/5644644.html)

```
-- like方法能一模一样的将一个表的结果复制生成一个新表，
-- 包括复制表的备注、索引、主键外键、存储引擎等。
CREATE TABLE IF NOT EXISTS taskInterveneData LIKE taskRecord;

-- select 方法将复制完整的数据和结构 
-- 其它表属性都有系统的配置文件决定；包括存储引擎、默认字符集等都是有系统的默认配置所决定
CREATE TABLE IF NOT EXISTS tb_base_select
AS
SELECT *
FROM tb_base;
```


### 查询数据库大小

[SQL命令查看Mysql数据库大小](https://blog.csdn.net/atec2000/article/details/7041352)

```
1、进入information_schema 数据库（存放了其他的数据库的信息）
use information_schema;

2、查询所有数据的大小：
SELECT concat(round(SUM(data_length / 1024 / 1024), 2), 'MB') AS data
FROM tables;

3、查看指定数据库的大小：
比如查看数据库home的大小
SELECT concat(round(SUM(data_length / 1024 / 1024), 2), 'MB') AS data
FROM tables
WHERE table_schema = 'home';

4、查看指定数据库的某个表的大小
比如查看数据库home中 members 表的大小
SELECT concat(round(SUM(data_length / 1024 / 1024), 2), 'MB') AS data
FROM tables
WHERE table_schema = 'home'
    AND table_name = 'members';
```
 

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

# 需要修改 /etc/mysql/my.cnf的配置 /etc/mysql/mysql.conf.d
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

-- 删除表
DROP TABLE  tbl_name;
-- 或者是
DROP TABLE IF EXISTS tbl_name;

-- 删除表记录
DELETE [LOW_PRIORITY] [QUICK] [IGNORE] FROM tbl_name 

[WHERE where_definition] 

[ORDER BY ...] 

[LIMIT row_count] 

delete from friends;
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

## Maven 

[maven in five minutes](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html)

### maven clean package

[执行逻辑](https://www.cnblogs.com/frankyou/p/6062179.html)  

1、使用清理插件：maven-clean-plugin:2.5执行清理删除已有target目录（版本2.5）；

2、使用资源插件：maven-resources-plugin:2.6执行资源文件的处理（版本2.6）；

3、使用编译插件：maven-compiler-plugin:3.1编译所有源文件生成class文件至target\classes目录下（版本3.1）；

4、使用资源插件：maven-resources-plugin:2.6执行测试资源文件的处理（版本2.6）；

5、使用编译插件：maven-compiler-plugin:3.1编译测试目录下的所有源代码（版本3.1）；

6、使用插件：maven-surefire-plugin:2.12运行测试用例（版本2.12）；

7、使用插件：maven-jar-plugin:2.4对编译后生成的文件进行打包，包名称默认为：artifactId-version，比如本例生成的jar文件：rtp-front-1.0-SNAPSHOT，包文件保存在target目录下（这个生成的包不能在命令行中直接执行，因为我们还没有入口类配置到Manifest资源配置文件中去，后续会阐述）。

备注：

不管是compile、package还是install等前三个步骤都是必不可少的。


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

## Git命令使用
可配合GUI工具和命令行工具参考。

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


## Windows命令

### outlook证书弹出提示

- 查看证书；
- 选择安装证书；
- 选择自定义的存储区；
- 选择"受信任的根证书颁发机构"

（因为自定义的证书通常没有其他证书的签名，所以需要接受为“根证书”）

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
