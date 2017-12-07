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

## Grails - Groovy - Gradle

### grails
[officail doc](http://docs.grails.org/snapshot/guide/single.html)</br>
[documentation](http://grails.org/documentation.html)</br>
[grails 2.5.6 doc](https://grails.github.io/grails2-doc/2.5.6/guide/single.html)

```
# 启动、指定端口、编码
grails -Dfile.encoding=UTF-8 -Dserver.port=8090 run-app

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



## Mysql

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

## Java IDE

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

## Maven 本地使用

- 下载apache-maven-3.5.0-bin.tar.gz到本地
- 解压：`tar -xvf apache-maven-3.5.0-bin.tar.gz`
- 更新配置文件：

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

### 16.04root密码

1. 默认root密码是随机的，即每次开机都有一个新的root密码
2. 在终端输命令 sudo passwd，然后输入当前用户的密码
3. 输入新的密码并确认，此时的密码就是root新密码
4. 修改成功后，输入命令 su root，再输入新的密码

```
#查看当前登录用户，需要root权限 w
root@localhost:~# w
 10:48:38 up 17 days, 19:06,  1 user,  load average: 0.00, 0.02, 0.05
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    10.32.22.66      10:10    6.00s  0.15s  0.00s w

# 踢掉在线用户
pkill -kill -t pts/0

```

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

### 查看系统版本 cat /etc/issue

```
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
### 0xc. 显示n到m行的文件内容
```
sed -n '100, 150p' path/to/filename
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

## Git命令使用
可配合GUI工具和命令行工具参考。

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

```
# 对于ssh的配置
#Default Git
Host github.com
 HostName github.com
 AddKeysToAgent yes
 User gebitang
 UseKeychain yes
 IdentityFile ~/.ssh/id_rsa_github

Host git.coding.net
 User gebitang
 PreferredAuthentications publickey
 IdentityFile ~/.ssh/id_rsa_coding

Host office
 HostName IP address
 AddKeysToAgent yes
 User userName
 UseKeychain yes
 IdentityFile ~/.ssh/id_rsa
```

- ssh配置

生成ssh-key
```
ssh-keygen -t rsa -C "email@address.com"
```
>添加ssh key（id_rsa_xxx、id_rsa_xxx.pub）到对应的repository，如github、gitlab，即可保证可以从对应的设备上就可以访问对应的repository。</br>

- git配置

>git config 配置完全局的 --global user.name 和 --global  user.email之后，可以在对应的仓库下，配置当前仓库使用的user.name 和user.email

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


## Windows命令

### 杀死进程
```
#命令行杀死进程
#查看进程 
tasklist
   
#杀死进程 查看帮助 taskkill /?
taskkill /PID 11112
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

## Wercker Status

[![wercker status](https://app.wercker.com/status/e0d626037da0e4c7f1bee4dcdda350e5/m/master "wercker status")](https://app.wercker.com/project/byKey/e0d626037da0e4c7f1bee4dcdda350e5)

{{< fluid_imgs
  "pure-u-1-1|https://cdn.80000hours.org/wp-content/uploads/2016/05/80K_incomewellnessC_V6.png|High income improves evaluation of life but not emotional well-being|https://80000hours.org/career-guide/job-satisfaction/#money-makes-you-happier-but-only-a-little"
>}}





[^1]: the footnote text.
[^2]: the footnote text 2.
