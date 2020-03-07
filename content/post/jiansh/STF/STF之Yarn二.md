+++
title = "STF之Yarn二"
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



[link on JianShu](https://www.jianshu.com/p/9c92020e854d)

yarn的确会快得飞起。 [STF之Yarn一](https://www.jianshu.com/p/7201f9ba28c3)中简单使用后，从头开始使用yarn安装STF。 

进入到bower安装前端模块时，会遇到一些问题，类似——
```
➜  RemoteDevice git:(master) yarn install
yarn install v1.17.3
[1/5] 🔍  Validating package.json...
[2/5] 🔍  Resolving packages...
[3/5] 🚚  Fetching packages...
[4/5] 🔗  Linking dependencies...
[5/5] 🔨  Building fresh packages...
success Saved lockfile.
$ bower install && not-in-install && gulp build || in-install
bower angular-hotkeys#~1.6.0    cached https://github.com/chieffancypants/angular-hotkeys.git#1.6.0
bower angular-hotkeys#~1.6.0  validate 1.6.0 against https://github.com/chieffancypants/angular-hotkeys.git#~1.6.0
bower angular-growl-v2#~0.7.9   cached https://github.com/JanStevens/angular-growl-2.git#0.7.9
bower angular-growl-v2#~0.7.9 validate 0.7.9 against https://github.com/JanStevens/angular-growl-2.git#~0.7.9
bower angular-borderlayout#7c9716aebd9260763f798561ca49d6fbfd4a5c67           cached git://github.com/filearts/angular-borderlayout.git#7c9716aebd
bower angular-borderlayout#7c9716aebd9260763f798561ca49d6fbfd4a5c67         validate 7c9716aebd against git://github.com/filearts/angular-borderlayout.git#7c9716aebd9260763f798561ca49d6fbfd4a5c67
bower ng-context-menu#~1.0.5                                                  cached https://github.com/AdiDahan/ng-context-menu.git#1.0.5
bower ng-context-menu#~1.0.5                                                validate 1.0.5 against https://github.com/AdiDahan/ng-context-menu.git#~1.0.5
bower ng-table#~1.0.0-beta.9                                                 EINVRES Request to https://bower.herokuapp.com/packages/ng-table failed with 502
✨  Done in 194.98s.
```

第一次只注意到了 `Done `，没注意到`failed with 502`，直接启动，会遇到`Module not found: Error: Cannot resolve module 'angular' ` 这是因为bower没有安装完整前端需要到模块。 

参考[Bower installation of Angular fails due to use of deprecated Bower registry on Heroku ](https://github.com/MicrosoftDocs/azure-docs/issues/10859) ，修改工程下到 .bowerrc 文件—— 

```
{
 "directory": "res/bower_components",
  "registry": "https://registry.bower.io"
}
```
还是可能出现其他安装错误，因为要从github验证并下载模块。 多试几次`yarn install`，最后出现树状输出则表示安装成功。

```
doc-ready#1.0.4 res/bower_components/doc-ready
└── eventie#1.0.6

fizzy-ui-utils#1.0.1 res/bower_components/fizzy-ui-utils
├── doc-ready#1.0.4
└── matches-selector#1.0.3

matches-selector#1.0.3 res/bower_components/matches-selector

lodash#3.10.1 res/bower_components/lodash
✨  Done in 76.18s.
```

[Yarn global command not working](https://stackoverflow.com/questions/40317578/yarn-global-command-not-working)
使用yarn安装到global的应用未生效问题 [yarn issues 1321](https://github.com/yarnpkg/yarn/issues/1321)



