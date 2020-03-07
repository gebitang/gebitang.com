+++
title = "BASed(二)"
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



[link on JianShu](https://www.jianshu.com/p/42819db35fe6)

[BASed(一)](https://www.jianshu.com/p/fe87180af541)里提到——
>使用Auth0站点的指南文章[01-login](https://auth0.com/docs/quickstart/spa/vuejs/01-login)
>
>跟着指南做一遍，有点感觉了。 

走遍天下都一样，代码永远比文档快。跟着指南走最终的效果还是跟官方的[代码库](https://github.com/auth0-samples/auth0-vue-samples/tree/master/01-Login)还略有差异。

前提：Node环境

步骤流程：
- 注册Auth0站点并创建一个应用。将回调和退出登录的URL都配置为默认的`http://localhost:3000`
- 全局安装vue-cli, `npm -g install @vue/cli `
- 使用cli创建基础站点`vue create my-project` 
- 安装路由插件`vue add router` 采用 history模式。

```
# Install the CLI
npm install -g @vue/cli

# Create the application using the Vue CLI.
# When asked to pick a preset, accept the defaults
vue create my-app

# Move into the project directory
cd my-app

# Add the router, as we will be using it later
# Select 'yes' when asked if you want to use history mode
vue add router
```
注意上面的注释提示。不确定我遇到的问题是否是因为没有完全按照提示操作导致的。

## 问题 1
- 1. core-js 导致的编译错误， 参考这里[core-js module error](https://github.com/vuejs/vue-cli/issues/3678)

提示错误——
```
ERROR  Failed to compile with 1 errors                                                                                                

This dependency was not found:

* core-js/modules/es.object.to-string in ./src/router/index.js

To install it, you can run: npm install --save core-js/modules/es.object.to-string
```

默认的 `babel.config.js`内容需要做对应的修改
```
module.exports = {
  presets: [
    '@vue/cli-plugin-babel/preset'
  ]
}

# 修改为
module.exports = {
  presets: [
    ['@vue/app', {
      polyfills: [
        'es6.promise',
        'es6.symbol'
      ],
      useBuiltIns: "entry"
    }]
  ]
}

```

## 问题 2
- 2. 提示找不到 Navigating to current location （xxx） is not allowed 

这个就完全是自己引入路径不正确导致的（诡异了，昨天看的文章操作方式是新建route文件夹，里面房index.js的方式，今天复盘大家都是直接添加 router.js的方式）

采用文件夹的方式时 router下的index.js引用profile时为 `
import Profile from '../views/Profile.vue'`方式；直接在router.js里引用则可以`import Profile from "./views/Profile.vue";`

真是见鬼了。通过这个练习，可以简单理解Vue框架下如何添加页面，如何配置路由。算是在门槛边上向里面撇了一眼。

---

详细步骤历史：

### 使用Vue cli创建项目

```
vue create my-project

Vue CLI v4.1.2
? Please pick a preset: Manually select features
? Check the features needed for your project: (Press <space> to select, <a> to toggle all, <i> to invert selection)Babel, Linter
? Pick a linter / formatter config: Basic
? Pick additional lint features: (Press <space> to select, <a> to toggle all, <i> to invert selection)Lint on save
? Where do you prefer placing config for Babel, ESLint, etc.? In dedicated config files
? Save this as a preset for future projects? No
? Pick the package manager to use when installing dependencies: Yarn

...


📄  Generating README.md...

🎉  Successfully created project my-project.
👉  Get started with the following commands:

 $ cd my-project
 $ yarn serve
```

### 安装route插件

```
➜  my-project git:(master) vue add router

📦  Installing @vue/cli-plugin-router...

yarn add v1.17.3
[1/4] 🔍  Resolving packages...
[2/4] 🚚  Fetching packages...
[3/4] 🔗  Linking dependencies...
[4/4] 🔨  Building fresh packages...

success Saved 1 new dependency.
info Direct dependencies
└─ @vue/cli-plugin-router@4.1.2
info All dependencies
└─ @vue/cli-plugin-router@4.1.2
✨  Done in 3.27s.
✔  Successfully installed plugin: @vue/cli-plugin-router

? Use history mode for router? (Requires proper server setup for index fallback in production) No

```
