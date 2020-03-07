+++
title = "STF之OAuth探索二"
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



[link on JianShu](https://www.jianshu.com/p/4d70b25cfea8)

### github OAuth 
参考[示例教程](http://www.ruanyifeng.com/blog/2019/04/github-oauth.html)，在github[申请](https://github.com/settings/applications/new)好OAuth Apps（settings-Developer settings-OAuth Apps-new），配置好对应的认证字段，就可以完成使用了。

启动脚本修改为类似——
```
#!/bin/bash

# use github for demo
export STF_AUTH_OAUTH2_OAUTH_CLIENT_ID=95a2b5dd3a05a8a3efab
export STF_AUTH_OAUTH2_OAUTH_CLIENT_SECRET=45f78a43486c63a88e6d6d7467b5e0a6f37da0d3
# should use 80 port to avoid  'ECONNREFUSED 127.0.0.1:80 ' error
export STF_AUTH_OAUTH2_OAUTH_CALLBACK_URL=http://stf.demo.com/auth/oauth/callback

export STF_AUTH_OAUTH2_OAUTH_AUTHORIZATION_URL=https://github.com/login/oauth/authorize
export STF_AUTH_OAUTH2_OAUTH_TOKEN_URL=https://github.com/login/oauth/access_token
export STF_AUTH_OAUTH2_OAUTH_USERINFO_URL=https://api.github.com/user
export STF_AUTH_OAUTH2_OAUTH_SCOPE=user email

bin/stf local --public-ip stf.demo.com --poorxy-port 80 --lock-rotation --auth-type oauth2
```
STF的OAuth采用授权码模式，自动完成下列步骤——
- A 网站让用户跳转到 GitHub。
- GitHub 要求用户登录，然后询问"A 网站要求获得 xx 权限，你是否同意？"
- 用户同意，GitHub 就会重定向回 A 网站，同时发回一个授权码。
- A 网站使用授权码，向 GitHub 请求令牌。
- GitHub 返回令牌.
- A 网站使用令牌，向 GitHub 请求用户数据。

### probable problem
可能的坑是需要使用80端口启动应用，否则可能出现 `ECONNREFUSED 127.0.0.1:80` 错误，暂时还没有定位原因。可能的原因是
>node内部是用的node url parse 去解析 /api 参数的，然后再传给相应的 http request所以默认就是80端口

### flow
下面的流程可以用来参考分析应用的逻辑——Lerning by doing for Node:)
```
Navigated to http://stf.demo.com/
GET http://stf.demo.com/
[HTTP/1.1 302 Found 37ms]

GET http://stf.demo.com/auth/oauth/
[HTTP/1.1 302 Found 18ms]

Navigated to https://github.com/login/oauth/authorize?response_type=code&redirect_uri=http%3A%2F%2Fstf.demo.com%2Fauth%2Foauth%2Fcallback&scope=user&client_id=95a2b5dd3a05a8a3efab
GET https://github.com/login/oauth/authorize?response_type=code&redirect_uri=http%3A%2F%2Fstf.demo.com%2Fauth%2Foauth%2Fcallback&scope=user&client_id=95a2b5dd3a05a8a3efab
[HTTP/1.1 200 Connection established 888ms]

GET http://stf.demo.com/auth/oauth/callback?code=4f9a5580c1eb6411cf93
[HTTP/1.1 302 Found 2503ms]

GET http://stf.demo.com/?jwt=eyJhbGciOiJIUzI1NiIsImV4cCI6MTU2Mjc0OTI2MDI0M30.eyJlbWFpbCI6ImFtQGdlYml0YW5nLmNvbSIsIm5hbWUiOiJhbSJ9.8UfOXBxUiS12ZlS7NURRvwZr5RAKJ-JCBPvO7aKEpmQ
[HTTP/1.1 302 Found 50ms]

GET http://stf.demo.com/
[HTTP/1.1 200 OK 350ms]

GET http://stf.demo.com/app/api/v1/state.js
[HTTP/1.1 200 OK 17ms]

GET http://stf.demo.com/static/app/build/entry/commons.entry.js
[HTTP/1.1 304 Not Modified 23ms]

GET http://stf.demo.com/static/app/build/entry/app.entry.js
[HTTP/1.1 304 Not Modified 18ms]

GET http://stf.demo.com/static/app/build/1.099268df698652b33c03.chunk.js
[HTTP/1.1 304 Not Modified 8ms]

GET http://stf.demo.com/static/logo/exports/STF-128.png
[HTTP/1.1 200 OK 0ms]

GET http://stf.demo.com/favicon.ico
[HTTP/1.1 200 OK 0ms]

GET http://stf.demo.com:7110/socket.io/?uip=%3A%3Affff%3A127.0.0.1&EIO=3&transport=websocket
[HTTP/1.1 101 Switching Protocols 11ms]

GET http://stf.demo.com/static/app/build/96978e6fcc6395ca44894139e0baa0b2.woff
[HTTP/1.1 200 OK 0ms]

GET http://stf.demo.com/static/app/build/a8a00e89adc0ba57870098be433da27a.woff
[HTTP/1.1 200 OK 0ms]

XHR GET http://stf.demo.com/api/v1/devices
[HTTP/1.1 200 OK 52ms]

GET http://stf.demo.com/static/app/build/db812d8a70a4e88e888744c1c9a27e89.woff2
[HTTP/1.1 200 OK 0ms]

GET http://stf.demo.com/static/app/build/443a27166608ea2aef94f5cf05ff19ec.woff
[HTTP/1.1 200 OK 0ms]

GET http://stf.demo.com/static/app/build/34f4fb7db2bd6acd7b011434f72adb5c.woff
GET http://stf.demo.com/static/app/build/13313a4435455086fb63899a9f7f3966.png
[HTTP/1.1 200 OK 54ms]

GET http://stf.demo.com/static/app/devices/icon/x120/_default.jpg
[HTTP/1.1 200 OK 35ms]
```
一些大概的业务逻辑：
- 完成最后一步后，会跳转到callback的地址，由 `lib/units/auth/oauth2/index.js`处理，获取到的用户名、邮箱会使用jwtutil进行encode加密后加到url地址……
- 中间件`lib/units/app/middleware/auth.js`利用jwt参数获取用户名、邮箱，保存到数据库中

后续再详细分析……

公司内网的OAuth流程暂时还没搞定- -【update on 7-11: 直接找了接口负责人，搞定：）】

下一步简单学习一下rethinkDB的内容：
- [参考](https://testerhome.com/topics/5848)
- `lib/db/api.js`中的代码
- 启动rethinkdb后的[浏览器实现](http://localhost:8080/#dataexplorer)
