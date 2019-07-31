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