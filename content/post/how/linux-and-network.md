+++
title = "linux and network"
description = "all about network"
tags = [
    "linux",
    "network"
]
date = "2022-05-15"
topics = [
    "network"
]
toc = true
+++

### socket信息查看

prometheus监控的socket相关数据包括—— `node_sockstat_TCP_alloc`，表示`Number of TCP sockets in state alloc`；已分配的socket数量。

查看`cat /proc/net/sockstat` 或者使用`ss -s`(ss优势是在大量链接时的速度优势)

```shell
# cat /proc/net/sockstat
sockets: used 185  
TCP: inuse 54 orphan 0 tw 0 alloc 155 mem 17349
UDP: inuse 5 mem 41
UDPLITE: inuse 0
RAW: inuse 0
FRAG: inuse 0 memory 0

# ss -s  (Socket Statistics)
Total: 184 (kernel 5651)
TCP:   198 (estab 53, closed 135, orphaned 0, synrecv 0, timewait 0/0), ports 0

Transport Total     IP        IPv6
*         5651      -         -
RAW       0         0         0
UDP       6         5         1
TCP       63        54        9
INET      69        59        10
FRAG      0         0         0
```

[字段解释](https://blog.csdn.net/zeweig/article/details/51760624)

- sockets: used：已使用的所有协议套接字总量
- TCP: inuse：正在使用（正在侦听）的TCP套接字数量。其值 `≤ netstat –lnt | grep ^tcp | wc –l`
- TCP: orphan：无主（不属于任何进程）的TCP连接数（无用、待销毁的TCP socket数）
- TCP: tw：等待关闭的TCP连接数。其值等于 `netstat –ant | grep TIME_WAIT | wc –l`
- TCP：alloc(allocated)：已分配（已建立、已申请到sk_buff）的TCP套接字数量。其值等于 `netstat –ant | grep ^tcp | wc –l`
- TCP：mem：套接字缓冲区使用量（单位不详。用scp实测，速度在4803.9kB/s时：其值=11，netstat –ant 中相应的22端口的Recv-Q＝0，Send-Q≈400）
- UDP：inuse：正在使用的UDP套接字数量
- RAW：
- FRAG：使用的IP段数量
