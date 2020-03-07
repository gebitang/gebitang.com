+++
title = "Mixin-Nodes-Info"
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



[link on JianShu](https://www.jianshu.com/p/dc074c4bbae4)

根据[这里](https://github.com/MixinNetwork/mixin/blob/master/config/nodes.json)的公开信息，做了一点整理工作。 

在[ipapi网站](http://ipapi.com/)注册后，可以利用接口查询域名或ip的地址信息。再对数据做简单的整理之后可以得到这样的列表内容。

上面的内容耗费不少精力，最后发现获取回来的GEO信息由偏差。结合这两个帮助信息[refer 1](https://www.tutsmake.com/add-show-multiple-markers-pins-on-google-map/)、[refer 2]( https://gist.github.com/parth1020/4481893)效果图都出来了。

类似这样——

![](https://upload-images.jianshu.io/upload_images/3296949-1bfbd843b52871b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但这个数据看起来有问题，从[ipip.net](https://www.ipip.net/ip.html)站点获取的信息显示得更准确。上面的图需要推到重做了。

手动查一遍也不是不可以，实际上已经获取回来了所有的信息。但……还是应该可以自动化更好。 

---
更新一下，现在这个是正确的——


