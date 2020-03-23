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
+++

这篇留在这里已经快一年了，也没啥动静。还是得**“业务”**驱动。

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

https://docker_practice.gitee.io/

https://github.com/gebitang/docker_practice


opensources curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add - 
OK
