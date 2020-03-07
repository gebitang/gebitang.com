+++
title = "BASed(一)"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "Front-End"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/fe87180af541)

>B --> A --> S

学习材料收集一下先：

- 1. PC端学习项目：[vue-element-admin](https://github.com/PanJiaChen/vue-element-admin)，DaXiang的前端使用的架子来自这个项目。
- 2. 移动端：Google搜索 `vue js mobile app example`推荐的就是 [NativeScript](https://github.com/NativeScript/NativeScript)了
- 3. Example示例教学应用是最好的[vuejsexamples.com](https://vuejsexamples.com/tag/mobile/)

先从PC端学起。最简单的方式就是 `clone repo; npm install ; npm run dev`基本上这样能跑起来，就表示环境没有问题。Windows+Mac环境目前都已经跑起来。

阅读文档。[中文：vue-element-admin-site](https://panjiachen.github.io/vue-element-admin-site/zh) 这个文档的初始版本——

*   [手摸手，带你用 vue 撸后台 系列一（基础篇）](https://juejin.im/post/59097cd7a22b9d0065fb61d2)
*   [手摸手，带你用 vue 撸后台 系列二(登录权限篇)](https://juejin.im/post/591aa14f570c35006961acac)
*   [手摸手，带你用 vue 撸后台 系列三 (实战篇)](https://juejin.im/post/593121aa0ce4630057f70d35)
*   [手摸手，带你用 vue 撸后台 系列四(vueAdmin 一个极简的后台基础模板)](https://juejin.im/post/595b4d776fb9a06bbe7dba56)
*   [手摸手，带你用 vue 撸后台 系列五(v4.0 新版本)](https://juejin.im/post/5c92ff94f265da6128275a85)
*   [手摸手，带你封装一个 vue component](https://segmentfault.com/a/1190000009090836)
*   [手摸手，带你优雅的使用 icon](https://juejin.im/post/59bb864b5188257e7a427c09)
*   [手摸手，带你用合理的姿势使用 webpack4（上）](https://juejin.im/post/5b56909a518825195f499806)
*   [手摸手，带你用合理的姿势使用 webpack4（下）](https://juejin.im/post/5b5d6d6f6fb9a04fea58aabc)

看阅读数就能发现有多少人跟我一样：热情高涨，开始看。第一篇看完了，依次递减……走马观花，上面的系列文章算过了一遍。

提到的有用链接  
- [webpack-and-spa-guide](https://github.com/wallstreetcn/webpack-and-spa-guide)  
- [introduce svg sprite technology/](https://www.zhangxinxu.com/wordpress/2014/07/introduce-svg-sprite-technology/)

---

接下来结合源码读 [中文：vue-element-admin-site](https://panjiachen.github.io/vue-element-admin-site/zh) 这个文档。

第一个问题：如何进行OAuth验证？

[How to add Auth0 Authentication to a Vue.js Application in 7 steps](https://www.storyblok.com/tp/how-to-auth0-vuejs-authentication)

使用Auth0站点的指南文章[01-login](https://auth0.com/docs/quickstart/spa/vuejs/01-login)

跟着指南做一遍，有点感觉了。 
