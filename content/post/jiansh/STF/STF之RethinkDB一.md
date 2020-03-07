+++
title = "STF之RethinkDB一"
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



[link on JianShu](https://www.jianshu.com/p/917faff25782)

### quick start
[quickstart](https://www.rethinkdb.com/docs/quickstart/)

```
#install
npm install rethinkdb 
# start from where it is installed.
$ rethinkdb
...
...
Listening for intracluster connections on port 29015
Listening for client driver connections on port 28015
Listening for administrative HTTP connections on port 8080
Listening on cluster addresses: 127.0.0.1, ::1
Listening on driver addresses: 127.0.0.1, ::1
Listening on http addresses: 127.0.0.1, ::1
To fully expose RethinkDB on the network, bind to all addresses by running rethinkdb with the `--bind all` command line option.
Server ready, "gebitangHoster_wil" c4bc2619-a3aa-4edf-9d4e-5c85aee397ff

#use the drivers from Node.js like this:
$ node
r = require('rethinkdb');
r.connect({ host: 'localhost', port: 28015 }, function(err, conn) {
  if(err) throw err;
  r.db('test').tableCreate('tv_shows').run(conn, function(err, res) {
    if(err) throw err;
    console.log(res);
    r.table('tv_shows').insert({ name: 'Star Trek TNG' }).run(conn, function(err, res)
    {
      if(err) throw err;
      console.log(res);
    });
  });
});

```

###  explore it from browser
- [dataexplorer from http://localhost:8080/#dataexplorer](http://localhost:8080/#dataexplorer)  
-  By default, RethinkDB creates a database named test. 

#### create table 

```
var r = require('rethinkdb');
r.db('test').tableCreate('tv_shows')
```

#### insert some JSON documents
```
var r = require('rethinkdb');
r.table('tv_shows').insert([{ name: 'Star Trek TNG', episodes: 178 },
                            { name: 'Battlestar Galactica', episodes: 75 }]);
```

#### queries 

```
var r = require('rethinkdb');
r.table('tv_shows').filter(r.row('episodes').gt(100));

# delete 
r.db('stf').table('users').filter(r.row('email').eq('test@mail.com')).delete()
```

### cook in Node
[practice in Node](https://www.rethinkdb.com/docs/guide/javascript/)  
[cook book](https://www.rethinkdb.com/docs/cookbook/javascript/)
[API ref](https://www.rethinkdb.com/api/javascript/)

#### 基本操作类型

新建db： 
```
# explore
r.dbCreate("geb");
# nodejs
r.dbCreate("blog").run(conn, function(err, result) {
    if (err) throw err;
    console.log(result);
});
```
重命名db：
`r.db("geb").config().update({name: "new_geb_name"})`

删除db：

`r.dbDrop('dbName');`

查询操作：

```
r.db('stf').table('users').filter(r.row('email').eq('am@gebitang.com')).delete()

r.db('stf').table('devices').filter(r.row('phone')("imei").eq('355967063870308'))

r.db('stf').table('devices').filter(r.row('serial').eq('4df7db0b52a6114b')).delete()
```

[api for drop table](https://www.rethinkdb.com/api/javascript/table_drop/)

文档齐全；API doc充分

[RethinkDB limitations](https://www.rethinkdb.com/limitations/)

- db个数无限制
- 表个数无限制
- 每张表最小10MB，空表默认也会占用4MB空间
- 单条记录大小无限制，但考虑到性能表现，最大不要超过16MB
- 最大查询JSON大小为64MB
- 主键最大127个字符

SQL -- NoSQL -- ReQL 对于STF来说，足够用了。更详细到数据库架构，参考[官网](https://www.rethinkdb.com/docs/architecture/)


[permissions-and-accounts](https://www.rethinkdb.com/docs/permissions-and-accounts/)
[system-tables](https://www.rethinkdb.com/docs/system-tables/)
[administration-tools ](https://www.rethinkdb.com/docs/administration-tools/)
