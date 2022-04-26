+++
title = "Ceph Basic"
description = "ceph practice"
tags = [
    "ceph"
]
date = "2022-02-22"
topics = [
    "ceph",
    "tech"
]
draft = false
toc = true
+++


### ceph

多看man手册

```
# 查看pool副本数量
ceph osd pool get default.rgw.buckets.test-data size

# 查看集群信息
ceph df

```

[基础操作参考](https://blog.csdn.net/a13568hki/article/details/119600249)
### radosgw-admin

多看man手册

```
# 出当前系统下所有的bucket信息
radosgw-admin bucket list

# 查看bucket详细信息
radosgw-admin bucket stats --bucket=harbor-dev

# 查看BUCKET的名称，所在的data pool, index pool. BUCKET ID.
radosgw-admin zone get
zone get
        Show zone cluster params.
zone set
        Set zone cluster params (requires infile).
zone list
        List all zones set on this cluster.

#查看用户信息，包含了用户的认证信息
radosgw-admin user info --uid=npm

```

### rados

rados - rados object storage utility

### s3cmd

多看man手册

```
# 配置s3cmd
s3cmd --configure

```
