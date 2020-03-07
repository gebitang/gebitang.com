+++
title = "STF之Docker系列一"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "STF"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/eb873d5fe759)

docker cp foo.txt mycontainer:/foo.txt
docker cp mycontainer:/foo.txt foo.txt
For emphasis, mycontainer is a container ID, not an image ID.

[zeromq install require libsodium](https://kerpanic.wordpress.com/2015/07/09/zero-mq-installation-package-requirements-libsodium-were-not-met/)

./configure --without-libsodium
[above not work, see this](https://www.unleashnetworks.com/blog/?p=651)


[stf on ubuntu](https://testerhome.com/articles/17696)
[[centos7][stf] 环境搭建](https://testerhome.com/topics/11055)


apt-get install android-tools-adb 

[docker run -it --rm ](https://docker_practice.gitee.io/image/pull.html)

### 容器在，镜像不能删除
如果有用这个镜像启动的容器存在（即使容器没有运行），那么同样不可以删除这个镜像。之前讲过，容器是以镜像为基础，再加一层容器存储层，组成这样的多层存储结构去运行的。因此该镜像如果被这个容器所依赖的，那么删除必然会导致故障。如果这些容器是不需要的，应该先将它们删除，然后再来删除镜像。

### EXPOSE 和 -p参数
EXPOSE 和在运行时使用 -p <宿主端口>:<容器端口> 区分开来。-p，是映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问，而 EXPOSE 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。

### 进入容器

[进入容器](https://docker_practice.gitee.io/container/attach_exec.html)

```
$ docker run -dit ubuntu
69d137adef7a8a689cbcb059e94da5489d3cddd240ff675c640c8d96e84fe1f6

$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
69d137adef7a        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           zealous_swirles

$ docker exec -i 69d1 bash
ls
bin
boot
dev
...

$ docker exec -it 69d1 bash
root@69d137adef7a:/#

```
如果从这个 stdin 中 exit，不会导致容器的停止。这就是为什么推荐大家使用` docker exec` 的原因。

更多参数说明请使用 `docker exec --help `查看。
