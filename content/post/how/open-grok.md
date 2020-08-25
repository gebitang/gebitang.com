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
- File Path (**path**)	path of the source file (no need to use dividers, or if, then use "/" - Windows users, "\" is an escape key in Lucene query syntax!   
Please don't use "\", or replace it with "/").    
Also note that if you want just exact path, enclose it in "", e.g. "src/mypath", otherwise dividers will be removed and you get more hits.  
- History	(**hist**) History log comments.

[23 Google Search Tips You'll Want to Learn](https://www.pcmag.com/how-to/23-google-search-tips-youll-want-to-learn)  
[https://www.google.com/advanced_search](https://www.google.com/advanced_search)  
[https://www.google.com/advanced_image_search](https://www.google.com/advanced_image_search)  
搜索操作符支持，类似的google支持的[Search operators](https://support.google.com/websearch/answer/2466433)

--- 

[Gitlab : List all the projects and all the groups](https://stackoverflow.com/a/45087988/1087122)

分别调用`projects`接口和`groups`接口即可，类似—— 

`curl --head "https://<host/api/v4/projects?private_token=<your private token>&per_page=100&page=<page_number>" `

只返回head部分，其中包括`X-Total`和`X-Total-Pages`的值。默认page从1开始计算，默认每页返回[20条](https://docs.gitlab.com/ce/api/#pagination)，最大返回[100个条目](https://docs.gitlab.com/11.11/ee/api/README.html#pagination)

官方推荐的[gitlab api客户端实现](https://about.gitlab.com/partners/#api-clients)

---

**任务：**将所有的项目平均分配给四个相同的服务实例分别处理。任务异步处理，时效要求不高，要所有的项目都完成后再统一发送结果

- API调用无法实现（对外提供服务是统一的出口，处理只会分配给任意的其中一台）
- 只能通过主动分别请求“任务分发”服务。使用数据库做中介
- “结果需要统一发送”
- 服务实例有可能被kill，分配给自己的任务部分还没全部完成

---

API提供接口进行任务初始化：

- 查询总量，分配任务
- 实例启动时，上报状态，
- 执行器空闲状态下轮询任务表
- 如果有任务：如果有自己：开始执行；如果没有，查询每个机器是否存活，然后更新任务表，自己接力已经被killed的机器任务
- 执行完一个任务后，更新自己所属的任务完成的数目


|id|ip|startPage|total|finish|status|
|---------|---------|---------|---------|---------|---------|
|15|10.19.3.130|25|2560|1005|1|


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