+++
title = "STF之从stf-local说开去"
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



[link on JianShu](https://www.jianshu.com/p/b536c700cf38)

STF使用命令行`stf local`方式运行。

[Node.js 命令行程序开发教程](http://www.ruanyifeng.com/blog/2015/05/command-line-with-node.html)，执行了 [npm link](https://docs.npmjs.com/cli/link)后，实际上执行的就是 `lib/cli/index.js` 中的命令。

使用yargs模块实现命令行支持
- options 方法允许将所有配置写进一个对象
```
#!/usr/bin/env node
var argv = require('yargs')
  .option('f', {
    alias : 'name',
    demand: true,
    default: 'tom',
    describe: 'your name',
    type: 'string'
  })
  .usage('Usage: hello [options]')
  .example('hello -n tom', 'say hello to Tom')
  .help('h')
  .alias('h', 'help')
  .epilog('copyright 2015')
  .argv;

console.log('hello ', argv.n);
# 英文参数本身说明了用法
```
- [子命令command用法](https://www.cnblogs.com/bymax/p/5748662.html)

```
.command(cmd, desc, [builder], [handler])
.command(cmd, desc, [module])
.command(module)
# module必须导出四个变量
# cmd, desc [builder], [handler]，其中builder和handler是方法，另外两个是字符串
```
所以  `stf local`实际由 `lib/cli/local/index.js` 处理。这里也是实际的程序入口。

基于[systemd](http://www.freedesktop.org/wiki/Software/systemd/)的思想，stf在migrate完成后同时启动了多个process/unit开始提供服务。

下一步：
学习各个进程见是如何使用  [ZeroMQ](http://zeromq.org/) 和 [Protocol Buffers](https://github.com/google/protobuf)进行通信的。

代码结构梳理，可参考[STF 二次开发辛酸之路](https://testerhome.com/topics/6114)
>

