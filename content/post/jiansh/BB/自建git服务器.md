+++
title = "自建git服务器"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "BB"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/d6cf0d30bc79)

之前折腾这个[Gitlab CE](https://www.jianshu.com/p/a9c8b9fc61aa)，最终小破机器上跑不起来。其实只是需要个git仓库而已。

不需要这么麻烦。[搭建Git服务器 by 廖雪峰](https://www.liaoxuefeng.com/wiki/896043488029600/899998870925664)自用的话足够了。 

1. 安装git：`apt-get install git` 已经有
2. 创建git用户，用来运行git服务：`adduser git`
3. 创建证书登录：将需要使用git登录这台机器的用户rsa文件`id_rsa.pub`添加`/home/git/.ssh/authorized_keys`文件里，一行一个。

4. 初始化Git仓库：`git init --bare longmen.git` 
```
git  init --bare longmen.git
Initialized empty Git repository in /srv/longmen.git/
```
5. owner改为git：`chown -R git:git sample.git`
6. 禁用shell登录：编辑/etc/passwd文件，将git用户的shell修改为`git-shell`，修改为：
`git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell`


然后在添加过rsa的机器上就可以克隆远程仓库了——
```
 git clone git@server.address:/srv/longmen.git
Cloning into 'longmen'...
warning: You appear to have cloned an empty repository.
```

---

表扬简书：在 [https://www.jianshu.com/settings/misc](https://www.jianshu.com/settings/misc)这里可以下载自己的全部文章（并且保留了当时创建时默认的文件格式：html或md格式）

暂时在手机端发布的都是html的格式。PC端可以切换。

