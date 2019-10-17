+++
title = "How to develop on MacOS for Apple platform"
description = "xcode"
tags = [
    "mac",
    "apple"
]
date = "2019-05-15"
topics = [
    "mac",
    "apple",
    "dev"
]
draft = false
toc=true
+++


[苹果开发者账号开启双重认证教程](https://www.jianshu.com/p/20c5e199d47d)
注册Apple账户且在iOS设备上进行登录（Phone or Pad），在账户信息页面选择 “密码与安全性”，开启双重认证。——目前，不开启双重认证的账户无法注册为开发者账户。

[开发者账户申请](http://www.applicationloader.net/blog/zh/547.html)
开启账户双重认证后，注册开发者时，可以选择短信方式进行验证。填写完所有必要信息后，会先收到订单确认信息，但需要等等审核通过。9号11:14申请开发者账号，10号14:00申请通过。

>快的话付款后十几分钟后就能用，一般要审核一两天，有可能会发邮件让补充地址信息或者上传身份证，留意好邮件就行了。

...develop your application...

[iOS开发之App打包上传详细步骤](http://www.xlgz520.com/2019/02/12/iOS%E4%B9%8BApp%E6%89%93%E5%8C%85%E4%B8%8A%E4%BC%A0%E4%B8%80%E6%9D%A1%E9%BE%99/)
其中第三方上传工具可以使用app-specific的密码 [Using app-specific passwords](https://support.apple.com/en-us/HT204397)

[What is a provisioning profile & code signing in iOS?](https://medium.com/@abhimuralidharan/what-is-a-provisioning-profile-in-ios-77987a7c54c2)

---

下面为官方内容，研究透下面的内容，不用再搜索任何其他资料了:) 
[developer apple center](https://developer.apple.com/support/)  

- [developer account](https://help.apple.com/developer-account/)
- [xcode](https://help.apple.com/xcode/mac/current/#/devc8c2a6be1)
- [app store connect](https://help.apple.com/app-store-connect/)
- [topic articles](https://developer.apple.com/support/articles/)
- [documentation](https://developer.apple.com/documentation/)

### Apple disk space issue

System Storage taken too much space   
存储空间只有3GB可用，频繁提示“空间不足”且这个提示无法禁止。——最终只能暂时打开勿扰模式。

查看系统占用近70GB。根据这个[视频说明](https://www.youtube.com/watch?v=4E-sX1NxW9o)实际是正常的。这个占用实际上包括了：System + Library（system + user）。这样可以查看这三个对应的文件夹到底哪些应用占用了大量空间。例如：删除Android sdk下载的多个platform之后，就释放了7GB左右的空间。

Other volumns，容器中的其他卷：[What is 'Other Volumes'?](https://apple.stackexchange.com/questions/305094/what-is-other-volumes)  

You can list these with `diskutil apfs list`. The standard configuration of such a container is as follows:

- disk1s1, the volume you boot from, mounted at /, shown in Disk Utility as Macintosh HD
- disk1s2, ‘Preboot’, not mounted, hidden
- disk1s3, ‘Recovery’, not mounted, hidden
- disk1s4, ‘VM’, mounted at /private/var/vm, hidden 

The last 3 are grouped as Other Volumes in Disk Utility. They're required by macOS and shouldn't be removed.




### 快捷键 锁屏

[macOS 使用手册](https://support.apple.com/zh-cn/guide/mac-help/welcome/mac)

系统自带锁屏快捷键：Ctrl + Command + Q
注意：QQ的快捷键和这个有冲突，需要修改QQ的快捷键。

[Mac 键盘快捷键](https://support.apple.com/zh-cn/HT201236)

Control–Shift–电源按钮 将显示器置于睡眠状态。  
Option–Command–电源按钮 将您的 Mac 置于睡眠状态

### 制作ubuntu启动盘

[使用自带磁盘工具+Etcher](https://cto.eguidedog.net/node/826)  
- 应用->其它->磁盘工具  
- 选择U盘，然后擦除
- 选在格式: MS-DOS (FAT)  
- 打开[Etcher](https://etcher.io/)
- 选择Ubuntu镜像文件
- 选择U盘
- 点击“Flash”刻录

使用命令行[制作](https://www.cnblogs.com/Primzahl/p/10524679.html), [Mac 制作 Ubuntu 18.04 启动盘](https://www.jianshu.com/p/0abdd301e0d6)    
```
cd Downloads/
# 1. 制作系统.img
hdiutil convert -format UDRW -o ubuntu.dmg ubuntu-18.04.2-desktop-amd64.iso
# 找到U盘挂载的目录
diskutil list
# 取消 U盘 的挂载（但是不要拔掉）
diskutil umountDisk /dev/disk5
# 制作启动盘
mv ubuntu.dmg ubuntu.iso
sudo dd if=./ubuntu.iso of=/dev/disk5 bs=1m

```

### XCode 更新更换账号

[How to update Xcode with a new Apple ID?](https://stackoverflow.com/a/13613340/1087122)  
XCode安装时使用到时旧ID；后续App Store更换了新ID，XCode更新时提示还是使用到旧ID。

1. Open Finder and navigate to Applications,
2. Ctrl+Click XCode and choose "Show Package Contents",
3. Expand the Contents directory and click _MASReceipt to select it,
4. Type Command+Delete to delete the directory permanently---you will be prompted for your credentials since this is a protected file.


### brew install local file

[How to install a local file in Homebrew](http://mygeekdaddy.net/2014/12/05/how-to-install-a-local-file-in-homebrew/)  
[How to use Homebrew to install local archive](https://apple.stackexchange.com/a/328946)

```
# download your archive
wget https://github.com/zeromq/libzmq/releases/download/v4.3.2/zeromq-4.3.2.tar.gz

# move the archive to your brew --cache
mv zeromq-4.3.2.tar.gz $(brew --cache)

# then install the archive as normal
brew install zeromq
```

使用`brew cat zeromq`查看zeromq的下载地址 

### xip format
[xip format install](https://ktatsiu.wordpress.com/2016/10/28/how-to-download-xcode-8-x-with-xip-format/)  

To verify that a package is signed, execute the following command on a computer with the package:
```
pkgutil –check-signature /path/to/package.pkg
```
If the output is “Status: signed by a certificate trusted by Mac OS X”, the package is signed. If the output is “Status: no signature”, the package is not signed.

### 运行多个Xcode版本
[from Baidu](https://jingyan.baidu.com/article/f0062228091dd1fbd3f0c8de.html)  
显示当前版本：`xcode-select --print-path`   
更换版本 `sudo xcode-select --switch /Applications/Xcode-beta.app/Contents/Developer`

### Mac使用brew安装python3

[install python3 on Mac](https://wsvincent.com/install-python3-mac/)    
[Permission denied ](https://www.jianshu.com/p/965fd0be3f0c)  
```
brew install python3
# 有可能出现文件夹不存在的错误和权限异常，brew会自动给出提示，安装提示进行赋权即可
sudo mkdir /usr/local/Frameworks
sudo chown $(whoami):admin /usr/local/Frameworks

#报错
Permission denied @ dir_s_mkdir - /usr/local/Frameworks

# do 
brew link python
```

#### python3: Virtual Environments

- python3 内置虚拟环境 venv 模块  
- 定义虚拟环境所在目录，如 `~/.virtualenvs`
- 新建一个虚拟环境目录 `python3 -m venv ~/.virtualenvs/firstenv`
- 激活这个虚拟环境 `source ~/.virtualenvs/firstenv/bin/activate`
- 退出虚拟环境 `deactivate`或关闭命令行即可

激活时，可使用 `pip freeze`查看虚拟环境下的所有已安装软件

```
➜  ~ mkdir .virtualenvs
➜  ~ python3 -m venv .virtualenvs/firstenv
➜  ~ source .virtualenvs/firstenv/bin/activate
(firstenv) ➜  ~ pip freeze
(firstenv) ➜  ~ deactivate
```

### 查看MacOS版本

```
# sw_vers -productVersion 

➜ sw_vers -productVersion
10.14.6
➜ sw_vers
ProductName:	Mac OS X
ProductVersion:	10.14.6
BuildVersion:	18G87

# Generates a text report with the standard detail level.
system_profiler SPSoftwareDataType

```