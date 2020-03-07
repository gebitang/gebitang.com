+++
title = "brew关键参数"
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



[link on JianShu](https://www.jianshu.com/p/525012a055ad)

[STF之Provider系列六：brew](https://www.jianshu.com/p/1a52b09203b9)里介绍里一些基础用法。

`-s, --build-from-source`: 使用源码方式
`--force-bottle`:  使用编译好的文件。

在 `fetch`、`install`、`--Cache`中都可以使用以上两个关键参数。

bottle为brew中定义的编译好的文件。更多[名词参考](https://docs.brew.sh/Formula-Cookbook)，可以手动从[这里](https://bintray.com/homebrew/bottles/)查找、下载这些文件。

---

- 安装前自动升级

设置环境参数[HOMEBREW_NO_AUTO_UPDATE=1](https://docs.brew.sh/Manpage#environment) : If set, Homebrew will not auto-update before running brew install, brew upgrade or brew tap.

- 强制使用源码进行安装

[brew install -s formula](https://docs.brew.sh/Manpage#install-options-formula) Compile `formula` from source even if a bottle is provided. Dependencies will still be installed from bottles if they are available.

`--force-bottle`: Install from a bottle if it exists for the current or newest version of macOS, even if it would not normally be used for installation.

- 使用缓存Cache 

brew --cache: 显示cache的目录地址。
brew --cache -s formula 显示formula在当前环境下的使用源码编译时**应该**存放的地址。安装时配合 -s参数使用。
brew --cache --force-bottle formula  当前环境使用编译好的bottle存放的缓存地址。

- 下载 fetch

上述Cache参数有相同的含义，先使用fetch进行实践的下载操作，再使用Cache查看下载结果


[source code of Manpage](https://github.com/Homebrew/brew/blob/master/docs/Manpage.md)


