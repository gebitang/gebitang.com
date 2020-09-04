+++
title = "SonarQube"
description = "Everything about Sonar"
tags = [
]
date = "2020-04-23"
topics = [
    "sonar",
    "tech"
]
toc = true
+++

### SonarQube Build Breaker

有需求希望在Sonar的静态代码扫描之后，直接判断是否终止打包流程。

按照现在的架构——

>好像有点麻烦。 因为 sonar分析完毕后将结果传递给sonarqube，SonarQube有个分析器分析结果（CE版本一次只能处理一个）。
>得到结果是异步的。

还是需要调研一下：

在5.2版本之前的SonarQube上是有这个功能的，作为Jenkins的插件使用，但后来SonarQube自身架构调整，分析器本身是不知道最终结果的，Server端聚合数据。

实际上，官方已经建议[不应该打破打包流程](https://blog.sonarsource.com/why-you-shouldnt-use-build-breaker/)，后来的实现(6.2版本开始)是[提供jenkins的webhook](https://blog.sonarsource.com/breaking-the-sonarqube-analysis-with-jenkins-pipelines/)。目前最新版本也是提供webhook方式。


但直接在pipeline中实现break的需求依然存在，于是[Breaking your build on SonarQube quality gate failure](http://qaware.blogspot.com/2020/02/breaking-your-build-on-sonarqube.html)出现。github对应代码[sonarqube-build-breaker](https://github.com/qaware/sonarqube-build-breaker)

插件实现逻辑——轮询SonarQube，查找结果，根据返回的结果判断是否应该终止pipeline（报错）。同时依赖SonarQube上的Quality Gate的设置。

```java
public void breakBuildIfNeeded(ProjectKey projectKey, BranchMode branchMode) throws IOException, SonarQubeException, InterruptedException, BreakBuildException {
        LOGGER.info("Fetching analysis tasks ...");
        AnalysisTasks analysisTasks = sonarQubeConnector.fetchAnalysisTasks(projectKey, branchMode);
        while (!analysisTasks.getQueue().isEmpty()) {
            LOGGER.info("Analysis task still running, checking again in {} second(s) ...", waitTime.toMillis() / 1000);
            Thread.sleep(waitTime.toMillis());
            analysisTasks = sonarQubeConnector.fetchAnalysisTasks(projectKey, branchMode);
        }

        if (analysisTasks.getLastFinished() == null) {
            LOGGER.error("Analysis queue is empty and there is no finished task. Make sure that you run SonarQube analysis before the build breaker!");
            throw new BreakBuildException("Analysis queue is empty and there is no finished task. Make sure that you run SonarQube analysis before the build breaker!");
        }

        if (analysisTasks.getLastFinished().getStatus() != AnalysisTasks.Status.SUCCESS) {
            LOGGER.error("Last analysis task failed, breaking build!");
            throw new BreakBuildException("Last analysis task failed, breaking build!");
        }

        LOGGER.info("Last analysis was successful, fetching quality gate status ...");
        QualityGateStatus qualityGateStatus = sonarQubeConnector.fetchQualityGateStatus(projectKey, branchMode);

        if (qualityGateStatus == QualityGateStatus.ERROR) {
            LOGGER.error("Quality gate failed, breaking build!");
            throw new BreakBuildException("Quality gate failed, breaking build!");
        }

        LOGGER.info("Great success, everything looks alright!");
    }
```

这看起来我们目前的做法更优雅乜。使用webhook，项目的质量阀采用自定义规则。


### SonarQube的测试覆盖和测试执行

[sonarqube coverage/](https://docs.sonarqube.org/latest/analysis/coverage/)

8.4版本包括两个概念：

- Test Coverage 测试覆盖 ——测试结果有多少覆盖率
- Test Execution 测试执行 ——具体包含多少个测试用例

低版本上，如7.6似乎没有“测试执行”的说法，从7.7版本有对应的文档：[https://docs.sonarqube.org/`7.7`/analysis/coverage/](https://docs.sonarqube.org/7.7/analysis/coverage/)后续的7.8, 7.9直到最新的8.4版本。

`mvn test`默认绑定的goal是`surefire:test`，用来执行单元测试——run tests using a suitable unit testing framework. These tests should not require the code be packaged or deployed.

所以默认的 `mvn test`只会生成`Test Execution`的结果；

添加了jacoco插件并与test阶段绑定后——
`mvn test`会同时产生（surefire插件生成）单测结果，和（jacoco插件生成）单测覆盖率结果。

此时再执行`sonar:sonar`后，单测结果就展示为 “Test Execution 测试执行”；单测覆盖率展示为“Test Coverage 测试覆盖”



完成的生命周期可以[参考官方说明lifecycle reference](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#lifecycle-reference)

最常用的打包`jar`阶段对应的plugin和goal如下表——

|Phase|plugin:goal|
|---------|---------|
|process-resources|resources:resources|
|compile|compiler:compile|
|process-test-resources|resources:testResources|
|test-compile|compiler:testCompile|
|test|surefire:test|
|package|jar:jar|
|install|install:install|
|deploy|deploy:deploy|

对应的具体plugin信息为[具体plugin信息](https://maven.apache.org/ref/3.6.3/maven-core/default-bindings.html) ——
```
<phases>
  <process-resources>
    org.apache.maven.plugins:maven-resources-plugin:2.6:resources
  </process-resources>
  <compile>
    org.apache.maven.plugins:maven-compiler-plugin:3.1:compile
  </compile>
  <process-test-resources>
    org.apache.maven.plugins:maven-resources-plugin:2.6:testResources
  </process-test-resources>
  <test-compile>
    org.apache.maven.plugins:maven-compiler-plugin:3.1:testCompile
  </test-compile>
  <test>
    org.apache.maven.plugins:maven-surefire-plugin:2.12.4:test
  </test>
  <package>
    org.apache.maven.plugins:maven-jar-plugin:2.4:jar
  </package>
  <install>
    org.apache.maven.plugins:maven-install-plugin:2.4:install
  </install>
  <deploy>
    org.apache.maven.plugins:maven-deploy-plugin:2.7:deploy
  </deploy>
</phases>
```

从上述信息也可以看出来plugin的命名规范和使用中的快捷方式。


### 使用sonar-maven-plugin插件执行sonar扫描

理论上，只需要配置`sonar-maven-plugin`插件就可以执行sonar扫描过程，需要有SonarQube服务可供访问。

需要如果不从命令行传递参数，可以在pom文件中指定对应的参数，在`properties`字段提供`sonar.host.url`和`sonar.login`字段的值，登录可使用token方式。示例如下——


```
<properties>
	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
	<java.version>1.8</java.version>

	<sonar.host.url>http://127.0.0.1:9000</sonar.host.url>
	<sonar.login>eef68547ea46d48aaefc472170ec87b833db182f</sonar.login>

</properties>

<plugin>
	<groupId>org.sonarsource.scanner.maven</groupId>
	<artifactId>sonar-maven-plugin</artifactId>
	<version>3.7.0.1746</version>
</plugin>
```

[SonarScanner for Maven](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-maven/)最新版本已经是8.4版本的SonarQube和3.7.0.1746版本的maven插件。

执行`mvn sonar:sonar`默认使用最新版本的sonar插件，可以使用`mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar`指定配置的插件版本

~~实测，即使没有配置jacoco插件，插件可以自动分析单测的测试结果(mvn test?)~~，可以看到覆盖率。目前我还清楚依赖的最小环境是怎样的？

不包含Jacoco插件的场景——

- 1. `mvn clean`清理环境
- 2.  执行`mvn test` 确保完成编译并执行了测试`maven-surefire-plugin:2.22.2:test` 

```
#target 目录下的文件目录
classes
generated-sources
generated-test-sources
maven-status
surefire-reports
test-classes
```

- 3. 执行 `sonar:sonar`结果将不包含覆盖率信息


包含Jacoco插件的场景(Jacoco插件于`test`阶段绑定)——

```xml
<plugin>
	<groupId>org.jacoco</groupId>
	<artifactId>jacoco-maven-plugin</artifactId>
	<version>0.8.5</version>
	<executions>
		<execution>
			<goals>
				<goal>prepare-agent</goal>
			</goals>
		</execution>
		<!-- attached to Maven test phase -->
		<execution>
			<id>report</id>
			<phase>test</phase>
			<goals>
				<goal>report</goal>
			</goals>
		</execution>
	</executions>
</plugin>
```

执行上述相同的三个步骤，则包含了覆盖率。

第二步结束时可以看到target文件夹下多出`site` 文件夹和 `jacoco.exec`文件。

```
#target 目录下的文件目录
classes
generated-sources
generated-test-sources
maven-status
site
surefire-reports
test-classes
jacoco.exec
```

**结论：**

SonarQube上展示的覆盖率依赖Jacoco插件的配合。


附录：

- 使用 `mvn help:effective-pom`可以看到当前项目完整的pom相关配置
- 使用`mvn dependency:treee`可以看到当前项目的依赖关系
- `pluginManagement`用来配置管理所有的plugin信息，用来提供给其他集成当前项目的项目使用，当前项目使用的plugin依然需要配置到`plugins`节点中。[官方解释Plugin_Management](http://maven.apache.org/pom.html#Plugin_Management)，对应的[提问](https://stackoverflow.com/questions/26736876/difference-between-plugins-and-pluginmanagement-tag-in-maven-pom-xml)
- `dependencyManagement`于此类似。[dependency-management](http://maven.apache.org/pom.html#dependency-management)




### 搭建8.3版本SonarQube服务 on CentOS7

[install-java-on-centos](https://phoenixnap.com/kb/install-java-on-centos)

系统环境：

`Linux version 3.10.0-693.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC) ) #1 SMP Tue Aug 22 21:09:27 UTC 2017`

#### 安装Open JDK 11

`sudo yum install -y java-11-openjdk-devel`

```
# install Java
>sudo yum install -y java-11-openjdk-devel
Installed:
  java-11-openjdk-devel.x86_64 1:11.0.7.10-4.el7_8

Dependency Installed:
  copy-jdk-configs.noarch 0:3.3-10.el7_5      giflib.x86_64 0:4.1.6-9.el7        java-11-openjdk.x86_64 1:11.0.7.10-4.el7_8   java-11-openjdk-headless.x86_64 1:11.0.7.10-4.el7_8
  javapackages-tools.noarch 0:3.4.1-11.el7    libXtst.x86_64 0:1.2.3-1.el7       lksctp-tools.x86_64 0:1.0.17-2.el7           pcsc-lite-libs.x86_64 0:1.8.8-8.el7
  python-javapackages.noarch 0:3.4.1-11.el7   python-lxml.x86_64 0:3.2.1-4.el7   ttmkfdir.x86_64 0:3.0.9-42.el7               tzdata-java.noarch 0:2020a-1.el7
  xorg-x11-fonts-Type1.noarch 0:7.5-9.el7

Complete!

```
不需要进行默认配置，机器已安装Java8环境。Sonar服务在 `conf/wrapper.conf`中指定Java11的地址即可 `wrapper.java.command=/usr/lib/jvm/java-11-openjdk/bin/java`


#### 安装PostgreSQL

[官方手册](https://www.postgresql.org/download/linux/redhat/) 根据环境选择对应的配置，会自动提示对应的脚本——

```
# Install the repository RPM:
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# Install PostgreSQL:
sudo yum install -y postgresql10-server

# Optionally initialize the database and enable automatic start:
/usr/pgsql-10/bin/postgresql-10-setup initdb
systemctl enable postgresql-10
systemctl start postgresql-10
```

执行到 初始化时出现权限问题——(切换到root用户执行初始化以及systemctl命令)

```
 /usr/pgsql-10/bin/postgresql-10-setup initdb
Initializing database ... mkdir: cannot create directory ‘/var/lib/pgsql’: Permission denied
failed, see /var/lib/pgsql/10/initdb.log

#log 显示
runuser: may not be used by non-root users
```

剩下的部分就都一样了——

安装完成后，默认会增加一个postgres用户，需要对该用户设置密码，并利用这个用户创建新的数据库用户提供给sonar使用——

```
#为postgres用户设置密码
passwd postgres
# 切换到postgres用户
su - postgres
# 新建postgres数据库用户 sonar
createuser sonar
# 登录数据库
psql
# 执行数据库操作：设置用户密码
ALTER USER sonar WITH ENCRYPTED password 'xxx';
# 创建数据库sonar
CREATE DATABASE sonar WITH ENCODING 'UTF8' OWNER sonar TEMPLATE=template0;
# 退出数据库连接
\q
```


使用正确的用户名、密码却一直无法登陆。[因为——]((https://confluence.atlassian.com/bitbucketserverkb/fatal-ident-authentication-failed-for-user-unable-to-connect-to-postgresql-779171564.html))

>By default PostgreSQL uses IDENT-based authentication and this will never allow you to login via -U and -W options.

命令行链接psql之后，执行`show hba_file ;`显示权限控制文件的位置。

- 按照下面的方式对文件做对应的修改
- 重启数据库`sudo systemctl restart postgresql-10.service`

```
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            md5
host    replication     all             ::1/128                 md5
```

#### PostgreSQL远程登录

[Connecting to a Remote PostgreSQL Database](https://www.netiq.com/documentation/identity-manager-47/setup_windows/data/connecting-to-a-remote-postgresql-database.html)

默认，PostgreSQL只监听本地连接，不允许远程通过TCP/IP链接，运行远程连接需要在Server端修改两个配置文件；` postgresql.conf `和` pg_hba.conf `文件。

- ` postgresql.conf `中修改监听配置，修改为 `listen_addresses = '*'` 
- ` pg_hba.conf `修改权限配置，`host all all 0.0.0.0/0 md5`
- 修改完成后，需要重启数据库才能生效。 `sudo systemctl restart postgresql-10.service`

这两个配置文件的位置可以在数据库中使用postgres用户通过以下sql进行查询：

- `show config_file; `显示 ` postgresql.conf `文件的具体位置
- `show hba_file; `显示 ` pg_hba.conf  `文件的具体位置

```
postgres=# show config_file;
              config_file
----------------------------------------
 /var/lib/pgsql/10/data/postgresql.conf
(1 row)

postgres=# show hba_file;
              hba_file
------------------------------------
 /var/lib/pgsql/10/data/pg_hba.conf
(1 row)

postgres=#

```


#### 遇到的问题

- ES端口被占有 -->  修改sonar.conf的`sonar.search.port`的值，更换一个端口
- bootstrap checks failed。

```
ERROR: [2] bootstrap checks failed
[1]: max file descriptors [65500] for elasticsearch process is too low, increase to at least [65535]
[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

```

第二个问题可以直接执行命令`sudo sysctl -w vm.max_map_count=262144`生效。

第一个问题`max file descriptor`的限制稍微麻烦点。 按照网上的介绍，修改了 `/etc/security/limits.conf` 增大用户限制，并不生效。

- [increasing-file-descriptors-and-open-files-limit-in-centos-7](https://www.99ideas.in/blog-post/increasing-file-descriptors-and-open-files-limit-in-centos-7/)
- 社区也遇到了同样的问题：[Error max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535] installing SonarQube 8.1 on CentOS](https://community.sonarsource.com/t/error-max-file-descriptors-4096-for-elasticsearch-process-is-too-low-increase-to-at-least-65535-installing-sonarqube-8-1-on-centos/20029)

使用`ulimit -a`查看时，`open files `的值依然是65000，执行 `sudo ulimit -n 65536`会提示`sudo ulimit command not found`

[解决这个问题](https://stackoverflow.com/a/17483998/1087122) 执行 `sudo sh -c "ulimit -n 65535 && exec su $LOGNAME"` 

`ulimit` is a shell builtin like `cd`, not a separate program. `sudo` looks for a **binary** to run, but there is no ulimit binary, which is why you get the error message. You need to run it in a shell.

However, while you do need to be root to raise the limit to 65535, you probably don’t want to run your program as root. So after you raise the limit you should switch back to the current user.

To do this, run:

`sudo sh -c "ulimit -n 65535 && exec su $LOGNAME"`

and you will get a **new shell**, without root privileges, but with the raised limit. The exec causes the new shell to replace the process with sudo privileges, so after you exit that shell, you won’t accidentally end up as root again.

如果上面的操作还是不生效， 实测下列操作可生效—— 这里有针对 [ubuntu的环境的做法](https://superuser.com/questions/1200539/cannot-increase-open-file-limit-past-4096-ubuntu)

[centos/redhat: change open files ulimit without reboot?](https://serverfault.com/a/389121)
```shell
$ ulimit -n  
4096
$ ulimit -n 8192
bash: ulimit: open files: cannot modify limit: Operation not permitted
$ sudo bash                    
# ulimit -n                    
4096
# ulimit -n 8192                  
# su - normaluser                 
$ ulimit -n                       
8192

```

上面的设置依然没有生效，需要组合使用——

- 编辑`sudo vi /etc/security/limits.conf`，将对应的设置修改为需要的值65536

```
 * soft nproc 65536
 * hard nproc 65536
 * soft nofile 65536
 * hard nofile 65536
```

- 执行`sudo sysctl -p`使得设置生效
- 切换到root`sudo bash`
- 再切换回当前用户`su - username`

```
[vip@test ~]$ sudo vi /etc/security/limits.conf
[vip@test ~]$ sudo sysctl -p
net.core.rmem_max = 25165824
net.core.wmem_max = 25165824
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 87380 16777216
net.core.netdev_max_backlog = 2500
net.ipv4.ip_local_port_range = 12000 65535
net.ipv4.conf.all.promote_secondaries = 1
vm.min_free_kbytes = 524288
vm.swappiness = 1
net.ipv4.tcp_tw_reuse = 1
[vip@test ~]$
[vip@test ~]$
[vip@test ~]$ ulimit -n
65500
[vip@test ~]$ sudo bash
[root@test /home/vip]# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 126546
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 65536
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 65536
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
[root@test /home/vip]# su - vip
Last login: Thu Jul 23 10:49:22 CST 2020 on pts/0
[vip@test ~]$ ulimit -n
65536
[vip@test ~]$
```


#### 注意事项

[官方要求](https://docs.sonarqube.org/latest/requirements/requirements/#header-5)

If you're running on Linux, you must ensure that:

- `vm.max_map_count` is greater than or equal to `524288`. check with `sysctl vm.max_map_count`
- `fs.file-max` is greater than or equal to `131072`. check with `sysctl fs.file-max`
- the user running SonarQube can open at least `131072` file descriptors. check with `ulimit -u`
- the user running SonarQube can open at least `8192` threads. check with `ulimit -n`


To set these values more permanently, you must update either `/etc/sysctl.d/99-sonarqube.conf` (or `/etc/sysctl.conf` as you wish) to reflect these values.

If the user running SonarQube (sonarqube in this example) does not have the permission to have at least 131072 open descriptors, you must insert this line in `/etc/security/limits.d/99-sonarqube.conf` (or `/etc/security/limits.conf` as you wish):

```
sonarqube   -   nofile   131072
sonarqube   -   nproc    8192
```

If you are using systemd to start SonarQube, you must specify those limits inside your unit file in the section [service] :

```
[Service]
...
LimitNOFILE=131072
LimitNPROC=8192
...
```


### 搭建8.3版本SonarQube服务 on Ubuntu

现有服务使用Java8版本，需要新安装Java 11并保留Java8

#### 安装Open JDK 11

` sudo apt-get install -y openjdk-11-jdk`

安装成功后使用`update-java-alternatives -l`查看已经安装版本——

```
java-1.11.0-openjdk-amd64      1111       /usr/lib/jvm/java-1.11.0-openjdk-amd64
java-1.8.0-openjdk-amd64       1081       /usr/lib/jvm/java-1.8.0-openjdk-amd64
```

本地设置的环境变量会覆盖默认设置，例如 `/etc/profile`和 `~/.zshrc`文件中的`JAVA_HOME`值。直接使用 `update-java-alternatives -s java-1.11.0-openjdk-amd64`时甚至会报错——"update-alternatives error no alternatives for mozilla-javaplugin.so"

使用` sudo update-alternatives --config java`进行手动选择来设置

```
➜  ~ java -version
openjdk version "1.8.0_252"
OpenJDK Runtime Environment (build 1.8.0_252-8u252-b09-1~18.04-b09)
OpenJDK 64-Bit Server VM (build 25.252-b09, mixed mode)
➜  ~
➜  ~
➜  ~ sudo update-alternatives --config java
There are 2 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                            Priority   Status
------------------------------------------------------------
  0            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      auto mode
  1            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      manual mode
* 2            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      manual mode

Press <enter> to keep the current choice[*], or type selection number: 1
update-alternatives: using /usr/lib/jvm/java-11-openjdk-amd64/bin/java to provide /usr/bin/java (java) in manual mode
➜  ~
➜  ~ java -version
openjdk version "11.0.7" 2020-04-14
OpenJDK Runtime Environment (build 11.0.7+10-post-Ubuntu-2ubuntu218.04)
OpenJDK 64-Bit Server VM (build 11.0.7+10-post-Ubuntu-2ubuntu218.04, mixed mode, sharing)
➜  ~

```

**注：** 如果不想修改系统默认使用的Java版本，可以在安装SonarQube之后，在` $SONARQUBE-HOME/conf/wrapper.conf `配置选项里指定所用的JVM路径，例如——
```
#当前环境下
wrapper.java.command=/usr/lib/jvm/java-11-openjdk-amd64/bin/java
```

#### 安装PostgreSQL

```
# 安装
$ apt -y install postgresql postgresql-contrib
#查看版本
$ /usr/lib/postgresql/10/bin/postgres -V
postgres (PostgreSQL) 10.12 (Ubuntu 10.12-0ubuntu0.18.04.1)

```

安装完成后，默认会增加一个postgres用户，需要对该用户设置密码，并利用这个用户创建新的数据库用户提供给sonar使用——

```
#为postgres用户设置密码
passwd postgres
# 切换到postgres用户
su - postgres
# 新建postgres数据库用户 sonar
createuser sonar
# 登录数据库
psql
# 执行数据库操作：设置用户密码
ALTER USER sonar WITH ENCRYPTED password 'sonar';
# 创建数据库sonar
CREATE DATABASE sonar WITH ENCODING 'UTF8' OWNER sonar TEMPLATE=template0;
# 退出数据库连接
\q
```

#### 安装8.3.1版本的SonarQube

[官方教程](https://docs.sonarqube.org/latest/setup/install-server/)

下载 [SonarQube downloads](https://www.sonarqube.org/downloads/)，解压后将bin目录添加到环境变量。

注意事项：  
- 不用解压在以数字开头的目录里
- 不要使用root用户运行sonar，可以创建一个用户只用来启动sonar服务

```
postgres:x:125:129:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
sonarqube:x:1001:1001::/home/sonarqube:/bin/sh
```

配置`/conf/sonar.properties `：  
```
# Example for PostgreSQL
sonar.jdbc.username=sonarqube
sonar.jdbc.password=mypassword
sonar.jdbc.url=jdbc:postgresql://localhost/sonardb

# 配置Elasticsearch 数据地址，默认在  $SONARQUBE-HOME/data 
sonar.path.data=/var/sonarqube/data
sonar.path.temp=/var/sonarqube/temp

# 配置web端信息
sonar.web.host=192.0.0.1
sonar.web.port=80
# 起始路径
sonar.web.context=/sonarqube

# 指定Java路径
wrapper.java.command=/usr/lib/jvm/java-11-openjdk-amd64/bin/java
```

最后在bin目录下启动sonar：`./sonar.sh start` 默认的用户名密码为`admin:admin`

最好将SonarQube放在当前用户(非root)用户有权限的目录下。

另外，可以根据[这个说明](https://docs.sonarqube.org/latest/setup/operate-server/)将Sonarqube设置为service、配置到Nginx后面等操作——暂未实操验证。目前这个环境仅作手动验证。

#### 可能遇到问题

- 启动失败，提示`ERROR: [1] bootstrap checks failed [1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]`

[issue fix：](https://github.com/docker-library/elasticsearch/issues/111#issuecomment-268511769) `sudo sysctl -w vm.max_map_count=262144`

- 提示` Failed to create table schema_migrations`，类似下面的内容

```
2020.07.01 16:21:53 ERROR web[][o.s.s.p.PlatformImpl] Web server startup failed
java.lang.IllegalStateException: Failed to create table schema_migrations

```

数据库连接指定了`?currentSchema=my_schema`，删除即可，使用默认的public schema。


### SonarQube 升级

现在的版本为7.6，使用中遇到类似`Objects.nonnull`方法无法支撑的问题， [Not validating Objects.nonNull](https://community.sonarsource.com/t/not-validating-objects-nonnull/12942)，官方建议升级到至少7.9版本。

升级的问题：
- 依赖Java 11
- 不再支撑MYSQL
- 数据库迁移必须确保在相同版本下的SonarQube进行 [https://github.com/SonarSource/mysql-migrator](https://github.com/SonarSource/mysql-migrator)


这意味着：先做低版本下的数据库迁移；再做升级动作。

[end-of-life-of-mysql-support](https://community.sonarsource.com/t/end-of-life-of-mysql-support/8667)

配合postgresql设置sonarqube
[https://dunterov.github.io/sq-psql/](https://dunterov.github.io/sq-psql/)


PostgreSQL

下载：
[https://www.enterprisedb.com/download-postgresql-binaries](https://www.enterprisedb.com/download-postgresql-binaries)

[https://www.enterprisedb.com/downloads/postgres-postgresql-downloads](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads)

简易教程：  
[https://www.runoob.com/postgresql/postgresql-create-database.html](https://www.runoob.com/postgresql/postgresql-create-database.html)



### 单module和多module下的Jacoco插件最小配置以及执行

[[Coverage & Test Data] Importing JaCoCo coverage report in XML format](https://community.sonarsource.com/t/coverage-test-data-importing-jacoco-coverage-report-in-xml-format/12151)

[Multi-module Apache Maven example](https://github.com/SonarSource/sonar-scanning-examples/tree/master/sonarqube-scanner-maven/maven-multimodule)

实践检验，亲测有效:) 

插播一下**Phase VS. goal**

Phase(阶段)包含了有一组goals(目标动作)。phase执行有依赖关系。比如执行 `mvn package`， 包含了`validate`, `compile`, `test`, and `package`动作。

goal就是maven用来管理项目的具体的task，可以属于某一个phase，也可以独立存在。比如`mvn dependency:tree`就不属于任何phase。

参考：[maven lifecycle-reference](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#lifecycle-reference)


goal在两种情况下触发：   

- 自动执行（如果plugin的goal默认绑定到了对应的phase，或手动定义了goal在哪个phase执行）
- 通过命令明确执行 `mvn <plugin name>:<goal>`

参考：[Maven plugin execution ID](https://stackoverflow.com/a/33279426/1087122)



插播结束。


#### 单module

比较简单，添加类似下面的配置即可。下面的配置将report重新绑定到了test阶段而已。

`report`默认绑定的阶段是在`verify`阶段。Binds by default to the [lifecycle phase](http://maven.apache.org/ref/current/maven-core/lifecycles.html): `verify`(run any checks to verify the package is valid and meets quality criteria.)

绑定的意思是在对应的phase下才会执行这一goal。

```
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.5</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
执行`mvn clean jacoco:prepare-agent install jacoco:report`执行成功则可以获取到单测文件`target/site/jacoco/jacoco.xml`  
（依次执行了多个动作，既包括maven自带的动作，也包括插件jacoco定义的goal）

#### 多module 

- 首先要在项目的pom文件下添加一个用来聚合的module，例如`utm`
- 项目pom文件依然需要添加jacoco插件（执行时需要在项目的根目录下，没有jacoco插件则无法执行对应的插件动作）依然使用 `report`动作
- 聚合module下需要：
  - a) 聚合所有的依赖模块，参考[JaCoCo-Multi-Module](../../jiansh/us/jacoco-multi-module/)； 
  - b) 定义jacoco的聚合动作，使用`report-aggregate`动作

```
<build>
  <plugins>
      <plugin>
        <groupId>org.jacoco</groupId>
        <artifactId>jacoco-maven-plugin</artifactId>
        <version>0.8.5</version>
        <executions>
          <execution>
            <id>self-define-report-aggregate</id>
            <phase>verify</phase>
            <goals>
                <goal>report-aggregate</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
  </plugins>
</build>
```

依然在项目根目录下执行`mvn clean jacoco:prepare-agent install jacoco:report` 执行成功则可以获取到聚合后的单测文件`utm/target/site/jacoco-aggregate/jacoco.xml`  

插件顺序：  

- 子module如果定义，使用子module的定义
- 子module没有定义的情况下，使用父module的定义，依次类推到根module
- 同一个module下的重复定义会被告警，但以后定义的作为最终生效的定义

### Jacoco Maven Plugin goals

[Jacoco Maven Plugin](https://www.jacoco.org/jacoco/trunk/doc/maven.html)

对应的github项目工程：[github jacoco-maven-plugin](https://github.com/jacoco/jacoco/tree/master/jacoco-maven-plugin)


The JaCoCo Maven plug-in provides the JaCoCo runtime agent to your tests and allows basic report creation.

与surefire插件配合使用时需要注意，surefire插件的`forkCount`参数不能为0。 

>When using the <tt>maven-surefire-plugin</tt> or <tt>maven-failsafe-plugin</tt> you **must not** use a [<tt>forkCount</tt>](http://maven.apache.org/surefire/maven-surefire-plugin/test-mojo.html#forkCount) of <tt>0</tt> or set the [<tt>forkMode</tt>](http://maven.apache.org/surefire/maven-surefire-plugin/test-mojo.html#forkMode) to <tt>never</tt> as this would prevent the execution of the tests with the <tt>javaagent</tt> set and no coverage would be recorded.

**forkCount**

>Option to specify the number of VMs to fork in parallel in order to execute the tests. When terminated with "C", the number part is multiplied with the number of CPU cores. Floating point value are only accepted together with "C". If set to "0", no VM is forked and all tests are executed within the main process.

---

一共定义了11个goals：可以通过配置了`jacoco-maven-plugin`的项目下执行 `mvn help:describe -Dplugin=org.jacoco:jacoco-maven-plugin -Ddetail`获取到详细的说明。或者参考下面的链接说明——

The JaCoCo Maven plug-in defines the following goals:

*   [help](https://www.jacoco.org/jacoco/trunk/doc/help-mojo.html)
*   [prepare-agent](https://www.jacoco.org/jacoco/trunk/doc/prepare-agent-mojo.html)
*   [prepare-agent-integration](https://www.jacoco.org/jacoco/trunk/doc/prepare-agent-integration-mojo.html)
*   [merge](https://www.jacoco.org/jacoco/trunk/doc/merge-mojo.html)
*   [report](https://www.jacoco.org/jacoco/trunk/doc/report-mojo.html)
*   [report-integration](https://www.jacoco.org/jacoco/trunk/doc/report-integration-mojo.html)
*   [report-aggregate](https://www.jacoco.org/jacoco/trunk/doc/report-aggregate-mojo.html)
*   [check](https://www.jacoco.org/jacoco/trunk/doc/check-mojo.html)
*   [dump](https://www.jacoco.org/jacoco/trunk/doc/dump-mojo.html)
*   [instrument](https://www.jacoco.org/jacoco/trunk/doc/instrument-mojo.html)
*   [restore-instrumented-classes](https://www.jacoco.org/jacoco/trunk/doc/restore-instrumented-classes-mojo.html)



### 多模块项目包含war类型依赖

[JaCoCo-Multi-Module](../../jiansh/us/jacoco-multi-module/)强调了如果有子模块为pom类型时，需要将type进行明确说明。

但如果子模块为war类型时，如果不明示，会按照默认的jar类型去查找——不符合实际情况。

这个[问题](https://stackoverflow.com/questions/56071617/getting-coverage-from-war-dependency-in-multi-module-project)描述的是这一现象；

这个[官方jacoco issue #991](https://github.com/jacoco/jacoco/issues/991)提示的解决方案也是增加`<type>war</type>`。

增加了`<type>war</type>`之后，在执行`spring-boot:repackage`任务时，提示“找不到主类”——

>Execution default of goal org.springframework.boot:spring-boot-maven-plugin:1.5.4.RELEASE:repackage failed: Unable to find main class

**解决方案**：用来聚合单侧结果的模块打包方式制定为`<packaging>pom</packaging>`。[有效的打包类型包括](https://maven.apache.org/pom.html#packaging) `pom`, `jar`, `maven-plugin`, `ejb`, `war`, `ear`, `rar`

这样在执行`mvn clean install`时聚合模块不做打包处理。pom类型意味着只有`install`和`deploy`两个[阶段动作](https://maven.apache.org/ref/3.6.3/maven-core/default-bindings.html#Plugin_bindings_for_pom_packaging)

### Abstract Syntax Tree

[官方文档之CUSTOM_RULES_101](https://github.com/SonarSource/sonar-java/blob/master/docs/CUSTOM_RULES_101.md)之[Using syntax trees and API basics](https://github.com/SonarSource/sonar-java/blob/master/docs/CUSTOM_RULES_101.md#first-version-using-syntax-trees-and-api-basics)提到——

>Prior to running any rule, the SonarQube Java Analyzer parses a given Java code file and produces an equivalent data structure: the **Syntax Tree**. 

项目依赖的[JDT](https://github.com/SonarSource/sonar-java/blob/master/jdt/pom.xml)模块就来自Eclipse项目[eclipse.jdt.core](https://github.com/eclipse/eclipse.jdt.core)

```
# JDT module in sonar-java
<dependency>
        <groupId>org.eclipse.jdt</groupId>
        <artifactId>org.eclipse.jdt.core</artifactId>
        <version>3.20.0</version>
        ...
</dependency>
```

AST(Abstract Syntax Tree)在Eclipse的[入门文章](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/) 

![](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/images/workflow.png)

- 1. Java源码：可以是源码文件，也可以是一组字符串；
- 2. 使用 `org.eclipse.jdt.core.dom.ASTParser`类解析Java源码
- 3. 得到抽象语法树AST
- 4. 操作AST
- 5. 保存修改后的AST到源码

---

1.  *Java source*: To start off, you provide some source code to parse. This source code can be supplied as a Java file in your project or directly as a `char[]` that contains Java source
2.  *Parse*: The source code described at [1](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#workflow-legend-1) is parsed. All you need for this step is provided by the class `org.eclipse.jdt.core.dom.ASTParser`. See [the section called “Parsing source code”](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#sec-parsing-a-source-file "Parsing source code").
3.  The *Abstract Syntax Tree* is the result of step [2](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#workflow-legend-2). It is a tree model that entirely represents the source you provided in step [1](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#workflow-legend-1). If requested, the parser also computes and includes additional symbol resolved information called "[bindings](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#sec-bindings "Bindings")".
4.  *Manipulating the AST*: If the AST of point [3](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#workflow-legend-3) needs to be changed, this can be done in two ways:

    1.  By directly modifying the AST.
    2.  By noting the modifications in a separate protocol. This protocol is handled by an instance of `ASTRewrite`.

    See more in [the section called “How to Apply Changes”](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#sec-how-to-apply-changes "How to Apply Changes").
5.  *Writing changes back*: If changes have been made, they need to be applied to the source code that was provided by [1](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#workflow-legend-1). This is described in detail in [the section called “Write it down”](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#sec-write-it-down "Write it down").
6.  *`IDocument`*: Is a wrapper for the source code of step [1](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#workflow-legend-1) and is needed at point [5](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/#workflow-legend-5)

AST的背景知识包括了`Java Model`，最新的内容参考[Java Model](https://help.eclipse.org/2020-03/topic/org.eclipse.jdt.doc.isv/guide/jdt_int_model.htm)

![](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/images/java-model-overview.png)

![](https://help.eclipse.org/2020-03/topic/org.eclipse.jdt.doc.isv/guide/images/openables.png)

![](https://help.eclipse.org/2020-03/topic/org.eclipse.jdt.doc.isv/guide/images/sourceelements.png)


关于JDT最新的[官网内容](https://help.eclipse.org/2020-03/topic/org.eclipse.jdt.doc.isv/guide/jdt_int_core.htm)，包括了

- [Java model](https://help.eclipse.org/2020-03/topic/org.eclipse.jdt.doc.isv/guide/jdt_int_model.htm)
- [Manipulating Java code](https://help.eclipse.org/2020-03/topic/org.eclipse.jdt.doc.isv/guide/jdt_api_manip.htm)
- [Eclipse JDT - Abstract Syntax Tree (AST) and the Java Model by Vogella](https://www.vogella.com/tutorials/EclipseJDT/article.html) 

[Creating CompilationUnit (AST) from file on disk](https://www.vogella.com/tutorials/EclipseJDT/article.html#creating-compilationunit-ast-from-file-on-disk)
```
private static void createFromDist() throws Exception {
    ASTParser parser = ASTParser.newParser(AST.JLS8);
    parser.setResolveBindings(true);
    parser.setStatementsRecovery(true);
    parser.setBindingsRecovery(true);
    parser.setKind(ASTParser.K_COMPILATION_UNIT);
    File resource = new File("./tests/resources/Snippet.java");
    Path sourcePath = Paths.get(resource.toURI());
    String sourceString = new String(Files.readAllBytes(sourcePath));
    char[] source = sourceString.toCharArray();
    parser.setSource(source);
    parser.setUnitName(sourcePath.toAbsolutePath().toString());
    CompilationUnit astRoot = (CompilationUnit) parser.createAST(null);
}
```

配合Eclipse版本的[ASTView](https://www.eclipse.org/jdt/ui/astview/index.php)，效果类似——

![](https://www.eclipse.org/articles/Article-JavaCodeManipulation_AST/images/md-astview.png)

Idea上也有对应的[插件](https://plugins.jetbrains.com/plugin/9345-jdt-astview)，展示效果当然不如Eclipse官方的酷炫，也足够理解对应的概念。


中文资料参考——  

- [Java代码分析器(一): JDT入门](https://segmentfault.com/a/1190000000609246)
- [Java代码分析器(二): 使用DOM API操作抽象语法树](https://segmentfault.com/a/1190000000638838)


### SonarQube添加自定义规则

[官方文档之CUSTOM_RULES_101](https://github.com/SonarSource/sonar-java/blob/master/docs/CUSTOM_RULES_101.md)

[官方示例工程](https://github.com/SonarSource/sonar-custom-rules-examples/tree/master/java-custom-rules)

按照官方文档在官方示例工程上进行操作，可以编译通过，生成最终的jar包文件。

在`extentions/plugins`添加上自定义jar包后，启动时总是提示类似下面的错误——

```
jvm 1    | 2020.05.06 18:58:43 WARN  app[][o.e.t.n.Netty4Transport] exception caught on transport layer [[id: 0xa148553e, L:/127.0.0.1:3333 - R:/127.0.0.1:9001]], closing connection
jvm 1    | java.io.IOException: 远程主机强迫关闭了一个现有的连接。
jvm 1    |      at sun.nio.ch.SocketDispatcher.read0(Native Method)
jvm 1    |      at sun.nio.ch.SocketDispatcher.read(Unknown Source)
jvm 1    |      at sun.nio.ch.IOUtil.readIntoNativeBuffer(Unknown Source)
jvm 1    |      at sun.nio.ch.IOUtil.read(Unknown Source)
jvm 1    |      at sun.nio.ch.SocketChannelImpl.read(Unknown Source)
jvm 1    |      at io.netty.buffer.UnpooledUnsafeDirectByteBuf.setBytes(UnpooledUnsafeDirectByteBuf.java:433)
jvm 1    |      at io.netty.buffer.AbstractByteBuf.writeBytes(AbstractByteBuf.java:1100)
jvm 1    |      at io.netty.channel.socket.nio.NioSocketChannel.doReadBytes(NioSocketChannel.java:372)
jvm 1    |      at io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:123)
jvm 1    |      at io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:644)
jvm 1    |      at io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized
```

第一次定位发现log中有 `java.lang.IllegalStateException: Name of rule [repository=mycompany-java, key=MyFirstCustomRule] is empty`的提示，意识到Rule的定义写错了位置，注解写到Test文件上去了。重新打包后，这个错误提示不再出现。

第二次，依然有网络连接关闭的提示，但删除自定义的jar包之后就可以正常运行。

本地使用的是sonarqube7.6版本，猜测有可能是不兼容导致的。但这个错误提示不知道是怎样的逻辑出现的。

---

SonarQube提供了独立的log服务，命令行启动时输出的是sonar.log的内容。上述`Netty4Transport exception caught on transport layer`远程连接关闭的问题log属于es.log(elastic search)的部分。

而由于自定义规则的plugin导致的问题会在ce.log(Compute Engine)里有具体的提示（但这个内容居然不在命令行输出，导致一直对上面的远程连接失败摸不着头脑）。基本会有类似——

>org.sonar.updatecenter.common.exception.IncompatiblePluginVersionException: The plugin 'java' is in version 5.10.1.16922 whereas the plugin 'javacustom' requires a least a version 6.3.0.21585.

猜测没错，属于兼容性问题。关键的两个版本值——

- `sonar.version`的值为plugin支持的SonarQube最低版本（目前8.0以上对应的LTS版本为7.9；目前看7.6根7.7还是兼容的）。最新的[兼容矩阵](https://docs.sonarqube.org/latest/instance-administration/plugin-version-matrix/)在7.9以下版本里没有对应的说明。
- `sonarjava.version`对应plugin支持的版本，需要跟使用的SonarQube中安装的版本一致，管理员登录后，可以在`admin/marketplace?filter=installed`位置查看`SonarJava`的版本。

根据pom.xml的git log对其他的依赖进行对应的修改。

```
# current 
<sonar.version>8.2.0.32929</sonar.version>
<sonarjava.version>6.3.0.21585</sonarjava.version>

# modify
<sonar.version>7.7</sonar.version>
<sonarjava.version>5.10.1.16922</sonarjava.version>
```

回滚到上述版本后，目前的测试方法都失效了。——这是因为`org.sonar.java.checks.verifier.JavaCheckVerifier`的 `public static CheckVerifier newVerifier()`方法在2020/3/17加入的。SONARJAVA-3315 Unify CheckVerifiers accross the plugin (#2871)`e7f1493fa8572fa88ed528be37fc8debabc5e671`

此时本地再执行单元测试时，会报错——

>The rule 'f.newTuple20(Object, Object)' has already been defined somewhere in the grammar.

这两个地方都讨论了这个问题：   
- [项目issue #38](https://github.com/SonarSource/sonar-custom-rules-examples/issues/38)  
- [官方社区 create-java-custom-rule](https://community.sonarsource.com/t/create-java-custom-rule/10320/5)

~~本地测试无法正常运行（环境改的比较多了）~~，但不影响打包操作。`mvn clean install`可以正常生成jar包。

放到对应的位置，重启SonarQube，可以看到这个自定义的plugin——

![](https://upload-images.jianshu.io/upload_images/3296949-6118488b181b12ff.png)

![](https://upload-images.jianshu.io/upload_images/3296949-de25702c52563d0f.png)

---

6.3版本与5.10版本最大的不同是对源码解析时使用的Parser类发生变化：

- 5.10版本使用独立的自研项目[sslr](https://github.com/SonarSource/sslr)(SonarSource Language Recognizer)
- 6.3版本使用Eclipse的[eclipse.jdt.core](https://github.com/eclipse/eclipse.jdt.core)项目


[官方示例工程](https://github.com/SonarSource/sonar-custom-rules-examples/tree/master/java-custom-rules)中的测试代码还会用到sslr项目，只在`MyJavaRulesDefinitionTest`这个测试类中使用

```xml
    <dependency>
			<groupId>org.sonarsource.sslr</groupId>
			<artifactId>sslr-testing-harness</artifactId>
			<version>${sslr.version}</version>
			<scope>test</scope>
		</dependency>
```

>~~本地测试无法正常运行（环境改的比较多了）~~

还是pom环境问题，对比了[5.10.1.16922](https://github.com/SonarSource/sonar-java/archive/5.10.1.16922.tar.gz)版本的Sonar-Java之后，已解决。

- 将ssrl升级到`1.23`版本
- 保留guava的版本为`26.0-jre`(降低到`19.0`时，会出现解析报错问题`java.lang.NoSuchMethodError: com.google.common.base.Preconditions.checkNotNull(Ljava/lang/Object;Ljava/lang/String;Ljava/lang/Object;)Ljava/lang/Object;`)


#### 扫描规则源码分析

- 依赖sonar-java项目最新版本(6.3)的示例工程可以进行运行单元测试。
- 目前使用的sonarqube对应的sonar-java的版本(5.10)
- 修改后兼容当前使用的sonarqube版本的示例工程分支无法进行单侧

只能人肉分析代码，学习规则是怎样运行起来的。

测试驱动——

- 一个unit测试类提供入库；
- 一个测试源码作为测试数据
- 一个规则类（最终自定义并且打包生效的就是这个文件，不同的规则写不同的规则类即可）

测试暴露的接口就是：测试源码+自定义的Check类。其他`注册`内容示例工程已经打好架子，暂时忽略。

调用入口如下： 
```java
// 5.x (在6.x版本上这个方法deprecated,等价于下面的6.x的方法)
JavaCheckVerifier.verify("src/test/files/MyFirstCustomCheck.java",
                             new MyFirstCustomCheck());
// 6.x
JavaCheckVerifier.newVerifier()
                .onFile("src/test/files/MyFirstCustomCheck.java")
                .withCheck(new MyFirstCustomCheck())
                .verifyIssues();
 
```

##### 5.10版本

过程式编程，一堆静态方法构造出来需要调用的组件。

进入如下方法——

- check就是自定义的规则类；
- verifier用来检查最后的结果。例如，如果出现issue时，如何描述。

其他类关系——

- MyFirstCustomCheck   
  - extends `IssuableSubscriptionVisitor`  
      - extends `SubscriptionVisitor`  
          - implements `JavaFileScanner`  Common interface for all checks analyzing a java file.
             - extends `JavaCheck` (empty interface)

- `ExpectedIssueCollector`
  - extends `SubscriptionVisitor`
     - implements `JavaFileScanner`

- `JavaCheckVerifier` extends `CheckVerifier`

```java
// line 261
private static void scanFile(String filename, JavaFileScanner check, JavaCheckVerifier javaCheckVerifier, Collection<File> classpath, boolean withSemantic) {
    //又添加了一个JavaScanner
    JavaFileScanner expectedIssueCollector = new ExpectedIssueCollector(javaCheckVerifier);
    VisitorsBridgeForTests visitorsBridge;
    File file = new File(filename);
    // 构造上下文关系context，所以放到了检查结果的CheckVerfiler中做静态方法
    SonarComponents sonarComponents = CheckVerifier.sonarComponents(file);
    if (withSemantic) {
        //进入这里初始化Bridge，
      visitorsBridge = new VisitorsBridgeForTests(Lists.newArrayList(check, expectedIssueCollector), Lists.newArrayList(classpath), sonarComponents);
    } else {
      visitorsBridge = new VisitorsBridgeForTests(Lists.newArrayList(check, expectedIssueCollector), sonarComponents);
    }
    //终于进入干活的地方，
    JavaAstScanner.scanSingleFileForTests(file, visitorsBridge, javaCheckVerifier.javaVersion);
    VisitorsBridgeForTests.TestJavaFileScannerContext testJavaFileScannerContext = visitorsBridge.lastCreatedTestContext();
    if (testJavaFileScannerContext == null) {
      Fail.fail("Semantic was required but it was not possible to create it. Please checks the logs to find out the reason.");
    }
    javaCheckVerifier.checkIssues(testJavaFileScannerContext.getIssues(), javaCheckVerifier.providedJavaVersion);
  }
```

新new一个scanner进行scan动作，对所有的文件进行扫描。其中的Parser用来将文件字符串内容读取为语法树结构AST？（`JavaParser`这块内容有点晕）
```java
  @VisibleForTesting
  public static void scanSingleFileForTests(File file, VisitorsBridge visitorsBridge, JavaVersion javaVersion) {
    if (!file.isFile()) {
      throw new IllegalArgumentException("File '" + file + "' not found.");
    }
    // 创建的Parser，用来将文件内容
    JavaAstScanner astScanner = new JavaAstScanner(JavaParser.createParser(), null);
    visitorsBridge.setJavaVersion(javaVersion);
    astScanner.setVisitorBridge(visitorsBridge);
    astScanner.scan(Collections.singleton(file));
  }
```

核心方法 
```java
private void simpleScan(File file) {
    //...
    try {
      String fileContent = getFileContent(file);
      Tree ast;
      if(fileContent.isEmpty()) {
        ast = parser.parse(file);
      } else {
        ast = parser.parse(fileContent);
      }
      visitor.visitFile(ast);
    } 
  }
```

```java
/**
 * Common interface for all checks analyzing a java file.
 */
@Beta
public interface JavaFileScanner extends JavaCheck {

  /**
   * Method called after parsing and semantic analysis has been done on file.
   * @param context Context of analysis containing the parsed tree.
   */
  void scanFile(JavaFileScannerContext context);

}
```

---

##### 6.3版本

6.3版本的`newVerifier()`对应的就是`InternalCheckVerifier`，builder模式，最终进入方法`verifyAll()`

- InternalCheckVerifier 
   - implements CheckVerifier

CheckVerifier从5.x的抽象类变为6.x的纯接口类

```java
private void verifyAll() {
    List<JavaFileScanner> visitors = new ArrayList<>(checks);
    if (withoutSemantic && expectations.expectNoIssues()) {
      visitors.add(expectations.noEffectParser());
    } else {
      visitors.add(expectations.parser());
    }
    SonarComponents sonarComponents = sonarComponents();
    VisitorsBridgeForTests visitorsBridge;
    if (withoutSemantic) {
      visitorsBridge = new VisitorsBridgeForTests(visitors, sonarComponents);
    } else {
      List<File> actualClasspath = classpath == null ? DEFAULT_CLASSPATH : classpath;
      visitorsBridge = new VisitorsBridgeForTests(visitors, actualClasspath, sonarComponents);
    }

    JavaAstScanner astScanner = new JavaAstScanner(sonarComponents);
    visitorsBridge.setJavaVersion(javaVersion == null ? DEFAULT_JAVA_VERSION : javaVersion);
    astScanner.setVisitorBridge(visitorsBridge);
    astScanner.scan(files);

    VisitorsBridgeForTests.TestJavaFileScannerContext testJavaFileScannerContext = visitorsBridge.lastCreatedTestContext();
    checkIssues(testJavaFileScannerContext.getIssues());
  }
```
核心方法也没有大的变化 

```java
private void simpleScan(InputFile inputFile) {
    ...
    try {
      String fileContent = inputFile.contents();
      final String version;
      if (visitor.getJavaVersion() == null || visitor.getJavaVersion().asInt() < 0) {
        version = /* default */ JParser.MAXIMUM_SUPPORTED_JAVA_VERSION;
      } else {
        version = Integer.toString(visitor.getJavaVersion().asInt());
      }
      //使用jdt的解析服务
      Tree ast = JParser.parse(
        version,
        inputFile.filename(),
        fileContent,
        visitor.getClasspath()
      );
      visitor.visitFile(ast);
    }
    ...
  }
```

核心流程都是将文件解析成`Tree` 

5.x的版本使用了 SonarSource自己的`sslr`工程——

- [解析表达式语法(Parsing Expression Grammars)](https://www.cnblogs.com/freeblues/p/8436523.html)  
- [An introduction to Parsing Expression Grammars with LPeg](https://leafo.net/guides/parsing-expression-grammars.html) 
- [wiki: Parsing Expression Grammars](https://en.wikipedia.org/wiki/Parsing_expression_grammar)
- [SonarSource Language Recognizer](https://github.com/SonarSource/sslr)   

>If you want to start working with SSLR, you must be familiar with the following standard concepts : `Lexical Analysis`, `Parsing Expression Grammar` and `AST`(Abstract Syntax Tree). 

```java
//5.x JavaParser.createParser().parse()
public static ActionParser<Tree> createParser() {
    return new JavaParser(JavaLexer.createGrammarBuilder(),
      JavaGrammar.class,
      new TreeFactory(),
      new JavaNodeBuilder(),
      JavaLexer.COMPILATION_UNIT);
}

// parser.parse()
private N parse(Input input) {
    //this.parseRunner = new ParseRunner(b.build().getRootRule());
    ParsingResult result = parseRunner.parse(input.input());
    ...
    N node = syntaxTreeCreator.create(result.getParseTreeRoot(), input);
    if (node instanceof AstNode) {
      astNodeSanitizer.sanitize((AstNode) node);
    }
    return node;
  }
```


6.x版本则使用了Eclipse的[`jdt`项目](https://git.eclipse.org/c/jdt/eclipse.jdt.core.git/) 

```java
public static CompilationUnitTree parse(
    String version,
    String unitName,
    String source,
    List<File> classpath
  ) {
    //   
    ASTParser astParser = ASTParser.newParser(AST.JLS13);
    ...
    CompilationUnit astNode;
    try {
      astNode = (CompilationUnit) astParser.createAST(null);
    } catch (Exception e) {
      LOG.error("ECJ: Unable to parse file", e);
      throw new RecognitionException(-1, "ECJ: Unable to parse file.", e);
    }
    ...
    // 做格式状态？
    JParser converter = new JParser();
    converter.sema = new JSema(astNode.getAST());
    converter.compilationUnit = astNode;
    converter.tokenManager = new TokenManager(lex(version, unitName, sourceChars), source, new DefaultCodeFormatterOptions(new HashMap<>()));

    JavaTree.CompilationUnitTreeImpl tree = converter.convertCompilationUnit(astNode);
    tree.sema = converter.sema;

    ASTUtils.mayTolerateMissingType(astNode.getAST());

    setParents(tree);
    return tree;
  }
```

- 5.x版本使用sslr将源码直接解析后识别为sonar-java识别的Tree对象；
- 6.x版本借用jdt将源码解析为AST，再转化为sonar-java识别的Tree对象；

- 最后大家都调用`visitor.visitFile(Tree parsedTree);`来进行具体的判断

#### visitFile规则源码分析

VisitorsBridge的构造：

##### 6.3版本

构造函数里初始化这样几个组件： 

- Scanners，可以理解为规则集合
- classPath，略
- sonarComponents：上下文关系（作用是什么？）
- SymbolicExecutionMode：包含三种模式：`DISABLED`,`ENABLED`, `ENABLED_WITHOUT_X_FILE`(影响的范围是？) ;

```java
public VisitorsBridge(Iterable<? extends JavaCheck> visitors, List<File> projectClasspath,
                        @Nullable SonarComponents sonarComponents, SymbolicExecutionMode symbolicExecutionMode,
                        @Nullable JavaFileScanner analysisIssueFilter) {
    this.allScanners = new ArrayList<>();
    for (Object visitor : visitors) {
      if (visitor instanceof JavaFileScanner) {
        allScanners.add((JavaFileScanner) visitor);
      }
    }
    this.analysisIssueFilter = analysisIssueFilter;
    this.classpath = projectClasspath;
    this.executableScanners = allScanners.stream().filter(IS_ISSUABLE_SUBSCRIPTION_VISITOR.negate()).collect(Collectors.toList());
    this.issuableSubscriptionVisitorsRunner = new IssuableSubsciptionVisitorsRunner(allScanners);
    this.sonarComponents = sonarComponents;
    this.classLoader = ClassLoaderBuilder.create(projectClasspath);
    this.symbolicExecutionEnabled = symbolicExecutionMode.isEnabled();
    this.behaviorCache = new BehaviorCache(classLoader, symbolicExecutionMode.isCrossFileEnabled());
  }
```

根据scanner过滤出来几类`runner`，在最终的`visitor.visitFile(Tree parsedTree);`里分别进行调用，每个runner会传递一个上下文对象`DefaultJavaFileScannerContext`

不同的scanner实现了不同的接口，runner中执行这个接口的实现，`JavaFileScanner`接口的`scanFile`方法

对于自定义的`MyFirstCustomCheck`checker来说，最终执行的是`issuableSubscriptionVisitorsRunner.run(javaFileScannerContext);`即——

```java
private void visit(Tree tree) throws CheckFailureException {
    Kind kind = tree.kind();
    List<SubscriptionVisitor> subscribed = checks.getOrDefault(kind, Collections.emptyList());
    Consumer<SubscriptionVisitor> callback;
    boolean isToken = (kind == Tree.Kind.TOKEN);
    if (isToken) {
    callback = s -> s.visitToken((SyntaxToken) tree);
    } else {
    callback = s -> s.visitNode(tree);
    }
    forEach(subscribed, callback);
    if (isToken) {
    forEach(checks.getOrDefault(Tree.Kind.TRIVIA, Collections.emptyList()), s -> ((SyntaxToken) tree).trivias().forEach(s::visitTrivia));
    } else {
    visitChildren(tree);
    }
    if(!isToken) {
    forEach(subscribed, s -> s.leaveNode(tree));
    }
}

// 
private final void forEach(Collection<SubscriptionVisitor> visitors, Consumer<SubscriptionVisitor> callback) throws CheckFailureException {
    for (SubscriptionVisitor visitor : visitors) {
    runScanner(() -> callback.accept(visitor), visitor);
    }
}
```

##### 5.10版本

构造函数——

```java
public VisitorsBridge(Iterable visitors, List<File> projectClasspath, @Nullable SonarComponents sonarComponents, SymbolicExecutionMode symbolicExecutionMode) {
    this.allScanners = new ArrayList<>();
    for (Object visitor : visitors) {
        if (visitor instanceof JavaFileScanner) {
        allScanners.add((JavaFileScanner) visitor);
        }
    }
    // private static Predicate<JavaFileScanner> isIssuableSubscriptionVisitor = s -> s instanceof IssuableSubscriptionVisitor;
    this.executableScanners = allScanners.stream().filter(isIssuableSubscriptionVisitor.negate()).collect(Collectors.toList());
    this.scannerRunner = new ScannerRunner(allScanners);
    this.sonarComponents = sonarComponents;
    this.classLoader = ClassLoaderBuilder.create(projectClasspath);
    this.symbolicExecutionEnabled = symbolicExecutionMode.isEnabled();
    this.behaviorCache = new BehaviorCache(classLoader, symbolicExecutionMode.isCrossFileEnabled());
}
```

实际的调用方法`visitor.visitFile(Tree parsedTree);`

```java
public void visitFile(@Nullable Tree parsedTree) {
    //...
    JavaFileScannerContext javaFileScannerContext = createScannerContext(tree, semanticModel, sonarComponents, fileParsed);
    // Symbolic execution checks
    if (symbolicExecutionEnabled && isNotJavaLangOrSerializable(PackageUtils.packageName(tree.packageDeclaration(), "/"))) {
      runScanner(javaFileScannerContext, new SymbolicExecutionVisitor(executableScanners, behaviorCache), AnalysisError.Kind.SE_ERROR);
      behaviorCache.cleanup();
    }
    executableScanners.forEach(scanner -> runScanner(javaFileScannerContext, scanner, AnalysisError.Kind.CHECK_ERROR));
    //
    scannerRunner.run(javaFileScannerContext);
    if (semanticModel != null) {
      classesNotFound.addAll(semanticModel.classesNotFound());
    }
  }
```

同6.3版本相似的逻辑。

#### 使用其他版本

官方版本提供了不同的[release版本](https://github.com/SonarSource/sonar-java/tags?after=5.13.1.18282)，可以下载对应的[5.10.1.16922](https://github.com/SonarSource/sonar-java/archive/5.10.1.16922.tar.gz)版本。

SonarQube强调TDD开发模式，测试代码很齐全，针对单一源码的测试代码可以直接在IDE中运行，例如`org.sonar.java.ast.JavaAstScannerTest`下的`comments`方法。

```java
@Test
  public void comments() {
    File file = new File("src/test/files/metrics/Comments.java");
    DefaultInputFile resource = new TestInputFileBuilder("", "src/test/files/metrics/Comments.java").build();
    fs.add(resource);
    NoSonarFilter noSonarFilter = mock(NoSonarFilter.class);
    JavaAstScanner.scanSingleFileForTests(file, new VisitorsBridge(new Measurer(fs, context, noSonarFilter)));
    verify(noSonarFilter).noSonarInFile(resource, ImmutableSet.of(15));
  }
```

事实上，对于测试包`org.sonar.java.checks`下的测试类都可以直接执行单元测试。

这样就可以针对使用的版本直接开发自定义规则并进行验证了

#### 规则开发

[Java custom rule writing without exploring the Syntax Tree](https://community.sonarsource.com/t/java-custom-rule-writing-without-exploring-the-syntax-tree/550/2)

>When implementing a Java custom rule, nothing forces you to use a `BaseTreeVisitor` or `IssuableSubscriptionVisitor`, you can perfectly only implement the `JavaFileScanner` interface, which will give you access to the content of the file:

```java
package org.sonar.samples.java.checks;

import org.sonar.check.Rule;
import org.sonar.plugins.java.api.JavaFileScanner;
import org.sonar.plugins.java.api.JavaFileScannerContext;

@Rule(key = "MyCheck")
public class MyCheck implements JavaFileScanner {

  @Override
  public void scanFile(JavaFileScannerContext context) {
    context.getFileContent(); // to retrieve the full content of the file as a String
    context.getFileLines();   // to retrieve the content of each lines of the file, as a String
  }
}
```

`DefaultJavaFileScannerContext`传递出去后，上报issue时，

```java
  context.reportIssue(this, idf, String.format("Avoid using annotation @%s", name));

  @Override
  public void reportIssue(JavaCheck javaCheck, Tree tree, String message) {
    //这个方法如何调用到了——VisitorsBridgeForTests的对应方法？
    reportIssue(javaCheck, tree, message, ImmutableList.of(), null);
  }
```

- 编译自定义规则，将产出物放到`\extensions\plugins`目录下
- 本地启动SonarQube（生成登录的token）
- 准备好测试工程（包含源码+编译后的class文件内容）
- 执行SonarScanner。`path\to\sonar-scanner.bat  -Dsonar.host.url=http://127.0.0.1:9000  -Dsonar.login=generated_token_value -Dsonar.projectKey=local-loop-prj  -Dsonar.sourceEncoding=UTF-8 -Dsonar.sources=src -Dsonar.java.binaries=target/classes`
- 在SonarQube上查看执行结果 

** sonar-scanner执行的最小参数：url、login、projectKey、sources、binaries。

写了一个**最基础**版本的自定义规则：`不允许在循环中对容器进行赋值`。忽略算法实现- -||

```java
package org.sonar.samples.java.checks;

import org.sonar.check.Priority;
import org.sonar.check.Rule;
import org.sonar.plugins.java.api.JavaFileScanner;
import org.sonar.plugins.java.api.JavaFileScannerContext;
import org.sonar.plugins.java.api.semantic.Type;
import org.sonar.plugins.java.api.tree.BaseTreeVisitor;
import org.sonar.plugins.java.api.tree.BlockTree;
import org.sonar.plugins.java.api.tree.ExpressionStatementTree;
import org.sonar.plugins.java.api.tree.ForStatementTree;
import org.sonar.plugins.java.api.tree.MethodTree;
import org.sonar.plugins.java.api.tree.StatementTree;
import org.sonar.plugins.java.api.tree.Tree;
import org.sonar.plugins.java.api.tree.VariableTree;
import org.sonar.samples.java.RulesList;

import java.util.List;

@Rule(
        key = "ForLoopAssignCheck",
        name = "assign container value in for loop",
        description = "don't try to assign or update container value in any loop operation.",
        priority = Priority.MAJOR,
        tags = {"custom"})
public class ForLoopAssignCheck extends BaseTreeVisitor implements JavaFileScanner {


    private JavaFileScannerContext context;


    @Override
    public void scanFile(JavaFileScannerContext context) {
        this.context = context;
        scan(context.getTree());
    }

    @Override
    public void visitMethod(MethodTree tree) {
        BlockTree block = tree.block();
        if(block != null) {
            boolean containerFound = false;
            String containerVar = "";
            List<StatementTree> statements = block.body();
            for(StatementTree s : statements) {
                if(s.is(Tree.Kind.VARIABLE)) {
                    VariableTree vt = ((VariableTree)s);
                    Type type = vt.type().symbolType();
                    if(isSubTypeOfSetOrMap(type)) {
                        containerFound = true;
                        containerVar = vt.simpleName().name();
                    }
                }

                if(s.is(Tree.Kind.FOR_STATEMENT)) {
                    ForStatementTree ft = (ForStatementTree)s;
                    if(ft.statement().is(Tree.Kind.BLOCK)) {
                        List<StatementTree> body = ((BlockTree)ft.statement()).body();

                        for(StatementTree state : body) {
                            if(state.is(Tree.Kind.EXPRESSION_STATEMENT)) {
                                ExpressionStatementTree et = (ExpressionStatementTree)state;

                                String var = et.expression().firstToken().text();
                                if(var.length() > 0 && var.equals(containerVar)) {
                                    context.reportIssue(this, et, String.format("avoid assigning container %s in loop.", var));
                                }
                            }
                        }
                    }
                }
            }

        }
        super.visitMethod(tree);
    }

    private boolean isSubTypeOfSetOrMap(Type type) {
        return type.isSubtypeOf("java.util.Set") || type.isSubtypeOf("java.util.Map");
    }
}

```

![](https://upload-images.jianshu.io/upload_images/3296949-9bc1bd3092103c77.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 定制规则

[How to deactivate a rule in SonarQube?](https://sqa.stackexchange.com/questions/24734/how-to-deactivate-a-rule-in-sonarqube)


Follow below steps to disable any rule in SonarQube:

1.  Login by admin

2.  Go to quality profile & Select java/php profile [whichever is appropriate to you]

3.  Enter the rule as key and Search

4.  Uncheck the box which will inactive the rule

5.  Run Sonar runner command once again to verify the modifications are working properly

I have borrowed my answer from [here](http://javadeveloperchoiceno1.blogspot.com/2014/12/how-to-disable-rules-in-sonar.html)

---

Profile（质量配置）--> 选择语言 --> copy（复制出新的profile）--> 点击新的profile进行规则设置 （可通过搜索rule）

- SonarQube不允许修改默认的profile配置
- 需要先复制一份出来
- 在复制的profile里修改rule的规则（激活、未激活）
- 然后将复制的profile设置为默认选项

--- 

### SonarQube设置排除目录

Administration（配置）--> Configuration（配置）--> Analysis Scope (排除）

可对Code Coverage（代码覆盖率）、File（源码文件）、Duplication Excusions（重复）分别进行单独的设置

---

### 覆盖率统计不准确

本地使用ide执行的测试覆盖率**明显高于**线上使用jacoco动态统计的结果。

本地模拟线上环境执行也得出了相同的结果。

> 对应的测试用例的确执行了，但没有对应的统计。

分析来看，所有使用了 `MockitoJUnitRunner ` (来自[mockito-core/2.8.9](https://mvnrepository.com/artifact/org.mockito/mockito-core/2.8.9)，[github mockito](https://github.com/mockito/mockito))运行junit测试的用例都被忽略了

看起来跟[PowerMock](https://github.com/powermock/powermock/wiki/Code-coverage-with-JaCoCo)是相同的问题？
>We are going to replace Javassist with ByteBuddy (#727) and it should help to resolve this old issue. But right now there is NO WAY TO USE PowerMock with JaCoCo On-the-fly instrumentation. And no workaround to get code coverage in IDE.

>简单来说，大家都是动态搞字节码的，所以我（Jacoco）是没办法知道你（mockito）是怎么搞（bytecode）的。

**PowerMock** is an open source mocking library for the Java world. It extends the existing mocking frameworks, such as **[EasyMock](http://www.easymock.org/ "easymock")** and **[Mockito](https://code.google.com/p/mockito/ "mockito")**, to add even more powerful features to them.


[java-code-coverage-jacoco-and-mockitojunitrunner](https://jacoco.narkive.com/Sh86kg5E/java-code-coverage-jacoco-and-mockitojunitrunner)——
>JaCoCo does not play well with other tools that modify bytecode on-the-fly. JaCoCo needs to see the same classes at runtime than at reporting time (as the warning text says).
>
>A possible (but not really recommended) workaround is to use JaCoCo offline instrumentation.


[mockito issues 969](https://github.com/mockito/mockito/issues/969)问题里认为可以兼容，应该都是只offline模式，直接读取class文件，而不是on-the-fly模式。

[Stack Overflow jacoco-mockito-android-tests-zero-coverage-reported](https://stackoverflow.com/questions/46517471/jacoco-mockito-android-tests-zero-coverage-reported/46614216#46614216)

前两个问题链接看起来是有希望兼容的？第三个告诉你目前还不行哈 :(   
[https://github.com/jacoco/jacoco/issues/193](https://github.com/jacoco/jacoco/issues/193)  
[https://github.com/mockito/mockito/issues/757](https://github.com/mockito/mockito/issues/757)  
[https://github.com/jacoco/jacoco/issues/51](https://github.com/jacoco/jacoco/issues/51)  

###  测试代码被误认为是源码

原因：`sonar.source`参数值未正确设置。

[Coverage info displayed on Test code](https://community.sonarsource.com/t/coverage-info-displayed-on-test-code/30452)

使用sonar-scanner执行扫描动作，Java项目工程指定项目目录作为`sonar.source`参数的值。`sonar.source=.`一直执行的挺好。

最近有项目组反馈“测试目录里*INT的代码”被识别为了源码——导致覆盖率降低。类似在`src/test/java`目录下创建了类似`ServiceXXXINT.java`类型的文件。

看起来是如果测试目录下的文件如果以Test结尾，即使使用之前的设定，也会被识别为测试代码；否则会被认为是源码文件。

官方人员建议：使用 `mvn sonar:sonar`执行，会默认识别出项目结构。
[文档](https://docs.sonarqube.org/latest/analysis/analysis-parameters/)中也做了说明——


`sonar.source` Comma-separated paths to directories containing main source files.  
>Read from build system for Maven, Gradle, MSBuild projects. Defaults to project base directory when neither sonar.sources nor sonar.tests is provided.

将此参数指定到`src/main/java`目录即可解决此问题。 

`mvn sonar:sonar`会自动识别项目结构：[Introduction to the Standard Directory Layout](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)

![](https://upload-images.jianshu.io/upload_images/3296949-265e8fe314e1edf1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)