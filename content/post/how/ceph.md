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


```
# 查看bucket详细信息
radosgw-admin bucket stats --bucket=harbor-dev

# 查看pool副本数量
ceph osd pool get default.rgw.buckets.test-data size

#查看用户信息
radosgw-admin user info --uid=npm

# 查看集群信息
ceph df

# 配置s3cmd
s3cmd --configure
```
