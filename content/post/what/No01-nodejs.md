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

## 重学前端

```
# 7 种基本类型
Undefined； 
Null； 
Boolean； 
Striing;
Number;
Symbol;
Object

#  7 种语言类型
List 和 Record： 用于描述函数传参过程。 
Set：主要用于解释字符集等。
Completion Record：用于描述异常、跳出等语句执行过程。
Reference：用于描述对象属性访问、delete 等。
Property Descriptor：用于描述对象的属性。
Lexical Environment 和 Environm Record：用于描述变量和作用域
Data Block：用于描述二进制数据。
```

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
# default registry
➜  ~ npm config get registry 
https://registry.npmjs.org/
➜  ~ npm config set registry https://registry.npm.taobao.org
➜  ~ npm config get registry                                
https://registry.npm.taobao.org/
➜  ~ npm install --global vue-cli
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

## Version规则

NPM使用语义版本号来管理代码：`MAJOR,MINOR.PATCH`  
MAJOR：这个版本号变化了表示有了一个不可以和上个版本兼容的大更改。  
MINOR：这个版本号变化了表示有了增加了新的功能，并且可以向后兼容。  
PATCH：这个版本号变化了表示修复了bug，并且可以向后兼容。
```
"dependencies": {
    "bluebird": "^3.3.4", //>=3.3.4 <4.0.0
    "body-parser": "~1.15.2" // >=1.15.2 <1.16.0  
}
```
波浪符号（~）：更新到当前minor version中最新的版本。body-parser:~1.15.2，这个库会去匹配更新到1.15.x的最新版本，如果出了一个新的版本为1.16.0，则不会自动升级。

插入符号（^）：更新到当前major version中最新的版本。bluebird:^3.3.4，这个库会去匹配3.x.x中最新的版本，但是他不会自动更新到4.0.0。  

波浪符号是曾经npm安装时候的默认符号，现在已经变为了插入符号。

