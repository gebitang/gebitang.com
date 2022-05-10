+++
title = "ElasticSearch And Kibana"
description = "the way to log k8s"
tags = [
    "elasticsearch",
    "kibana"
]
date = "2022-05-09"
topics = [
    "elasticsearch",
    "kibana"
]
draft = false
toc = true
+++

[ElasticSearch 7.8.1集群搭建](https://www.cnblogs.com/chenyanbin/p/13493920.html)

## FileBeat 

从6.3开始[官方：prospectors重命名为inputs](https://www.elastic.co/blog/brewing-in-beats-rename-filebeat-prospectors-to-inputs)，但保留了兼容性：`filebeat.prospectors`依然有效

### 日志目录

- 目录`/val/log/containers`下的日志是软连接，命名格式为：`<pod-name>_<namespace>_<containername>_<containerID>.log`
- 指向地址为 `/val/log/pods/`目录下对应的`<namespace>_<pod-name>_<pod_uid>/<containername>/<restart_time>.log`，这依然是一个软连接地址
- 指向最终的log地址：`/var/lib/docker/containers/`目录下对应的`<containerID>/<containerID>-json.log`

日志格式每一行都被重新处理为json格式。

```shell
[root@v08 /var/lib/docker]#  ls -l /var/log/containers/bootcase-java-default-c8bd9d766-2tmvf_harbor_cname-075714f528415bb7c607ad781dd7dff20366520a16d5a165a6e30f8f55dc88fa.log
lrwxrwxrwx 1 root root 107 May  9 15:54 /var/log/containers/bootcase-java-default-c8bd9d766-2tmvf_harbor_cname-075714f528415bb7c607ad781dd7dff20366520a16d5a165a6e30f8f55dc88fa.log -> /var/log/pods/harbor_bootcase-java-default-c8bd9d766-2tmvf_a4aea2ed-31ee-4356-bd4b-89c84c45063c/cname/0.log


[root@v08 /var/lib/docker]# ls -l /var/log/pods/harbor_bootcase-java-default-c8bd9d766-2tmvf_a4aea2ed-31ee-4356-bd4b-89c84c45063c/cname/0.log
lrwxrwxrwx 1 root root 165 May  9 15:54 /var/log/pods/harbor_bootcase-java-default-c8bd9d766-2tmvf_a4aea2ed-31ee-4356-bd4b-89c84c45063c/cname/0.log -> /var/lib/docker/containers/075714f528415bb7c607ad781dd7dff20366520a16d5a165a6e30f8f55dc88fa/075714f528415bb7c607ad781dd7dff20366520a16d5a165a6e30f8f55dc88fa-json.log

```

条件：做了对应的映射
```
"Source": "/var/lib/kubelet/pods/e2a9e4c7-db27-40c9-9750-7e64406e3485/volume-subpaths/med-log/module-1/7", # 重启次数
"Destination": "/med/log",
```
deploy时做了映射关系——
```yaml
- mountPath: /med/log
  name: med-log
  subPathExpr: $(POD_NAME)/module-name

- hostPath:
  path: /var/log/k8s/harbor/bootcase
  type: DirectoryOrCreate
  name: med-log
```

如果应用将日志保存着 `/med/log/apps/xxx.log`之后，走node节点的`/var/log/k8s`目录下按照类似`harbor/bootcase/bootcase-java-default-7dc8878f64-bcn4s/module-1/apps/`路径存储log文件的