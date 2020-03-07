+++
title = "Spring-boot服务(二)：aliyun"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "Spring"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/25d597023e22)

从头再来。旧的镜像在我安装gitlab之后[Gitlab CE](https://www.jianshu.com/p/a9c8b9fc61aa)，现在远程登录相当慢。重新初始化了一下镜像。使用强密码策略。

- 0. 安装zsh+oh my zsh `apt-get install -y zsh; sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`
- 1. 安装Maven, git, java `apt-get update; apt-get install -y maven git openjdk-8-jdk`
- 2. 生成SSH public key `ssh-keygen -t rsa -b 4096 -C "e@example.com"` 并配置多个key `~/.ssh/config` 内容如下
- 3. ~~安装mysql~~，[how-to-install-mysql-on-ubuntu-18-04](https://linuxize.com/post/how-to-install-mysql-on-ubuntu-18-04/)
- 4. 配置安全组策略，打开需要访问的端口。

```
Host github.com
    # Specifies the real host name to log into. Numeric IP addresses are also permitted.
    HostName github.com
    # Defines the username for the SSH connection.
    User gebitang
    # Specifies a file from which the user’s DSA, ECDSA or DSA authentication identity is read.
    IdentityFile ~/.ssh/id_rsa_github
Host gitlab.com
    # Specifies the real host name to log into. Numeric IP addresses are also permitted.
    HostName gitlab.com
    # Defines the username for the SSH connection.
    User gebitang
    # Specifies a file from which the user’s DSA, ECDSA or DSA authentication identity is read.
    IdentityFile ~/.ssh/id_rsa

```

> 要细心查看问题提示。Java11环境下Maven提示错误。
> 新建分支，移除mysql相关的业务逻辑。


