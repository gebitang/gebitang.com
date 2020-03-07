+++
title = "STF之Rethinkdb四：导入导出"
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



[link on JianShu](https://www.jianshu.com/p/4eeccb6c8567)

[rethinkdb backup](https://rethinkdb.com/docs/backup/)
[rethinkdb import](https://rethinkdb.com/docs/importing/)

```
# export
rethinkdb dump -e stf.usageRecord -f record.tag.gz
# import
rethinkdb import -f usageRecord.json --table stf.usageRecord --force
```

使用导出功能`rethinkdb dump`需要安装对应的python driver。默认安装的`sudo pip install rethinkdb`版本为 2.4.2，跟安装的rethinkdb 2.3.6版本不兼容，会提示 `No handlers could be found for logger "rethinkdb.logger"`的[问题](https://github.com/rethinkdb/rethinkdb/issues/6724)。

```
➜  ~ rethinkdb --version
rethinkdb --version
rethinkdb 2.3.6.srh.1~0bionic (CLANG 6.0.0 (tags/RELEASE_600/final))
➜  ~ sudo pip uninstall rethinkdb
The directory '/home/stf/.cache/pip/http' or its parent directory is not owned by the current user and the cache has been disabled. Please check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
Uninstalling rethinkdb-2.4.2.post1:
  /usr/local/bin/rethinkdb-dump
  /usr/local/bin/rethinkdb-export
```

到[这里](https://pypi.org/project/rethinkdb/2.3.0.post6/)安装`pip install rethinkdb==2.3.0.post6`之后，就可以正常导出了
```
rethinkdb dump -e stf.usageRecord -f record.tar.gz
NOTE: 'rethinkdb-dump' saves data and secondary indexes, but does *not* save
 cluster metadata.  You will need to recreate your cluster setup yourself after
 you run 'rethinkdb-restore'.
Exporting to directory...
[========================================] 100%
155 rows exported from 1 table, with 0 secondary indexes
  Done (0 seconds)
Zipping export directory...
  Done (0 seconds)
```
上面导出的文件会自动生成文件夹`rethinkdb_dump_<date>_<time>.tar.gz`，里面保护对应数据表信息+数据json格式
```
rethinkdb_dump_2019-08-23T14:19:58 
└── stf
    ├── usageRecord.info
    └── usageRecord.json

1 directory, 2 files
```

导入操作：
```
rethinkdb import -f usageRecord.json --table stf.usageRecord --force
[========================================] 100%
155 rows imported in 1 table
  Done (0 seconds)
```
