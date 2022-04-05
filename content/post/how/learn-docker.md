+++
title = "docker practice"
description = "the way to docker"
tags = [
    "docker",
    "tech"
]
date = "2020-03-23"
topics = [
    "docker",
    "tech"
]
draft = false
toc = true
+++

学习要扎实，知识点要进行应用才能被真正的理解。自己体悟出来的知识才是属于自己的知识。学习实践基础知识。如下每一个知识点准确理解后就构建了自己的Docker知识库

- [DOCKER基础技术：LINUX NAMESPACE（上）](https://coolshell.cn/articles/17010.html)
- [DOCKER基础技术：LINUX NAMESPACE（下）](https://coolshell.cn/articles/17029.html)
- [DOCKER基础技术：LINUX CGROUP](https://coolshell.cn/articles/17049.html)
- [DOCKER基础技术：AUFS](https://coolshell.cn/articles/17061.html)
- [DOCKER基础技术：DEVICEMAPPER](https://coolshell.cn/articles/17200.html) 

- [GO语言、DOCKER 和新技术](https://coolshell.cn/articles/18190.html)十年
- [记一次KUBERNETES/DOCKER网络排障](https://coolshell.cn/articles/18654.html)


- JavaAgent


--- 


[Docker — 从入门到实践](https://docker_practice.gitee.io/), [Source](https://github.com/gebitang/docker_practice)


这篇留在这里已经快一年了，也没啥动静。还是得**“业务”**驱动。

### 配置docker engine

默认的配置文件`insecure-registries`和`registry-mirrors`都为空，本地私服和公网镜像可以添加到配置中，类似——

```
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "features": {
    "buildkit": true
  },
  "insecure-registries": [
    "hub.xxx.com"
  ],
  "registry-mirrors": [
    "https://nmhwesdd.mirror.aliyuncs.com"
  ]
}

```

### init / tini

```
"Entrypoint": [
        "/sbin/tini",
        "--",
        "/entrypoint.sh"
    ],
```
标准的使用tini启动业务场景：使用tini作为init系统启动镜像，实际的启动脚本为`/entrypoint.sh`的内容。

- [github tini](https://github.com/krallin/tini)，其中[#issue 8](https://github.com/krallin/tini/issues/8)说明了tini的优势
  - 作为镜像的pid 1 启动
  - 负责传递外部发送的Signal如SIGTERM
  - 负责回收僵尸进程
- docker 1.3之后默认集成了tini功能，可以在`docker run `中使用参数 `--init`启用

### CMD 和 Entrypoint

从[docker tini](https://cloud-atlas.readthedocs.io/zh_CN/latest/docker/init/docker_tini.html)的dockerfile中可以看出区别

```dockerfile 
# Add Tini
ENV TINI_VERSION v0.19.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini
ENTRYPOINT ["/tini", "--"]

# Run your program under Tini
CMD ["/your/program", "-and", "-its", "arguments"]
# or docker run your-image /your/program ...
```

- 可以看到CMD的内容其实是昨晚ENTRYPOINT的参数，执行的顺序相当于`/tini -- "/your/program", "-and", "-its", "arguments"`
- CMD的内容在执行`docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`时，镜像中定义的CMD内容将被替换
- 作为对比ENTRYPOINT的内容将始终会被执行，上一步的COMMAND命令将会在其之后被执行

[参考一](https://system51.github.io/2020/09/17/docker-need-to-know/)，参考[Docker CMD vs ENTRYPOINT: What’s The Difference & How To Choose](https://www.bmc.com/blogs/docker-cmd-vs-entrypoint)

另外， CMD和ENTRYPOINT都有`exec`和`shell`两种模式——

```shell
# exec模式，推荐模式
CMD ["executable", "param1", "param2"] 
# shell模式 
CMD command param1 param2

# exec模式，推荐模式
ENTRYPOINT ["executable", "param1", "param2"]
# shell模式 在shell环境下执行command
ENTRYPOINT command param1 param2
```
### docker使用代理

[Configure Docker to use a proxy server](https://docs.docker.com/network/proxy/)，windows下需要注意的是：配置的宿主机代理时，ip地址需要使用vEthernet(WSL)虚拟网卡对应的IP+宿主机端口

登录hub.docker.com，可以先logout，会提示出准确的登录ulr: `https://index.docker.io/v1/`，将对应的image重新打tag，然后推送到公共仓库

```shell
#设置好代理之后
docker pull us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0
docker pull us-docker.pkg.dev/google-samples/containers/gke/hello-app:2.0

# 去除代理，登录到hub.com:  docker login https://index.docker.io/v1/
E:\tempd>docker images
REPOSITORY                                                  TAG       IMAGE ID       CREATED        SIZE
us-docker.pkg.dev/google-samples/containers/gke/hello-app   2.0       7318b44bb289   4 weeks ago    11.5MB
us-docker.pkg.dev/google-samples/containers/gke/hello-app   1.0       33daf70e7689   4 weeks ago    11.5MB
kindest/node                                                <none>    32b8b755dee8   7 months ago   1.12GB

# 生成新镜像  docker tag image_id tag_name
E:\tempd>docker tag 7318b44bb289 gebitang/hello:1.0

E:\tempd>docker images
REPOSITORY                                                  TAG       IMAGE ID       CREATED        SIZE
gebitang/hello                                              1.0       7318b44bb289   4 weeks ago    11.5MB
us-docker.pkg.dev/google-samples/containers/gke/hello-app   2.0       7318b44bb289   4 weeks ago    11.5MB
us-docker.pkg.dev/google-samples/containers/gke/hello-app   1.0       33daf70e7689   4 weeks ago    11.5MB
kindest/node                                                <none>    32b8b755dee8   7 months ago   1.12GB

#推送镜像
E:\tempd>docker push gebitang/hello:1.0
The push refers to repository [docker.io/gebitang/hello]
382df8daaccb: Pushed
8d3ac3489996: Pushed
1.0: digest: sha256:7d98c53d7b133ed40b7fcc55dd5d2cff62d1ee81035de6e693dfba6cadfad2b3 size: 739

E:\tempd>docker tag 33daf70e7689 gebitang/hello:2.0

E:\tempd>docker push gebitang/hello:2.0
The push refers to repository [docker.io/gebitang/hello]
856dd7b64f3c: Pushed
8d3ac3489996: Layer already exists
2.0: digest: sha256:e7389f0a331243818bc5ad057e8de0c797fe368f3076b173d6f573477bad3fc1 size: 739

E:\tempd>docker images
REPOSITORY                                                  TAG       IMAGE ID       CREATED        SIZE
gebitang/hello                                              1.0       7318b44bb289   4 weeks ago    11.5MB
us-docker.pkg.dev/google-samples/containers/gke/hello-app   2.0       7318b44bb289   4 weeks ago    11.5MB
gebitang/hello                                              2.0       33daf70e7689   4 weeks ago    11.5MB
us-docker.pkg.dev/google-samples/containers/gke/hello-app   1.0       33daf70e7689   4 weeks ago    11.5MB
kindest/node                                                <none>    32b8b755dee8   7 months ago   1.12GB

```

[Pull an Image from a Private Registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/) k8s中使用这个镜像需要验证。需要创建secret，然后指定这个secret

如果选择为公开镜像，则可以直接拉取。

- 部署失败后，可以删除`k delete -f xx.yaml`，或者查看部署内容，然后指定部署名称

```
kubectl get deploy
kubectl delete deploy xxx-name
```

### Sonar支持go工程覆盖率

[Sonar实践问题：支持go工程的覆盖率](../../../post/2020/0319-sonar-go-with-coverage/)本地执行跑完了简易流程。线上使用时要求：Java环境+Go环境

Go环境的配置比较简单，所以就Java的环境上新做一个镜像。

使用私有仓库：hub.mytest.com

- 查看本地现有的镜像`docker images` 
- 拉取一个Java8-latest的镜像 `docker pull hub.mytest.com/library/java:8-latest`
- 启动容器`docker run -it image-id` （i for interactive, t for terminal)
- 搭建go环境：
 - 下载  `wget https://dl.google.com/go/go1.14.1.linux-amd64.tar.gz`
 - 解压  `tar -C /usr/local -xzf go1.14.1.linux-amd64.tar.gz`
 - 配置变量 `echo 'export GOPATH=/usr/local/go '>>/root/.bashrc; echo 'export export PATH=$GOPATH/bin:$PATH'>>/root/.bashrc`
- 退出docker，使用commit创建新的镜像 `docker commit 96dac7c6b872 javaWithGoEnv:0.1`
- 添加tag并上传到私有仓库：`docker tag javaWithGoEnv:0.1 hub.mytest.com/gebitang/javaWithGoEnv:0.1; docker push hub.mytest.com/gebitang/javaWithGoEnv:0.1`

验证上传的镜像：
- 登录 `docker login hub.mytest.com` (windows环境下直接使用 双引号可以识别用户名密码；Mac环境下特殊字符会被截断，可以不直接使用-u -p参数)
- 执行并验证go环境`docker run -it imageid; go version`

```
exit;
#列出容器操作记录
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                         PORTS               NAMES
96dac7c6b872        2b5cf8a5a385        "/bin/bash"         5 minutes ago       Exited (130) 3 seconds ago                         wonderful_knuth
5da89c979e26        2b5cf8a5a385        "/bin/bash"         About an hour ago   Up About an hour                                   hardcore_hugle
2fd722989ad4        2b5cf8a5a385        "/bin/bash"         About an hour ago   Exited (0) About an hour ago                       trusting_lewin
a159c52429eb        b3fa0f1247c1        "/bin/bash"         About an hour ago   Exited (0) About an hour ago                       quirky_shirley
962f0252e8b0        1b261c693e4b        "/bin/bash"         2 hours ago         Exited (0) 2 hours ago                             clever_bohr
dda1589c78cf        5197ffb8c58f        "bash"              2 hours ago         Exited (0) 2 hours ago                             nifty_golick

$ docker commit --help
Usage:  docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

Create a new image from a container's changes

$ docker tag --help
Usage:  docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

$ docker login --help
Usage:  docker login [OPTIONS] [SERVER]

Log in to a Docker registry

```

--- 

```
opensources curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add - 
OK
```

--- 

还没学会如何使用DockerFile[从头开始](https://docs.docker.com/develop/develop-images/baseimages/)创建docker镜像。业务需要，直接从本地的容器里进行环境更新。

### 系统环境变量

目前的环境需要——

- Java 
- sonar-scanner
- sonar-scanner-eslint （前端扫描自定义的sonar）
- eslint-cars-server （前端封装后的服务）
- node
- go

使用root用户，将需要的环境变量添加到`/etc/profile`和`/root/.bashrc`文件下，类似——

```
export JAVA_HOME=/usr/java/jdk1.8.0_231-amd64
export GOROOT=/usr/local/go
export GOPATH=/go
export PATH=/usr/local/nodejs/node-v0.10.13-linux-x64/bin:$JAVA_HOME/bin:$GOROOT/bin:$PATH
```
从terminal里启动是没有问题，但上述配置在docker环境启动后没有生效。默认的PATH只识别 `/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin `这些内容。

可以使用软连接的方式，在现有的PATH路径下创建新的软连接——

```
ln -s /usr/local/go/bin/go /usr/local/bin/go
ln -s /usr/local/nodejs/node-v10.13.0-linux-x64/bin/node /usr/local/bin/node
ln -s /usr/local/nodejs/node-v10.13.0-linux-x64/bin/npm /usr/local/bin/npm
ln -s /usr/local/nodejs/node-v10.13.0-linux-x64/bin/eslint-cars-server /usr/local/bin/eslint-cars-server


ls -l /usr/local/bin/
total 8
lrwxrwxrwx 1 root root   20 Apr 14 04:59 go -> /usr/local/go/bin/go
lrwxrwxrwx 1 root root   50 Apr 14 05:23 node -> /usr/local/nodejs/node-v10.13.0-linux-x64/bin/node
lrwxrwxrwx 1 root root   49 Apr 14 05:23 npm -> /usr/local/nodejs/node-v10.13.0-linux-x64/bin/npm
-rwxr-xr-x 1 root root 1771 Apr 13 07:05 sonar-scanner
-rwxr-xr-x 1 root root  610 Apr 13 07:05 sonar-scanner-debug
```

### go env变量
默认的配置如下——
```
GO111MODULE=""
GOARCH="amd64"
GOBIN=""
GOCACHE="/root/.cache/go-build"
GOENV="/root/.config/go/env"
...
...
```
相同的问题，更新go env相关的变量时，采取上面更新配置文件的的方式依然无法生效。

观察默认的变量有个值为`GOENV="/root/.config/go/env"`，手动验证一下`go env -w GOPRIVATE="golang.my.test.com"`，这个值会直接保存在GOENV对应的文件下。

所有可以采用这种方式更新需要的go env变量

### docker exec -it container-id bash

[How to enter in a Docker container already running with a new TTY](https://stackoverflow.com/a/26496854/1087122)

### Docker Compose with example

[Compose file version 3 reference](https://docs.docker.com/compose/compose-file)

- 执行`docker-compose up`，启动一个服务 [Get started with Docker Compose](https://docs.docker.com/compose/gettingstarted/)
- 服务由当前目录下的 `docker-compose.yml`文件进行定义
- 服务定义中的`build`阶段可以定义`DOCKERFILE`文件
- 可以使用当前目录下的 `.env` 文件作为参数定义环境变量 [Environment variables in Compose](https://docs.docker.com/compose/environment-variables/)

- 使用`docker-compose up --build `重新构建镜像，然后再启动（这一步让项目可以正常运行起来）

```yml
version: '3'
services:
  web:
    build:
      context: .
      dockerfile: "${DOCKERFILE}"
    ports:
      - "5000:5000"
    # bind mount. dymaticly change code
    volumes:
      - .:/code
  redis:
    image: "redis:alpine"

```

其他定义并使用变量的方法——   

- 使用shell环境下的变量，例如`POSTGRES_VERSION=9.3`，则下面的postgres的版本将使用9.3版本
```
db:
  image: "postgres:${POSTGRES_VERSION}"
```

- 使用`-e`参数，例如 `docker compose run -e VARIABLE=VALUE `

强制重新编译生效： 

- 执行`docker-compose build --no-cache` 重新编译主程序
- 启动服务`docker-compose up --build` 先执行编译，再启动服务。（会自动判断与cache相比是否有更新？）

WARNING: Image for service xxx was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.

#### ElasticSearch 

[ElasticSearch API：](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html)

```
GET /<index>/_search

GET /_search

POST /<index>/_search

POST /_search
```
- 发送数据 ` curl -H "Content-Type: application/json" -XPOST "http://localhost:9200/filebeat-6.4.3-2020.07.31/doc" -d $info` 
- 查询数据 `curl http://localhost:9200/filebeat-6.4.3-2020.07.31/_search`
- 通过proxy发送数据 `curl -H "Content-Type: application/x-ndjson" -XPOST "http://localhost:9200/_bulk" -d $info` 

先将`ndjson`类型的数据读入到变量info中，`info=$(cat data.info)`。

>[NDJSON - Newline delimited JSON](https://github.com/ndjson/ndjson-spec)  
> Consists of individual lines where each individual line is any valid JSON text and each line is delimited with a newline character.

ES中调用`_bulk`接口时，默认使用这种类型。[https://github.com/elastic/elasticsearch-py/issues/609](https://github.com/elastic/elasticsearch-py/issues/609)

#### etcd

>[etcd:](https://github.com/etcd-io/etcd) Distributed reliable key-value store for the most critical data of a distributed system


从[https://github.com/etcd-io/etcd/releases/latest](https://github.com/etcd-io/etcd/releases/latest)下载对应操作系统的最新版本，目前(2020-08-12)为`3.4.10`版本。

解压后将对应的目录添加到环境变量，或直接到对应目录下使用。

（go编译打包的服务优势：windows下也只需要一个exe文件就可以直接使用——永远的“绿色免安装”版本）

- 解压安装
- `etcd.exe --version`查看版本
- 启动etcd，`etcd.exe`执行起来。
- 检查`etcdctl.exe member list`

添加key，查找key对应的值。可以使用endpoints参数指定连接的server地址

```
D:\tools\etcd-v3.4.10-windows-amd64>etcd.exe --version
etcd Version: 3.4.10
Git SHA: 18dfb9cca
Go Version: go1.12.17
Go OS/Arch: windows/amd64

D:\tools\etcd-v3.4.10-windows-amd64>etcdctl.exe --endpoints=127.0.0.1:2379 put testKey "value of testKey"
OK

D:\tools\etcd-v3.4.10-windows-amd64>etcdctl.exe --endpoints=127.0.0.1:2379 get testKey
testKey
value of testKey

```

Watcher功能可用来实现动态控制功能。

[A concise tutorial of golang etcd](https://programmer.help/blogs/a-concise-tutorial-of-golang-etcd.html)

[etcd架构介绍](https://segmentfault.com/a/1190000020742981)  

[etcd常用操作介绍](https://segmentfault.com/a/1190000020787391)

[etcd demo](https://etcd.io/docs/v3.4.0/demo/)

#### 调用链
```
//调用链
main.go --> init() --> newServerCmd() *cobra.Command
//异步执行，进入实际业务逻辑
// confPath路径固定？ /etc/esproxy/config.toml
// server.go
serveProxy() --> 
    InitProxyMux() ——初始化内容过多:(
    wrapperMiddleware 添加http请求的中间件

decode -> processor chains -> encode -> dispatchToES

cmd/inject/log_pipeline_inject.go

pre Chain： 
    alerterPipeline
    routePipeline
    ratelimitePipeline

pos Chain:
    courierPipeline

//依赖的组件包括：组件工作的关系？应用的场景
grafana
kafka
mysql
prometheus
etcd
elasticsearch
filebeat
redis
```


### 删除container，删除images

[docker rm](https://docs.docker.com/engine/reference/commandline/rm/) 

[Remove Docker Containers, Images, Volumes, and Networks](https://linuxize.com/post/how-to-remove-docker-images-containers-volumes-and-networks)

- `docker rm -vf $(docker ps -aq)` To delete all containers including its volumes use
- `docker rmi -f $(docker images -aq)` To delete all the images 

```
# list all containers 
docker container ls -a

# remove container id
docker container rm cc3f2ff51cab cd20b396a061

# remove all stopped containers 
docker container prune
```

### Cannot create container for service filebeat: Drive has not been shared

windows环境下使用`docker-compose up`时报错，提示类似`Cannot create container for service filebeat: Drive has not been shared`的错误。

[解决方案](https://github.com/Cyb3rWard0g/HELK/issues/79#issuecomment-397637262) ，包含[官方issue #4303](https://github.com/docker/compose/issues/4303)

- 环境变量设置正确，`.env`文件中设置环境变量`COMPOSE_CONVERT_WINDOWS_PATHS=1`，或者在命令行执行 `set COMPOSE_CONVERT_WINDOWS_PATHS=1`\
- 设置settings的Shared Dreives选项` Windows settings > Shared Drives > Reset credentials > select drive > Apply`
- 正确配置了windows下docker的share volumns，参考[微软的说明](https://docs.microsoft.com/en-us/archive/blogs/stevelasker/configuring-docker-for-windows-volumes)

### slow on Windows

相同版本，相同docker配置（2核2G）。Windows上宿主机更强（Intel Core i7-8700 3.2GHz / 32GB Memory），Mac（Intel Core i5/ 8GB Memory）

Docker Community Edition 

- Version 17.06.2-ce-win27 (13194) Channel: stable 428bd6c
- Version 17.06.0-ce-mac18 (18433) Channel: stable d9b66511e0


跑相同的compose工程。启动速度Windows上差一大截，[官方issue](https://github.com/docker/for-win/issues/1936)吐槽，有人[建议](https://github.com/docker/for-win/issues/1936#issuecomment-380257581)——

**Here are directories to exclude**

- development project directories that contain .git repos or node_modules, etc
- docker image directories
- anything with thousdands of files

**What services to configure exclusions on**

- add directory to Windows Defender Exclusion list [https://support.microsoft.com/en-us/help/4028485/windows-10-add-an-exclusion-to-windows-defender-antivirus](https://support.microsoft.com/en-us/help/4028485/windows-10-add-an-exclusion-to-windows-defender-antivirus)
- add directory to Windows Indexing exclusion list-- Control Panel -> Indexing Options -> Add all of your development directories

别瞎折腾了，迁移到Mac/Linux上跑不香吗~ 

## 由浅入深吃透 Docker

[1元课程：](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=455)

Docker 是利用 Linux 的 Namespace 、Cgroups 和联合文件系统三大机制来保证实现的：使用 Namespace 做主机名、网络、PID 等资源的隔离，使用 Cgroups 对进程或者进程组做资源（例如：CPU、内存等）的限制，联合文件系统用于镜像构建和容器运行环境。