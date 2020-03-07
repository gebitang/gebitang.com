+++
title = "ios-app-打包学习"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "Apple"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/bbcd61b9e64b)

美区账户似乎搞起来比较麻烦了，既然是开源项目，在自己的Mac上编译打包安装不是也可以？正好顺带学习一下iOS开发。

直接打包ios-app时提示类似：[Xcode, Pods ProjectName.debug.xcconfig unable to open file. Wrong directory](https://forums.developer.apple.com/thread/115475)问题。

什么是pod？——Mac下Xcode下的包管理工具，[参考](https://www.jianshu.com/p/b64b4fd08d3c)，[官网文档：https://guides.cocoapods.org/](https://guides.cocoapods.org/)

### 安装

` sudo gem install cocoapods`

```
sudo gem install cocoapods
Password:
/System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/lib/ruby/2.3.0/universal-darwin18/rbconfig.rb:215: warning: Insecure world writable dir /usr/local/opt in PATH, mode 040777
Fetching: thread_safe-0.3.6.gem (100%)
Successfully installed thread_safe-0.3.6
Fetching: tzinfo-1.2.5.gem (100%)
Successfully installed tzinfo-1.2.5
...
...
Installing ri documentation for cocoapods-1.8.4
Done installing documentation for thread_safe, tzinfo, concurrent-ruby, i18n, activesupport, nap, fuzzy_match, httpclient, algoliasearch, cocoapods-core, claide, cocoapods-deintegrate, cocoapods-downloader, cocoapods-plugins, cocoapods-search, cocoapods-stats, netrc, cocoapods-trunk, cocoapods-try, molinillo, atomos, CFPropertyList, colored2, nanaimo, xcodeproj, escape, fourflusher, gh_inspector, ruby-macho, cocoapods after 28 seconds
30 gems installed
```

`pod --help`可以查看命令使用参数。 使用install 参数可以安装Podfile.lcok中定义但依赖。

Ruby生态，Podfile中也是ruby代码，类似——

```
target 'MyApp' do
  pod 'AFNetworking', '~> 3.0'
  pod 'FBSDKCoreKit', '~> 4.9'
end

```

[pod使用指南](https://guides.cocoapods.org/using/using-cocoapods.html)

