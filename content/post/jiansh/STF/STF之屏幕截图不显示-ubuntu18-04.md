+++
title = "STF之屏幕截图不显示-ubuntu18-04"
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

[link to JianShu](https://www.jianshu.com/p/f6d334c822be)


TL, DR. 

>GraphicsMagic的依赖不满足，无法支持jpeg图片的resize功能。

解决方案： 安装对应的依赖，目前截图功能使用的为jpeg格式，仅安装jpeg支持即可`sudo apt install libjpeg9`

~~目前的环境显示libjpeg8已经安装。根据[README](http://www.graphicsmagick.org/README.html)说明——~~
>GraphicsMagick requires the Independent JPEG Group's software available from  [http://www.ijg.org/](http://www.ijg.org/) to read and write the JPEG v1 image format.

~~把[http://www.ijg.org/](http://www.ijg.org/)下的源码[jpegsrc.v9c.tar.gz](http://www.ijg.org/files/jpegsrc.v9c.tar.gz)也进行了~~
```
tar -zxvf jpegsrc.v9c.tar.gz
cd jpeg-9c 
./configure
make 
sudo make install
```
~~不确定是哪个安装起效。安装完之后~~

源码安装后，重新对GraphicsMagic进行编译安装。再次执行`gm version`时提示`gm: error while loading shared libraries: libjpeg.so.9: cannot open shared object file: No such file or directory` 

安装`sudo apt install libjpeg9`之后，显示jpeg已经支持
```
Feature Support:
  Native Thread Safe       yes
  Large Files (> 32 bit)   yes
  Large Memory (> 32 bit)  yes
  BZIP                     no
  DPS                      no
  FlashPix                 no
  FreeType                 no
  Ghostscript (Library)    no
  JBIG                     no
  JPEG-2000                no
  JPEG                     yes
  Little CMS               no
```
STF也可以正常显示截图了。

--- 

问题场景：Mac环境下搭建的STF服务截图可以正常显示；但ubuntu 18.04环境下截图不显示。 

推论：不可能是代码层面的问题，更多是环境方面的问题。 一开始怀疑是目录权限的问题。所以先梳理了一下STF的截图流程。

截图流程梳理时，修改了默认的图片上传地址并添加了log输出。——以我现在的node水平只能search+guess业务逻辑。详细过程参考下文。

梳理完之后判断代码没有问题，linux环境下依然不显示图片。Firefox会提示`The image cannot be displayed because it contains errors` Chrome下是空白图片。 

登录服务器，查看无论是上传但图片还是本地截图的图片。文件本身是完整的，也可以直接在浏览器中展示。所以可以确定是**代码环境依赖**的问题。 

在这些依赖`rethinkdb graphicsmagick zeromq protobuf yasm pkg-config`中，显然只有 `*  [GraphicsMagick](http://www.graphicsmagick.org/) (for resizing screenshots)`是跟图片展示相关的。 首先需要确保GraphicsMagick是正常的。

[graphicsmagick INSTALL-unix](http://www.graphicsmagick.org/INSTALL-unix.html#verifying-the-build)看起来可以通过命令 `gm version`进行确认是否正确安装（其实都跑起来了，安装肯定是OK的）

遇到的问题是执行 gm命令，总是提示 `fatal: Not a git repository (or any of the parent directories): .git`。使用`man gm`显示的的确是GraphicsMagic的相关内容。 

[原因](https://stackoverflow.com/questions/29452564/git-fatal-error-when-using-graphics-magick-convert-command)是安装的 oh-my-zsh的插件里`~/.oh-my-zsh/plugins/git/git.plugin.zsh`自动添加了alias `alias gm='git merge'` 注释掉即可。

对比了linux和mac环境下的gm version 

```
# under linux 
gm version
GraphicsMagick 1.3.33 2019-07-20 Q8 http://www.GraphicsMagick.org/
Copyright (C) 2002-2019 GraphicsMagick Group.
Additional copyrights and licenses apply to this software.
See http://www.GraphicsMagick.org/www/Copyright.html for details.

Feature Support:
  Native Thread Safe       yes
  Large Files (> 32 bit)   yes
  Large Memory (> 32 bit)  yes
  BZIP                     no
  DPS                      no
  FlashPix                 no
  FreeType                 no
  Ghostscript (Library)    no
  JBIG                     no
  JPEG-2000                no
  JPEG                     no
  Little CMS               no
  Loadable Modules         no
  Solaris mtmalloc         no
  OpenMP                   yes (201511 "4.5")
  PNG                      no
  TIFF                     no
  TRIO                     no
  Solaris umem             no
  WebP                     no
  WMF                      no
  X11                      no
  XML                      no
  ZLIB                     no

Host type: x86_64-pc-linux-gnu

Configured using the command:
  ./configure

Final Build Parameters:
  CC       = gcc
  CFLAGS   = -fopenmp -g -O2 -Wall -pthread
  CPPFLAGS =
  CXX      = g++
  CXXFLAGS = -pthread
  LDFLAGS  =
  LIBS     = -lm -lpthread

# under Mac
➜  ~ gm version
GraphicsMagick 1.3.31 2018-11-17 Q16 http://www.GraphicsMagick.org/
Copyright (C) 2002-2018 GraphicsMagick Group.
Additional copyrights and licenses apply to this software.
See http://www.GraphicsMagick.org/www/Copyright.html for details.

Feature Support:
  Native Thread Safe       yes
  Large Files (> 32 bit)   yes
  Large Memory (> 32 bit)  yes
  BZIP                     yes
  DPS                      no
  FlashPix                 no
  FreeType                 yes
  Ghostscript (Library)    no
  JBIG                     no
  JPEG-2000                yes
  JPEG                     yes
  Little CMS               no
  Loadable Modules         yes
  OpenMP                   no
  PNG                      yes
  TIFF                     yes
  TRIO                     no
  UMEM                     no
  WebP                     no
  WMF                      no
  X11                      no
  XML                      yes
  ZLIB                     yes

Host type: x86_64-apple-darwin18.2.0

Configured using the command:
  ./configure  '--prefix=/usr/local/Cellar/graphicsmagick/1.3.31' '--disable-dependency-tracking' '--enable-shared' '--disable-static' '--with-modules' '--without-lzma' '--disable-openmp' '--with-quantum-depth=16' '--without-gslib' '--with-gs-font-dir=/usr/local/share/ghostscript/fonts' '--with-webp=no' '--without-x' '--without-lcms2' '--without-wmf' 'CC=clang' 'CXX=clang++' 'PKG_CONFIG_PATH=/usr/local/opt/libpng/lib/pkgconfig:/usr/local/opt/freetype/lib/pkgconfig:/usr/local/opt/jpeg/lib/pkgconfig:/usr/local/opt/jasper/lib/pkgconfig:/usr/local/opt/libtiff/lib/pkgconfig' 'PKG_CONFIG_LIBDIR=/usr/lib/pkgconfig:/usr/local/Homebrew/Library/Homebrew/os/mac/pkgconfig/10.14'

Final Build Parameters:
  CC       = clang
  CFLAGS   = -g -O2 -Wall -D_THREAD_SAFE
  CPPFLAGS = -I/usr/local/opt/freetype/include/freetype2 -I/usr/include/libxml2
  CXX      = clang++
  CXXFLAGS = -D_THREAD_SAFE
  LDFLAGS  = -L/usr/local/opt/freetype/lib
  LIBS     = -lfreetype -lbz2 -lz -lltdl -lm -lpthread
```

---

### 截图流程
涉及到到文件：
![](https://upload-images.jianshu.io/upload_images/3296949-db1feb584dc4cc13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

前端入口：
```
<div ng-include="tab.templateUrl" class="ng-scope">
   <div ng-controller="ScreenshotsCtrl" class="widget-container fluid-height stf-screenshots ng-scope">
    <div class="heading">
     <button ng-click="takeScreenShot()" title="屏幕截图"
```
前端：res.app.control-panes.screenshots.screenshots-controller.js 
	定义交互方法：清除clear、截图takeScreenShot、缩放zoom
```
$scope.takeScreenShot = function() {
    $scope.control.screenshot().then(function(result) {
      $scope.$apply(function() {
        $scope.screenshots.unshift(result)
      })
    })
  }
```
scope.control.screenshot()
调用到：res.app.components.stf.control.control-service.js 
```
this.screenshot = function() {
      return sendTwoWay('screen.capture')
    }
```
socket 通信：screen.capture
-->进入websocket：lib.units.websocket.jindex.js
```
.on('screen.capture', function(channel, responseChannel) {
          joinChannel(responseChannel)
          push.send([
            channel
          , wireutil.transaction(
              responseChannel
            , new wire.ScreenCaptureMessage()
            )
          ])
        })
```

wire.ScreenCaptureMessage 的响应在——
lib.units.device.plugins.screen.capture.js 
```
router.on(wire.ScreenCaptureMessage, function(channel) {
      var reply = wireutil.reply(options.serial)
      plugin.capture()
        .then(function(file) {
          push.send([
            channel
          , reply.okay('success', file)
          ])
        })
        .catch(function(err) {
          log.error('Screen capture failed', err.stack)
          push.send([
            channel
          , reply.fail(err.message)
          ])
        })
    })
```

lib/units/device/plugins/screen/capture.js 
line: 32 : capture  --> storage.store: 

进入到：lib.units.device.support.storage.js
store方法三个参数：type, stream, meta

res.app.components.stf.storage.storage-service.js ?
StorageServiceFactory
adb截图后直接post请求传输到后台进行保存


后天响应请求：
lib.units.storage.temp.js: line 85:
```
app.post('/s/upload/:plugin', function(req, res) {
```
利用storage保存数组对象内容，并将结果以json样式返回


前端服务对应的路径时，由lib.units.storage.plgins.image.index.js处理：

```
var log = logger.createLogger('storage:plugins:image')
 app.get(
    '/s/image/:id/:name'
  , requtil.limit(options.concurrency, function(req, res) {
      var orig = util.format(
        '/s/blob/%s/%s'
      , req.params.id
      , req.params.name
      )
      return get(orig, options)
        .then(function(stream) {
          return transform(stream, {
            crop: parseCrop(req.query.crop)
          , gravity: parseGravity(req.query.gravity)
          })
        })
 ```

相当于访问：/s/blob/:id/:name
在 lib.units.storage.temp.js 
```
app.get('/s/blob/:id/:name', function(req, res) {
    var file = storage.retrieve(req.params.id)
    if (file) {
      if (typeof req.query.download !== 'undefined') {
        res.set('Content-Disposition',
          'attachment; filename="' + path.basename(file.name) + '"')
      }
      res.set('Content-Type', file.type)
      res.sendFile(file.path)
    }
    else {
      res.sendStatus(404)
    }
  })
```
如：url= http://localhost:7100/s/image/ba45cb22-1e31-4984-ac13-85c3ad5a7146/c90bde03.jpg


```
#本地stf local执行
2019-08-07T06:52:00.798Z INF/device:plugins:screen:capture 78944 [c90bde03] Capturing screenshot
2019-08-07T06:52:01.114Z INF/storage:temp 78940 [*] Uploaded "c90bde03.jpg" to "/Users/gebitang/temp/tempSaveDir/upload_db67714e97510336861651bc8bfda3b9"
2019-08-07T06:52:01.118Z INF/device:support:storage 78944 [c90bde03] { success: true,
  resources:
   { file:
      { date: '2019-08-07T06:52:01.115Z',
        plugin: 'image',
        id: '6f6967f6-53bb-4003-b518-47e29a908552',
        name: 'c90bde03.jpg',
        href: '/s/image/6f6967f6-53bb-4003-b518-47e29a908552/c90bde03.jpg' } } }
2019-08-07T06:52:01.118Z INF/device:support:storage 78944 [c90bde03] http://localhost:7100/s/upload/image  ====Uploaded to "%s" /s/image/6f6967f6-53bb-4003-b518-47e29a908552/c90bde03.jpg
2019-08-07T06:55:25.612Z INF/storage:temp 78940 [*] Cleaning up inactive resource "7e52fb33-5c04-4556-b49d-315428e963d3"
2019-08-07T06:55:46.021Z INF/device:plugins:screen:capture 78944 [c90bde03] Capturing screenshot
2019-08-07T06:55:46.407Z INF/storage:temp 78940 [*] Uploaded "c90bde03.jpg" to "/Users/gebitang/temp/tempSaveDir/upload_9e809c19cdcdbb652a865ef1a65b6bcf"
2019-08-07T06:55:46.410Z INF/device:support:storage 78944 [c90bde03] { success: true,
  resources:
   { file:
      { date: '2019-08-07T06:55:46.408Z',
        plugin: 'image',
        id: 'edea7119-a92a-4554-889f-4c9abcdaae5a',
        name: 'c90bde03.jpg',
        href: '/s/image/edea7119-a92a-4554-889f-4c9abcdaae5a/c90bde03.jpg' } } }
2019-08-07T06:55:46.410Z INF/device:support:storage 78944 [c90bde03] http://localhost:7100/s/upload/image  ====Uploaded to "%s" /s/image/edea7119-a92a-4554-889f-4c9abcdaae5a/c90bde03.jpg
```

使用provider方式配合
```
server:
2019-08-07T08:33:12.973Z INF/storage:temp 26505 [*] Uploaded "c90bde03.jpg" to "/home/stf/tempSaveDir/upload_9b7f26652e277b652419dc34eda882bf"


provider:
2019-08-07T08:33:12.415Z INF/device:plugins:screen:capture 81872 [c90bde03] Capturing screenshot
2019-08-07T08:33:12.943Z INF/device:support:storage 81872 [c90bde03] { success: true,
  resources:
   { file:
      { date: '2019-08-07T08:33:12.973Z',
        plugin: 'image',
        id: '008a3bc7-3ea4-4414-919e-cdb0497052c9',
        name: 'c90bde03.jpg',
        href: '/s/image/008a3bc7-3ea4-4414-919e-cdb0497052c9/c90bde03.jpg' } } }
2019-08-07T08:33:12.944Z INF/device:support:storage 81872 [c90bde03] http://10.112.78.174:8888/s/upload/image  ====Uploaded to  /s/image/008a3bc7-3ea4-4414-919e-cdb0497052c9/c90bde03.jpg
```
