+++
title = "STF之新增页面三：照猫画虎"
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



[link on JianShu](https://www.jianshu.com/p/a2a4e186fc77)

目前[STF之新增页面二：融合](https://www.jianshu.com/p/b83027a4e295)方案对我来说要求还比较高，先照葫芦画瓢吧。 

参照`res/app/settings`目录下对代码结构+前端页面对比，针对使用统计，增加一个这样对目录结构——
![UsageRecord](https://upload-images.jianshu.io/upload_images/3296949-acf858b88defbb70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

配合app.js里的引入、menu.pug里的更新、以及翻译文件。可以进行页面跳转维护了。——瓢先画出个样子。

下一步是前后端的联动。目前前端还是空白，没有展示样式；后端接口也没有，怎么联动还不清楚——昨天画完瓢就是这么个状态。 

---

今天参考设备详情页面的逻辑，大概理清楚了里面的逻辑。东西越学越多，API定义使用Swagger，后端请求使用Oboe，即使pug、Angular里的scope、derective之类的还没搞清楚，不影响这个新葫芦的样子。

```
# res/app/device-list/index.js
.config(['$routeProvider', function($routeProvider) {
    $routeProvider
      .when('/devices', {
        template: require('./device-list.pug'),
        controller: 'DeviceListCtrl'
      })
  }])
  .run(function(editableOptions) {
    // bootstrap3 theme for xeditables
    editableOptions.theme = 'bs3'
  })
  .controller('DeviceListCtrl', require('./device-list-controller'))
```
设备页面打开的时候会调用 ` $scope.tracker = DeviceService.trackAll($scope)`，这个方法就是进行API调用`/api/v1/devices` 
```
# res/app/components/stf/device/device-service.js
deviceService.trackAll = function($scope) {
    var tracker = new Tracker($scope, {
      filter: function() {
        return true
      }
    , digest: false
    })

    oboe('/api/v1/devices')
      .node('devices[*]', function(device) {
        tracker.add(device)
      })

    return tracker
  }
```

根据swagger的定义，这个api的后端实现这`lib/units/api/controllers/devices.js`里

研究清楚这个设备管理的所以逻辑，整个STF也就精通了。目前只是需要这个设备详情页的逻辑就足够了。

- 获取到数据
- 可以筛选数据
