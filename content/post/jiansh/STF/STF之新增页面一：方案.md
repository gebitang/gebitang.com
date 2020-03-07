+++
title = "STF之新增页面一：方案"
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



[link on JianShu](https://www.jianshu.com/p/400293c94f03)

[STF之RethinkDB三:设备使用](https://www.jianshu.com/p/538e516643aa)里已经在数据库里记录里设备使用时长到信息，需要一个前端页面进行展示。

-  [ STF 二次开发辛酸之路](https://testerhome.com/topics/6114# "from testerhome")里有提到如何在当前框架下进行增加——

>比如需要增加一个task（任务管理的界面），则新建一个task文件件，里面创建几个文件：
index.js
task-controller.js
task.pug等
修改app目录下的app.js,增加require(‘./task’).name

>增加task的service到目录stf/res/app/components/stf/下增加目录task文件夹，并创建index.js与taskservice.js，用于通讯。
一般都是使用get,post请求，而stf使用的是oboe模块来接收发送的。

- 另外一个思路是直接在Angular App里使用Vue的组件，有这样的简单例子
[How to use Vue 2.0 components in an angular application](https://medium.com/@graphicbeacon/how-to-use-vue-2-0-components-in-an-angular-application-4d85bacc42dc)，[评论](https://medium.com/p/4d85bacc42dc/responses/show)里提到里在Angular 1.7里使用Vue组件的问题——[by rad Williams ](https://medium.com/@bradw2k/thanks-for-the-article-be504964bd6a)：
>In our case we load a Vue app as one route within our AngularJS 1.7 app. AngularJS wasn’t seeing the DOM loaded by our Vue components, because those rendered too late, hence AngularJS directives used within Vue component templates were ignored. One solution was to tell AngularJS to compile the new part of the DOM after the component is mounted:

```
mounted() {
 $compile(this.$el)($scope);
},
```

- 第三个方案 [ngVue](https://github.com/ngVue/ngVue) Use Vue2 components in Angular 1.x
- [Migrating an Angular 1.x app to Vue 2.x ](https://exmachina.ch/technical/migrating-angular-to-vue/)这个看起来是更好的选择

先跟着最后这个方案练手试试。实操看起来就用到了nvVue，还包括方案2的效果。
