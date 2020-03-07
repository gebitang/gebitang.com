+++
title = "Desktop-App"
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



[link on JianShu](https://www.jianshu.com/p/4d8f6ef0586c)

[how to clean cache in yarn](https://stackoverflow.com/questions/39991508/how-to-clear-cache-in-yarn)
`yarn cache clean`

安装下面的 操作在Window环境下执行OK了。没想到在Mac环境上反而出了问题。
```
git clone https://github.com/MixinNetwork/desktop-app.git
cd desktop-app
yarn install
yarn electron:serve
```

一开始是`yarn install`时无法下载，这个问题多试几次基本就OK了。 最终编译通过，启动时却报错——
```
A JavaScript error occurred in the main process
Uncaught Exception:
TypeError: Cannot read property 'show' of undefined
    at App.eval (webpack:///./src/background.js?:106:9)
    at App.emit (events.js:200:13)
```
![](https://upload-images.jianshu.io/upload_images/3296949-11b04e9ff53924ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对比了一下，两边的环境不一致，重新将Mac的Node环境升级到`v12.13.1`版本，重新执行`yarn install`，需要先clean一下`yarn cache clean`。同步最新代码，依然报相同的错误。 

重新clone一份新的仓库，依然是同样问题。最后还是撸代码吧。

搜索了 `show`关键字，既然是打开窗口时出现的问题，在这里增加了输出——
```
if (win === null) {
    createWindow()
  } else {
    console.log("before show")
    console.log(win)
    win.show()
  }
```
运气不错，输出了是 `undefined`，查了一些判断undefined的方式，增加一个判断条件，启动成功。
```
if (win === null || typeof(win) == 'undefined') {
    createWindow()
  } else {
    win.show()
  }
```

merge request merged [here](https://github.com/MixinNetwork/desktop-app/pull/266). quick and cool:) 
