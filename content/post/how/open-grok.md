+++
title = "部署验证OpenGrok"
description = "How to setup OpenGrok"
tags = [
]
date = "2020-08-16"
topics = [
    "opengrok"
]
toc=true
+++

### 环境搭建和基础使用

[How to setup OpenGrok](https://github.com/oracle/opengrok/wiki/How-to-setup-OpenGrok)

*   [opengrok site](https://oracle.github.io/opengrok/)
*   [Features](https://github.com/oracle/opengrok/wiki/Features)
*   [Comparison with Similar Tools](https://github.com/oracle/opengrok/wiki/Comparison-with-Similar-Tools)
*   [Supported Revision Control Systems](https://github.com/oracle/opengrok/wiki/Supported-Revision-Control-Systems)
*   [Supported Languages and Formats](https://github.com/oracle/opengrok/wiki/Supported-Languages-and-Formats)
*   [How to build OpenGrok from source](https://github.com/oracle/opengrok/wiki/How-to-build-OpenGrok-from-source)
*   [How to install OpenGrok](https://github.com/oracle/opengrok/wiki/How-to-install-OpenGrok)
*   [Internals](https://github.com/oracle/opengrok/wiki/Internals)
*   [Story of OpenGrok](https://github.com/oracle/opengrok/wiki/Story-of-OpenGrok)


在用户目录下执行，仅作验证使用。

#### 安装准备
-  从[https://github.com/oracle/opengrok/releases](https://github.com/oracle/opengrok/releases)下载压缩包，目前最新的为`opengrok-1.3.16.tar.gz`
- 创建目录 `mkdir -p opengrok/{src,data,dist,etc,log}` src目录作为源码仓库，data目录作为indexer目录
- 解压压缩包`tar -C opengrok/dist --strip-components=1 -xzf opengrok-1.3.16.tar.gz` (--strip-components=1 参数Strip NUMBER leading components from file names on extraction。 将移除一层目录)
- 指定log配置`cp opengrok/dist/doc/logging.properties opengrok/etc`
- 在src目录下git clone几个代码仓库

####  ubuntu下安装universal-ctags 

[官方教程](https://github.com/universal-ctags/ctags/blob/master/docs/autotools.rst)

- 安装依赖环境 

```
sudo apt install \
    gcc make \
    pkg-config autoconf automake \
    python3-docutils \
    libseccomp-dev \
    libjansson-dev \
    libyaml-dev \
    libxml2-dev
```

- 拉取代码`git clone https://github.com/universal-ctags/ctags.git` 
- 在ctags目录下，执行`./autogen.sh`， `./configure`， `make`， `sudo make install`

成功后，默认ctags位于 `/usr/local/bin/ctags` 


```
$ ./autogen.sh
$ ./configure --prefix=/where/you/want # defaults to /usr/local
$ make
$ make install # may require extra privileges depending on where to install
```

- 生成配置文件

```
java \
    -Djava.util.logging.config.file=/opengrok/etc/logging.properties \
    -jar opengrok/dist/lib/opengrok.jar \
    -c /usr/local/bin/ctags \
    -s opengrok/src -d opengrok/data -H -P -S -G \
    -W opengrok/etc/configuration.xml 
```

- 复制opengrok/dist/lib/目录下的source.war文件到tomcat服务器的webapps目录

#### 安装管理工具(可选)

python包包括了一下封装后的命令，包括indexer等。在release版本的tar包解压后的`tools`文件夹下包含`opengrok-tools.tar.gz`文件，使用pip命令安装此文件即可。

- 先确保有python3环境
- 安装python3环境下的pip `sudo apt install python3-pip`(查看帮助`pip3 --help`，或者`pip3 install --help`)， 检查版本`pip3 --version` pip 9.0.1 from /usr/lib/python3/dist-packages (python 3.6)
- 到tool目录下执行`pip3 install opengrok-tools.tar.gz`


[OpenGrok tools](https://github.com/oracle/opengrok/tree/master/opengrok-tools)

- 确保有python3环境，最好使用独立的虚拟环境。创建虚拟环境`python3 -m venv opengroktools` （ubuntu环境下需要先安装 `apt-get install python3-venv`）
- 将虚拟环境中的pip升级到最新版本`opengroktools/bin/python -m pip install --upgrage pip` (否则在安装opengrok-tools.tar.gz时可能会报错)
- 安装工具包`opengroktools/bin/python -m pip install /tools/opengrok-tools.tar.gz`

在单位虚拟环境目录下可以使用这些tools，例如`opengroktools/bin/opengrok-indexer`

激活与失效展示：

```
#使用当前虚拟环境
➜  ~ . opengroktools/bin/activate
(opengroktools) ➜  ~ opengrok-indexer --help
usage: opengrok-indexer [-h] [-l LOGLEVEL] [-v] [-j JAVA] [-J JAVA_OPTS]
                        [-e ENVIRONMENT] [--doprint boolean]
                        (-a JAR | -c CLASSPATH) [-C]
                        options [options ...]

OpenGrok indexer wrapper

positional arguments:
  options               options

optional arguments:
  -h, --help            show this help message and exit
  -l LOGLEVEL, --loglevel LOGLEVEL
                        Set log level (e.g. "ERROR")
  -v, --version         Version of the tool
  -j JAVA, --java JAVA  path to java binary
  -J JAVA_OPTS, --java_opts JAVA_OPTS
                        java options. Use one for every java option, e.g.
                        -J=-server -J=-Xmx16g
  -e ENVIRONMENT, --environment ENVIRONMENT
                        Environment variables in the form of name=value
  --doprint boolean     Enable/disable printing of messages from the
                        application as they are produced.
  -a JAR, --jar JAR     Path to jar archive to run
  -c CLASSPATH, --classpath CLASSPATH
                        Class path
  -C, --no_ctags_check  Suppress checking for ctags
# 退出虚拟环境
(opengroktools) ➜  ~ deactivate
➜  ~
# 使用命令
➜  ~ opengrok-index --help
zsh: command not found: opengrok-index
# 使用绝对路径
➜  ~ opengroktools/bin/opengrok-indexer --help
usage: opengrok-indexer [-h] [-l LOGLEVEL] [-v] [-j JAVA] [-J JAVA_OPTS]
                        [-e ENVIRONMENT] [--doprint boolean]
                        (-a JAR | -c CLASSPATH) [-C]
                        options [options ...]
...
```

暴力删除虚拟环境： `rm -r opengroktools`

#### 安装web服务器

[Setup Apache Tomcat 8 | 8.5 on Ubuntu 16.04 | 18.04 LTS](https://websiteforstudents.com/setup-apache-tomcat-8-8-5-on-ubuntu-16-04-18-04-lts/)

- 下载[压缩包](https://tomcat.apache.org/download-80.cgi)，解压压缩包，指定端口号，直接到bin目录执行`./catalina.sh start` 将启动默认的服务器
- 修改source工程（即openGrok服务）的配置文件`tomcat8/webapps/source/WEB-INF/web.xml`，指定服务变量 `CONFIGURATION`的值

```
<context-param>
  <description>Full path to the configuration file where OpenGrok can read its configuration</description>
  <param-name>CONFIGURATION</param-name>
  <param-value>/home/xxx/opengrok/etc/configuration.xml</param-value>
</context-param>
```


#### 搜索方法

[OpenGrok 搜索用法](https://www.jianshu.com/p/d41d1ca66a2f)

- 主页的`Help`按钮包含了基础的使用方法
- 官方API调用说明文档:[opengrop API on apiary](https://opengrok.docs.apiary.io/)(Powerful API Design Stack. Built for Developers)
- openGrok底层使用lucene，支持的正则表达式参考[Lucene regexp page](http://lucene.apache.org/core/8_5_2/core/org/apache/lucene/util/automaton/RegExp.html)

- Full Search	(**full**) Search through all text tokens (words,strings,identifiers,numbers) in index.
- Definition	(**defs**) Only finds symbol definitions (where e.g. a variable (function, ...) is defined).
- Symbol	(**refs**)  Only finds symbols (e.g. methods, classes, functions, variables).
- File Path (**path**)	path of the source file (no need to use dividers, or if, then use "/" - Windows users, "\\" is an escape key in Lucene query syntax! Please don't use "\\", or replace it with "/").    
Also note that if you want just exact path, enclose it in "", e.g. "src/mypath", otherwise dividers will be removed and you get more hits.  
- History	(**hist**) History log comments.

[23 Google Search Tips You'll Want to Learn](https://www.pcmag.com/how-to/23-google-search-tips-youll-want-to-learn)  
[https://www.google.com/advanced_search](https://www.google.com/advanced_search)  
[https://www.google.com/advanced_image_search](https://www.google.com/advanced_image_search)  
搜索操作符支持，类似的google支持的[Search operators](https://support.google.com/websearch/answer/2466433)


### gitlab API 获取project

[Gitlab : List all the projects and all the groups](https://stackoverflow.com/a/45087988/1087122)

分别调用`projects`接口和`groups`接口即可，类似—— 

`curl --head "https://<host/api/v4/projects?private_token=<your private token>&per_page=100&page=<page_number>" `

只返回head部分，其中包括`X-Total`和`X-Total-Pages`的值。默认page从1开始计算，默认每页返回[20条](https://docs.gitlab.com/ce/api/#pagination)，最大返回[100个条目](https://docs.gitlab.com/11.11/ee/api/README.html#pagination)

官方推荐的[gitlab api客户端实现](https://about.gitlab.com/partners/#api-clients)

[How to retrieve repository size through REST API](https://forum.gitlab.com/t/how-to-retrieve-repository-size-through-rest-api/28088/2)

```
"statistics": {
      "commit_count": 5914,
      "storage_size": 1727206,
      "repository_size": 0,
      "wiki_size": 52428,
      "lfs_objects_size": 0,
      "job_artifacts_size": 1674778
    },
```
The size is listed in Bytes. To convert to KB, divide by 1,000. To convert to MB, divide by 1,000,000.

[How can I see the size of a GitHub repository before cloning it?](https://stackoverflow.com/questions/8646517/how-can-i-see-the-size-of-a-github-repository-before-cloning-it)

There's a way to access this information through the [GitHub API](https://docs.github.com/en/rest/reference/repos).

*   Syntax: `GET /repos/:user/:repo`
*   Example: [https://api.github.com/repos/git/git](https://api.github.com/repos/git/git)

When retrieving information about a repository, a property named `size` is valued with the size of the whole repository (including all of its history), in kilobytes.

For instance, the Git repository weights around 124 MB. The `size` property of the returned JSON payload is valued to `124283`.

[403 Access Denied on Tomcat 8 Manager App without prompting for user/password](https://stackoverflow.com/questions/38551166/403-access-denied-on-tomcat-8-manager-app-without-prompting-for-user-password)

[Manager App HOW-TO](http://tomcat.apache.org/tomcat-8.0-doc/manager-howto.html#Configuring_Manager_Application_Access)

### centOS 上可能遇到的问题

- 依赖环境的参数不同

```
# on Centos： diff with Fedora on python3-docutils vs. python3-docutils-doc
sudo yum install -y \
    gcc make \
    pkgconfig autoconf automake \
    python3-docutils-doc \
    libseccomp-devel \
    jansson-devel \
    libyaml-devel \
    libxml2-devel  
```

- centOS7上默认的git版本检查`git --version`为`git version 1.8.3.1`，使用opengrok创建indexer的时候回报错——[官方issue #131：seeing lots of warning messages for unknown date format with older Git](https://github.com/oracle/opengrok/issues/1331)

>17:23:31 WARNING: GitRepository not working (missing binaries?): xxx  
>17:23:31 WARNING: Failed to determineCurrentVersion for xxx: java.io.IOException: fatal: unknown date format iso8601-strict


[Install Latest Git ( Git 2.x ) on CentOS 7](https://computingforgeeks.com/how-to-install-latest-version-of-git-git-2-x-on-centos-7/)

> 1. 卸载旧git `sudo yum remove git*`  
> 2. 添加endpoint源`sudo yum -y install https://packages.endpoint.com/rhel/7/os/x86_64/endpoint-repo-1.7-1.x86_64.rpm`    
> 3. 安装新git `sudo yum install git`  
> 4. 检查版本： `git version 2.24.1`


### opengrok tools使用 

[Per project management and workflow](https://github.com/oracle/opengrok/wiki/Per-project-management-and-workflow)

[OpenGrok项目管理](https://www.jianshu.com/p/63c88743b880) 最后使用 `opengrok-indexer`时认为最后两个参数被忽略了——

```
opengroktools/bin/opengrok-indexer -a opengrok/dist/lib/opengrok.jar -- \
    -c /usr/local/bin/ctags \
    -U 'http://localhost:8080/source' \
    -s /data1/data/groups -d opengrok/data0907 \
    -R fresh_config.xml
    -H 234_sale \
    234_sale
```

事实上，`opengrok-indexer`执行当个项目的index动作时，不需要指定源码、数据目录。这些值在配置文件`-R fresh_config.xml`中已经定义。使用官方的方法即可，效果如下——

```
opengroktools/bin/opengrok-indexer -a opengrok/dist/lib/opengrok.jar -- \
>     -c /usr/local/bin/ctags \
>     -U 'http://localhost:8080/source' \
>     -R fresh_config.xml
    -H sub1 \
    sub1 Sep 11, 2020 3:59:44 PM org.opengrok.indexer.configuration.Configuration read
INFO: Reading configuration from /home/vip/fresh_config.xml
Sep 11, 2020 3:59:45 PM org.opengrok.indexer.index.Indexer parseOptions
...
..
.
INFO: Starting traversal of directory /t24_2650_ai-cv-ocr
Sep 11, 2020 4:20:11 PM org.opengrok.indexer.util.Statistics report
INFO: Done traversal of directory /t24_2650_ai-cv-ocr (took 33.529 seconds)
Sep 11, 2020 4:20:11 PM org.opengrok.indexer.index.IndexDatabase update
INFO: Starting indexing of directory /t24_2650_ai-cv-ocr
Sep 11, 2020 4:20:11 PM org.opengrok.indexer.util.Statistics report
INFO: Done indexing of directory /t24_2650_ai-cv-ocr (took 0 ms)
Sep 11, 2020 4:20:12 PM org.opengrok.indexer.index.IndexDatabase update
INFO: Starting traversal of directory /t34_69_public_service

```

`-H`参数实际是由opengrok.jar包提供的，详见下。这样默认会将log输出到控制台，等待结果中，看起来新添加项目后，还需要对已有项目“巡视”一番？

`opengroktools/bin/opengrok-projadm -b opengrok -d 342_znkf`删除效果——

```
$ opengroktools/bin/opengrok-projadm -b opengrok -d 342_znkf
Deleting project 342_znkf and its index data
Removing source code under /data1/data/groups/342_znkf
Refreshing configuration
```
将删除索引文件、源码文件、刷新配置文件。

使用`curl -s -X GET http://localhost:8080/source/api/v1/configuration -o fresh_config.xml` 可以下载更新后的配置文件。可以看到已不再包含刚删除的项目。

### 新增项目

- 拉取newPrj项目
- 获取最新配置信息
- 添加newPrj项目

以上三步对应以下三条命令

```
opengroktools/bin/opengrok-projadm -b opengrok -a newPrj
curl -s -X GET http://localhost:8080/source/api/v1/configuration -o fresh_config.xml
opengroktools/bin/opengrok-indexer -a opengrok/dist/lib/opengrok.jar -- -c /usr/local/bin/ctags -U 'http://localhost:8080/source' -R fresh_config.xml -H newPrj newPrj 

```

### OpenGrok 认证

将gitlab上的项目作为OpenGrok的输入，直接可以搜索全量代码。需要做基本的认证。

- 申请内部域名，支持域名访问
- 禁止IP+port+path的访问方式：使用nginx作为代理，利用iptables关闭端口的incoming流量
- 先直接在nginx上做基本的认证

这样可以确保本地的同步服务可以继续使用，但外部访问需要使用用户名+密码的方式。

#### 关闭8080端口incoming流量

[CentOS / RHEL : How to block incoming and outgoing ports using iptables](https://www.thegeekdiary.com/centos-rhel-how-to-block-incoming-and-outgoing-ports-using-iptables/)

~~`iptables -A INPUT -p tcp --destination-port 8080 -j DROP`关闭8080端口incoming流量~~
这样把本地请求也给关闭了，使用下面的方式运行本地连接，拒绝外部访问，[参考](https://serverfault.com/a/247180)

```
iptables -A INPUT -p tcp -s localhost --dport 8080 -j ACCEPT
iptables -A INPUT -p tcp --dport 8080 -j DROP

```

[iptables 删除rule](https://www.digitalocean.com/community/tutorials/how-to-list-and-delete-iptables-firewall-rules)

- 按照具体的规则删除：`iptables -D INPUT -p tcp --destination-port 8080 -j DROP`
- 按行删除：`sudo iptables -L --line-numbers`列出行，执行`sudo iptables -D INPUT 1`


#### 配置nginx认证

- 安装httpd-tools以生产密码文件 `yum  -y install httpd-tools` 

`htpasswd -2 -c /usr/local/src/passwd.db username` 为username生成密码，并使用SHA-256进行密码加密。保持在-c的参数文件中。

这个文件每行保持一组用户名密码信息。nginx的[` auth_basic_user_file`](http://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)模块支持[配置多个用户](https://serverfault.com/a/896901)

可以使用`htpasswd -h`查看其它参数

- 配置nginx

```
server {
    listen      80;
    server_name  opengrok.domain.com;
    auth_basic  "not ready, only master can go";   #添加此配置
    auth_basic_user_file  /usr/local/src/passwd.db; #加载生成的密码文件

    if ($host != "opengrok.domain.com") {
      return 404;
    }

    location / {
            proxy_pass http://localhost:8080/;
    }
}
```

- 重启nginx即生效，`nginx -s reload`(需要root用户配置nginx服务，先执行`nginx -t`进行验证test)


下一步再研究接入SSO的方式如何实现。

[Opengrok SSO configuration: issue #3189](https://github.com/oracle/opengrok/issues/3189)

### 使用API搜索

- 配置了nginx的basic认证时，可以通过url传递认证参数调用实际的search请求
- 除了`/metrics`之外，其他的api目录在`/api/v1/`路径下
- 除了`/metrics`,`/suggester`,`/search`之前，其他的api都需要在本地进行调用

search参数如果没有指定，则默认为空值（不做对应的限制）

full参数表示全文本搜索，如果搜索的值包括分隔符，例如`.`，需要将完整的文本使用双引号引起来。例如"test.abc.com"，双引号在url中要做转义，此参数值为 `full=%22abc.test.com%22`

上述参数定义可以在[官方api站点](https://opengrok.docs.apiary.io/#reference/0/search/return-search-results?console=1)进行定义使用

加上服务部署在8080端口的`/source`目录，查找所以`upload.js`文件中包含`abc.test.com`的项目。

最终的查询url为：`http://127.0.0.1:8080/source/api/v1/search?full=%22guazi01.kss.ksyun.com%22&path=%22upload.js%22`

在服务器上使用wget执行查询结果，需要再将url使用双引号引起来，否则`&`符号之后的值将被忽略。执行 `wget "http://127.0.0.1:8080/source/api/v1/search?full=%22guazi01.kss.ksyun.com%22&path=%22upload.js%22" -O result.txt`将查询结果保持到当前目录的result.txt文件中