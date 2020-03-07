+++
title = "Jekyll-on-Windows"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "Mixin-lol"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/8f8133631870)

看了[官方文档](https://jekyllrb.com/docs/installation/windows/)才意识到Windows 10已经默认支持了[windows subsystem for linux](https://docs.microsoft.com/en-us/windows/wsl/about?redirectedfrom=MSDN)，直接在命令行里执行 `bash`就可以切换到linux环境。

查看Windows 10的系统版本： cmd 运行 `winver`即可查看。或者在`window + x`打开控制面板查看

当然还是需要先安装一下[Windows Subsystem for Linux Installation Guide for Windows 10](https://docs.microsoft.com/en-us/windows/wsl/install-win10)

还是专注在Jekyll本身吧，使用exe方式安装即可，暂时不折腾Windows上的linux了。

---

Jekyll本身[依赖环境](https://jekyllrb.com/docs/installation/#requirements)包括：  
*   [Ruby](https://www.ruby-lang.org/en/downloads/) version **2.4.0** or above, including all development headers (ruby version can be checked by running `ruby -v`)
*   [RubyGems](https://rubygems.org/pages/download) (which you can check by running `gem -v`)
*   [GCC](https://gcc.gnu.org/install/) and [Make](https://www.gnu.org/software/make/) (in case your system doesn’t have them installed, which you can check by running `gcc -v`,`g++ -v` and `make -v` in your system’s command line interface)

- 安装ruby环境完成后，使用gem安装jekyll：`gem install jekyll bundler` （27 gems installed）
- 检查是否安装成功 `jekyll -v` （jekyll 4.0.0）

搞定。

---

Window环境下使用 MSYS2 based RubyInstaller for Windows [https://rubyinstaller.org](https://rubyinstaller.org/) 安装。[rubyinstaller2 ](https://github.com/oneclick/rubyinstaller2#using-the-installer-on-a-target-system)本身是一个开源软件。最新的下载版本[下载页面](https://rubyinstaller.org/downloads/)

RubyInstaller combines the possibilities of native Windows programs with the rich UNIX toolset of [MSYS2](http://www.msys2.org/) and the [large repository of MINGW libraries](https://github.com/Alexpux/MINGW-packages). It is a great foundation to use Ruby for development and production

安装完成后，可以勾选择安装 [MSYS2](http://www.msys2.org/)( a software distro and building platform for Windows)

也可以安装完成后，手动安装`ridk install`，可以使用 `ridk help`查看帮助

![ridk install](https://upload-images.jianshu.io/upload_images/3296949-ceb874c72ced758d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![finish ridk install](https://upload-images.jianshu.io/upload_images/3296949-c3e2acfbfd25213f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



