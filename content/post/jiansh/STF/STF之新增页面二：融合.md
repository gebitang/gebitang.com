+++
title = "STF之新增页面二：融合"
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



[link on JianShu](https://www.jianshu.com/p/b83027a4e295)

跟着练手： [Code](https://github.com/arcadeJHS/AngularVueIntegration) + [article](https://exmachina.ch/technical/migrating-angular-to-vue/)已经很周到了。

- tag-01-angular-app 
第一阶段：纯angular方式

$ctrl [含义](https://stackoverflow.com/a/46682826/1087122)
>**\$ctrl** is the view model object in your controller. This \$ctrl is a name you choose (vm is another most common name), if you check your code you can see the definition as `$ctrl = this`;, so basically its the this keyword of the controller function.
So now if you are using `$ctrl.latestMeasurement = 'someValue'`, then its like you are adding a property latestMeasurement to controller function.

在`angular.min.js`中可以看到 `b.controllerAs||"$ctrl",`也说明上面的定义。

import与@import的区别: 
```
alias: {
      '@': resolve('src'),
      'pages': path.join(__dirname, './src/pages')
    }
# 在index.js中，就可以这么写：import page2 from 'pages/page2'，缩短
```

跟到tag 4
