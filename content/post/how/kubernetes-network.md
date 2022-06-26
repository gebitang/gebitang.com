+++
title = "kubernetes network"
description = "kube-proxy, Calico, coreDNS, etc..."
tags = [
    "calico",
    "kube-proxy"
]
date = "2022-05-12"
topics = [
    "calico",
    "kube-proxy"
]
draft = false
toc = true
+++

多看man手册，多看man手册，多看man手册。  

- [netfilter/iptables 简介](https://www.cnblogs.com/sparkdev/p/9328713.html) + [IPVS 介绍](https://www.cnblogs.com/huanggze/p/12486158.html) 简单两篇入门
- [IPVS从入门到精通kube-proxy实现原理](https://mp.weixin.qq.com/s/QTNlkgde2UI5ABx3tw6ndw)——这篇写得很清楚，包含了iptables和ipvs的内容
- [利用 kubeadm 创建高可用集群](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/high-availability/) ——官方文档
- [k8s官方文档：设置kube-proxy的模式](https://kubernetes.io/zh/docs/reference/config-api/kube-proxy-config.v1alpha1/#kubeproxy-config-k8s-io-v1alpha1-ProxyMode)
- [kubernetes/pkg/proxy/ipvs/README.md](https://github.com/kubernetes/kubernetes/blob/master/pkg/proxy/ipvs/README.md)

- [Kubernetes 集群部署 之 多Master节点 实现高可用](https://blog.csdn.net/duanbaoke/article/details/119667328)——备用，还未实践
- [办公环境下 kubernetes 网络互通方案](https://www.qikqiak.com/post/office-env-k8s-network/) —— 学习参考


- [iptables详解及一些常用规则](https://www.jianshu.com/p/ee4ee15d3658)：表、链、hook函数点——入门学习用，学完就忘
- [LVS+Iptables实现FULLNAT及原理分析](https://blog.dianduidian.com/post/lvs-snat%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90/)——验证说明LVS实现FULLNAT的方式 `启用内核一个参数，net.ipv4.vs.conntrack=1`
- [kube-proxy ipvs模式的conn_reuse_mode问题](https://maao.cloud/2021/01/15/%E6%B7%B1%E5%85%A5kube-proxy%20ipvs%E6%A8%A1%E5%BC%8F%E7%9A%84conn_reuse_mode%E9%97%AE%E9%A2%98/)——源码级别分析
- [IPVS: The Linux Load Balancer (Deep Dive)](https://debugged.it/blog/ipvs-the-linux-load-balancer/)

关于ipvs：  
- [ipvsadm - Linux Virtual Server administration](https://manpages.debian.org/testing/ipvsadm/ipvsadm.8.en.html)——包含10个完整的路由规则。
- [2013：[PATCH] ipvsadm: support for scheduler flags](https://archive.linuxvirtualserver.org/html/lvs-users/2013-06/msg00001.html)提到`flag-1,flag-2`只是为了【未来适用的】(future-proofing)，目前没实际效果。 
- [2019: [PATCH 3/3] ipvsadm: Add support for mh scheduler](https://archive.linuxvirtualserver.org/html/lvs-devel/2019-01/msg00001.html)开始正式支持mh(Maglev Hashing)算法的路由规则 
- 关于mh：[一致性哈希算法（四）- Maglev一致性哈希法](https://writings.sh/post/consistent-hashing-algorithms-part-4-maglev-consistent-hash)

关于keepalived：  
- [keepalived.conf - configuration file for Keepalived](https://man.archlinux.org/man/keepalived.conf.5.en)
- [LVS+Keepalived 搭建高可用群集（DR模式）](https://blog.csdn.net/duanbaoke/article/details/117997765) 

```shell
# sudo ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.15.64.3:53 mh (flag-1,flag-2)
  -> 10.19.0.38:53                Masq    1      0          0
  -> 10.19.1.170:53               Masq    1      0          0
```

### CoreDNS 

Kube-dns或CoreDNS，基本原理都是利用watch Kubernetes的Service和Pod，生成DNS记录，然后通过重新配置Kubelet的DNS选项让新启动的Pod使用Kube-dns或CoreDNS提供的Kubernetes集群内域名解析服务


