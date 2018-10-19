+++
title = "Nodejs Vue"
description = "dashboard"
tags = [
    "work"
]
date = "2018-10-01"
topics = [
    "work"
]
toc = true
+++

[Use Dashboard](https://zenhabits.net/dashboard/) to [Go Deeper, Not Wider](http://www.raptitude.com/2017/12/go-deeper-not-wider/) 

## Window env
安装完成后，自带npm目录，可以在安装目录的`/nodejs/node_modules/npm/html/doc/index.html`查看帮助文档

```
npm -l 

npm <command> -h  quick help on <command>
npm -l            display full usage info
npm help <term>   search for help on <term>
npm help npm      involved overview

Specify configs in the ini-formatted file:
    C:\Users\joechin\.npmrc
or on the command line via: npm <command> --key value
Config info can be viewed via: npm help config

npm@6.4.1 C:\Program Files\nodejs\node_modules\npm
```

### 配置国内repository

1. 临时使用`npm --registry https://registry.npm.taobao.org install express`

2. 持久使用 `npm config set registry https://registry.npm.taobao.org`

3. 通过cnpm使用`npm install -g cnpm --registry=https://registry.npm.taobao.org`
 
>使用cnpm进行安装 `cnpm install express`


配置后可通过下面方式来验证是否成功 
```
# 查看配置的repository信息
npm config get registry
# View registry info for express
npm info express
```

```
# save to app.js, then: node app.js
# visit http://localhost:3000, and you will see a message 'Hello World'
const http = require('http');

const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World\n');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});

```