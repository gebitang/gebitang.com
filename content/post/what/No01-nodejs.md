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

## VueJS created() vs mounted()

[VueJS created() vs mounted()](https://devpremier.medium.com/vuejs-created-vs-mounted-life-cycle-hooks-74c522b9ceee)

- `mounted()` : it will executed before creating the component.  
- `created()` : it will executed after creating the component for render.

## requirejs

[Javascript模块化编程（三）](https://www.ruanyifeng.com/blog/2012/11/require_js.html)，目前项目中用到，回顾一下。

## Error: unable to resolve dependency tree

根据提示，重新使用 `npm install --legacy-peer-deps`(安装时忽略所有`peerDependencies`)可以解决，参考[一](https://blog.csdn.net/TIAN20121221/article/details/117173319)，或者[二](https://stackoverflow.com/questions/71582397/eresolve-unable-to-resolve-dependency-tree-while-installing-a-pacakge)

## TypeScript

[TypeScript](https://www.typescriptlang.org/)做为JS的超集，增强了Type的检查。但检查结果不影响最终的执行，因为依然使用JS相同的runtime。例如`console.log(4/[])`对应JS来说是有效的(- -|自由的过了火?)，返回`Infinity`，对于TS来说检查出问题：`The right-hand side of an arithmetic operation must be of type 'any', 'number', 'bigint' or an enum type.` 但编译到JS之后依然可以正常执行。参考[TypeScript: A Static Type Checker](https://www.typescriptlang.org/docs/handbook/typescript-from-scratch.html#types)

JS过于自由，所以诞生TS，支持静态检查，引入了类似静态语言的类型检查、语法支持等扩展(但扩展的内容在编译后又会擦除这一切，因为最后再编译到JS语言(使用跟JS完全相同的runtime)进行使用——这样就可以依赖现有的JS生态。TS这个切入方式厉害了。不需要单独为TS写任何框架，现有的JS框架中都可以使用TS。[TS in 5 minutes](https://www.typescriptlang.org/docs/handbook/typescript-tooling-in-5-minutes.html)

有Java背景的看一篇基础介绍基本就可以上手[TypeScript 入门教程](https://segmentfault.com/a/1190000022876390)

实际上比Java更灵活，Type类型检查只是“名义上的”，实际上是一组set，所以可以这样定义`type Foo = string | number | boolean;`。另外，“类型擦除”又类似Go语言中的“实现了所有接口方法的结构体都被认为实现了改接口”，例如——

```
interface Pointlike {
  x: number;
  y: number;
}
interface Named {
  name: string;
}

function logPoint(point: Pointlike) {
  console.log("x = " + point.x + ", y = " + point.y);
}

function logName(x: Named) {
  console.log("Hello, " + x.name);
}

const obj = {
  x: 0,
  y: 0,
  name: "Origin",
};

logPoint(obj);
logName(obj);
```

看完上面这些就可以直接找项目上手练习了。


## Jekyll Usage 

### 图片增加title和link
[参考1](https://eduardoboucas.com/blog/2014/12/07/including-and-managing-images-in-jekyll.html)、[参考2](https://stackoverflow.com/a/19360305/1087122)

处理为——
```
<figure class="image">
    <figcaption>{{ include.description }}</figcaption>
    {% if include.link %}
    <a href="{{ include.link }}" target="_blank"><img src="{{ include.url }}" alt="{{ include.description }}"> </a>
    {% else %}
    <img src="{{ include.url }}" alt="{{ include.description }}">    
    {% endif %}    
</figure>  
```

### 文章文件命名规范

Jekyll requires blog post files to be named according to the following format:

>`YEAR-MONTH-DAY-title.MARKUP`

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

```
{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

{% highlight java %}
public static void main(String[] args) {
    System.out.println("Welcome to Mixin 101");
}
{% endhighlight %}
```

### 链接方法

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/



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
[SemVer](https://semver.org/lang/zh-CN/)  
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

## guides
[Don't Block the Event Loop (or the Worker Pool)](https://nodejs.org/en/docs/guides/dont-block-the-event-loop/)  

Event Loop:  
the Event Loop executes the JavaScript callbacks registered for events, and is also responsible for fulfilling non-blocking asynchronous requests like network I/O.

Worker Pool:

1. I/O-intensive  
    1. [DNS](https://nodejs.org/api/dns.html): `dns.lookup()`, `dns.loopupService`.  
    2. [File System](https://nodejs.org/api/fs.html#fs_threadpool_usage):All file system APIs except `fs.FSWatcher()` and those that are explicitly synchronous use libuv's threadpool.  
2.  CPU-intensive  
    1. [Crypto](https://nodejs.org/api/crypto.html): `crypto.pbkdf2()`, `crypto.scrypt()`, `crypto.randomBytes()`, `crypto.randomFill()`, `crypto.generateKeyPair()`.
    2. [Zlib](https://nodejs.org/api/zlib.html#zlib_threadpool_usage)All zlib APIs except those that are explicitly synchronous use libuv's threadpool.  

## require 模块

{{< fluid_imgs
  "pure-u-1-1|https://www.runoob.com/wp-content/uploads/2014/03/nodejs-require.jpg|node require"
>}}

```
1. 如果 X 是内置模块
   a. 返回内置模块
   b. 停止执行
2. 如果 X 以 '/' 开头
   a. 设置 Y 为文件根路径
3. 如果 X 以 './' 或 '/' or '../' 开头
   a. LOAD_AS_FILE(Y + X)
   b. LOAD_AS_DIRECTORY(Y + X)
4. LOAD_NODE_MODULES(X, dirname(Y))
5. 抛出异常 "not found"

LOAD_AS_FILE(X)
1. 如果 X 是一个文件, 将 X 作为 JavaScript 文本载入并停止执行。
2. 如果 X.js 是一个文件, 将 X.js 作为 JavaScript 文本载入并停止执行。
3. 如果 X.json 是一个文件, 解析 X.json 为 JavaScript 对象并停止执行。
4. 如果 X.node 是一个文件, 将 X.node 作为二进制插件载入并停止执行。

LOAD_INDEX(X)
1. 如果 X/index.js 是一个文件,  将 X/index.js 作为 JavaScript 文本载入并停止执行。
2. 如果 X/index.json 是一个文件, 解析 X/index.json 为 JavaScript 对象并停止执行。
3. 如果 X/index.node 是一个文件,  将 X/index.node 作为二进制插件载入并停止执行。

LOAD_AS_DIRECTORY(X)
1. 如果 X/package.json 是一个文件,
   a. 解析 X/package.json, 并查找 "main" 字段。
   b. let M = X + (json main 字段)
   c. LOAD_AS_FILE(M)
   d. LOAD_INDEX(M)
2. LOAD_INDEX(X)

LOAD_NODE_MODULES(X, START)
1. let DIRS=NODE_MODULES_PATHS(START)
2. for each DIR in DIRS:
   a. LOAD_AS_FILE(DIR/X)
   b. LOAD_AS_DIRECTORY(DIR/X)

NODE_MODULES_PATHS(START)
1. let PARTS = path split(START)
2. let I = count of PARTS - 1
3. let DIRS = []
4. while I >= 0,
   a. if PARTS[I] = "node_modules" CONTINUE
   b. DIR = path join(PARTS[0 .. I] + "node_modules")
   c. DIRS = DIRS + DIR
   d. let I = I - 1
5. return DIRS
```

## exports module.exports 
[node module](https://www.runoob.com/nodejs/nodejs-module-system.html)  
如果要对外暴露属性或方法，就用 exports 就行，要暴露对象(类似class，包含了很多属性和方法)，就用 module.exports。

## install --save 

-save和save-dev可以省掉你手动修改package.json文件的步骤。  
npm install module-name -save 自动把模块和版本号添加到dependencies部分  
npm install module-name -save-dve 自动把模块和版本号添加到devdependencies部分  

## package-lock package.json
锁定安装时的包的版本号，以保证其他人在npm install时大家的依赖能保证一致。  
[source](https://blog.csdn.net/aaa333qwe/article/details/78021704)  

- 使用npm install xxx命令安装模块时，不再需要--save选项，会自动将模块依赖信息保存到 package.json 文件；  
- 安装模块操作（改变 node_modules 文件夹内容）会生成或更新 package-lock.json 文件  
- 发布的模块不会包含 package-lock.json 文件  
- 如果手动修改了 package.json 文件中已有模块的版本，直接执行npm install不会安装新指定的版本，只能通过npm install xxx@yy更新

## install error:  Unexpected end of JSON input 

[issue](https://github.com/facebook/jest/issues/6944)  
npm install error:  Unexpected end of JSON input while parsing near '...edc4cd","size":396934'
solution: npm cache clean --force

## 多版本管理

[node 多版本控制](https://segmentfault.com/a/1190000010252661)

- 安装[nvm](https://github.com/nvm-sh/nvm) 

>Node Version Manager - POSIX-compliant bash script to manage multiple active node.js versions 

```shell
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash

# 或者 Wget:
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash
```
- 查看可用版本
```
# lts long-time-stable
#nvm ls-remote --lts 
nvm list available
```
- 安装指定版本： nvm install v8.12.0

- 切换到指定版本 `nvm use v8.12.0`

- 需要安装yarn到对应版本上`npm install -g yarn`

[删除nvm](https://github.com/nvm-sh/nvm/issues/298)  
默认会安装到 $HOME/.nvm 文件夹下 + 更新了环境变量。 反向操作即可。

## nrm 管理镜像

```
安装nrm
$ npm install -g nrm

列出可用镜像
$ nrm ls

使用淘宝镜像
$ nrm use taobao 
```

## npm ERR! Cannot read property 'match' of undefined 错误处理

[Cannot read property 'match' of undefined](https://blog.csdn.net/Jane_96/article/details/81451759) 

```
# first just 
rm package-lock.json
# if it doesn't work
rm -rf node_modules
rm package-lock.json
npm cache clear --force
npm install
```

## Request to https://bower.herokuapp.com/packages/ failed with 502

[bower issue](https://stackoverflow.com/a/51020318/1087122)

## phantomjs配置
使用`npm install`的时候，可以看到会进行phantomjs的安装，有类似如下信息——
![](https://upload-images.jianshu.io/upload_images/3296949-f98fe9c9dbaf7bc3.png)

这个包[phantomjs](https://github.com/Medium/phantomjs/releases/download/v2.1.1/phantomjs-2.1.1-macosx.zip) 实际上是一个“可执行的包”，作用为 `An NPM installer for [PhantomJS](http://phantomjs.org/), headless webkit with JS API`

- 受限于网络环境，下载比较缓慢，可以进行本地安装。然后将bin目录加入\$PATH即可。 `export PATH=/Users/gebitang/phantomjs-2.1.1-macosx/bin:$PATH` 

- 如何有本地的私有源，也可以通过npm进行配置 
`npm config set phantomjs_cdnurl http://npm.stf.demo.com/mirrors/phantomjs/ -g`
[phantomjs ](https://github.com/Medium/phantomjs)If github is down, or the Great Firewall is blocking github, you may need to use a different download mirror. To set a mirror, set npm config property phantomjs_cdnurl.

更详细介绍——[README.md ](https://github.com/Medium/phantomjs/blob/master/README.md)


## npm默认安装目录

使用 `npm root -g`查看默认安装目录，`/usr/local/lib/node_modules`，可以使用修改默认的安装地址

```shell
# Make directory 
mkdir ~/.npm-global
#Set configuration folder location
npm config set prefix ~/.npm-global
#Add path to user environment path
export PATH=~/.npm-global/bin:$PATH
```

使用 `npm config list`查看配置的默认变量；使用`npm list -g`查看已安装的应用
