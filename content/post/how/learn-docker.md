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

[Docker — 从入门到实践](https://docker_practice.gitee.io/), [Source](https://github.com/gebitang/docker_practice)


这篇留在这里已经快一年了，也没啥动静。还是得**“业务”**驱动。

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

### Docker Compose

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

