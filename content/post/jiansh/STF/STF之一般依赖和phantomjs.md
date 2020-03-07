+++
title = "STF之一般依赖和phantomjs"
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



[link on JianShu](https://www.jianshu.com/p/ae1e013bf99e)

## brew 本地安装
只使用provider服务的话，可以不安装rethinkdb（恰好这是最大的一个文件）。`brew install graphicsmagick zeromq protobuf yasm pkg-conf`

![](https://upload-images.jianshu.io/upload_images/3296949-8dd420be0f4be2a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果需要，可以使用brew的本地安装方式。 

```
# download your archive
wget https://github.com/zeromq/libzmq/releases/download/v4.3.2/zeromq-4.3.2.tar.gz

# move the archive to your brew --cache
mv zeromq-4.3.2.tar.gz $(brew --cache)

# then install the archive as normal
brew install zeromq
```

## phantomjs配置
使用`npm install`的时候，可以看到会进行phantomjs的安装，有类似如下信息——
![](https://upload-images.jianshu.io/upload_images/3296949-f98fe9c9dbaf7bc3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个包[phantomjs](https://github.com/Medium/phantomjs/releases/download/v2.1.1/phantomjs-2.1.1-macosx.zip) 实际上是一个“可执行的包”，作用为 `An NPM installer for [PhantomJS](http://phantomjs.org/), headless webkit with JS API`

- 受限于网络环境，下载比较缓慢，可以进行本地安装。然后将bin目录加入\$PATH即可。 `export PATH=/Users/gebitang/phantomjs-2.1.1-macosx/bin:$PATH` 

- 如何有本地的私有源，也可以通过npm进行配置 
`npm config set phantomjs_cdnurl http://npm.stf.demo.com/mirrors/phantomjs/ -g`
[phantomjs ](https://github.com/Medium/phantomjs)If github is down, or the Great Firewall is blocking github, you may need to use a different download mirror. To set a mirror, set npm config property phantomjs_cdnurl.

更详细介绍——[README.md ](https://github.com/Medium/phantomjs/blob/master/README.md)




