+++
title = "Gitlab-CE"
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



[link on JianShu](https://www.jianshu.com/p/a9c8b9fc61aa)

[官方快速入门文档](https://docs.gitlab.com/ce/ci/quick_start/README.html)  
`.gitlab-ci.yml`文件[官方示例](https://gitlab.com/gitlab-org/gitlab-foss/blob/master/.gitlab-ci.yml)   
[.gitlab-ci.yml完整说明](https://docs.gitlab.com/ce/ci/yaml/README.html)

语法验证，可以在项目对应的目录`/gitlab-org/project-123/-/ci/lint` 进行验证
查看搭建的gitlab系统版本`/api/v4/version`，返回类似`{"version":"11.11.3","revision":"e3eeb779d72"}`的内容。

配置完成后，需要对应的`Gitlab Runner`配合执行`.gitlab-ci.yml`文件中对应的任务。[runner 官方文档](https://docs.gitlab.com/runner/)。

---
[安装一个CE版本](https://about.gitlab.com/install/?version=ce#ubuntu)，自己实践一下——
```
#依赖
sudo apt-get install -y curl openssh-server ca-certificates
#邮件
sudo apt-get install -y postfix
#添加源
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
#安装
sudo EXTERNAL_URL="http://your.domain:port" apt-get install gitlab-ce


Reading state information... Done
The following packages were automatically installed and are no longer required:
  libopts25 sntp
Use 'sudo apt autoremove' to remove them.
The following NEW packages will be installed:
  gitlab-ce
0 upgraded, 1 newly installed, 0 to remove and 50 not upgraded.
Need to get 742 MB of archives.
After this operation, 1,927 MB of additional disk space will be used.

```
按照固定带宽在下载，看起来要慢慢等了。

 [502 Whoops, GitLab is taking too much time to respond](https://stackoverflow.com/questions/33254100/502-whoops-gitlab-is-taking-too-much-time-to-respond)

看起来我的小破机器跑不起来gitlab的。GitLab requires at least 2GB RAM + 2GB swap memory 

---

手动安装，下载deb包之后，执行`sudo EXTERNAL_URL="http://your.domain:port" dpkg -i gitlab-ce_12.6.2-ce.0_amd64.deb`即可。

[manual install](https://docs.gitlab.com/omnibus/manual_install.html)

---


```
#Turn off all swap processes
sudo swapoff -a

#Resize the swap
sudo dd if=/dev/zero of=/swapfile bs=1G count=8
#if = input file
#of = output file
#bs = block size
#count = multiplier of blocks

#Make the file usable as swap
sudo mkswap /swapfile

#Activate the swap file
sudo swapon /swapfile
#Check the amount of swap available
grep SwapTotal /proc/meminfo
```


手动增大SWAP区域之后，至少启动起来了
![](https://upload-images.jianshu.io/upload_images/3296949-edb2e36c11231fd9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[修改主页](https://docs.gitlab.com/omnibus/settings/configuration.html#configuring-the-external-url-for-gitlab)


[安装完成后的配置](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md)

```
# Start all GitLab components
sudo gitlab-ctl start

# Stop all GitLab components
sudo gitlab-ctl stop

# Restart all GitLab components
sudo gitlab-ctl restart

# check log 
sudo gitlab-ctl tail
```
