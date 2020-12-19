+++
title = "Service Mesh"
description = "Leaning about service mesh"
tags = [
    "mesh"
]
date = "2020-12-19"
topics = [
    "mesh",
    "tech"
]
toc=true
+++

## Basic info 

微服务下一步，在服务层再做一层抽象，处理服务之间的通信、认证、流量控制，blablabla~ 关键词：**SideCar**

- [What Is a Service Mesh? -- by Nignx](https://www.nginx.com/blog/what-is-a-service-mesh/)
- [What's a service mesh? --by Red Hat](https://www.redhat.com/en/topics/microservices/what-is-a-service-mesh)
- [Service Mesh Ultimate Guide: Managing Service-to-Service Communications in the Era of Microservices -- by InfoQ](https://www.infoq.com/articles/service-mesh-ultimate-guide/)

>A **service mesh** is a configurable, low‑latency infrastructure layer designed to handle a high volume of network‑based interprocess communication among application infrastructure services using application programming interfaces (APIs).

![](https://upload-images.jianshu.io/upload_images/3296949-188781ea7b951e83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/3296949-8289c1faf36275af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所谓提供L7层的服务路由(处理service之间的通信，自然需要到7层) 

![](https://upload-images.jianshu.io/upload_images/3296949-417afc9a0aa38bd8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


目前的实现——

*   [Linkerd](https://linkerd.io/)
*   [Istio](https://istio.io/)
*   [Consul](https://www.consul.io/)
*   [Kuma](https://kuma.io/)
*   [Maesh](https://containo.us/maesh/)
*   [AWS App Mesh](https://aws.amazon.com/app-mesh/)

[Kubernetes Service Mesh: A Comparison of Istio, Linkerd and Consul](https://platform9.com/blog/kubernetes-service-mesh-a-comparison-of-istio-linkerd-and-consul/) 

### Octo, Envoy, CAP 

[美团服务治理 Octo](https://tech.meituan.com/tags/octo.html) 
>美团采用数据面基于 Envoy 二次开发、控制面自研为主、SDK 协同升级的方案（内部项目名是 OCTO Mesh ）

控制面 VS 数据面 

[What is Envoy](https://www.envoyproxy.io/docs/envoy/latest/intro/what_is_envoy)
>Envoy is an L7 proxy and communication bus designed for large modern service oriented architectures. The project was born out of the belief that:
>>The network should be transparent to applications. When network and application problems do occur it should be easy to determine the source of the problem.

[xDS](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/operations/dynamic_configuration)  “某某服务发现”的意思
>These APIs are collectively known as [“xDS”](https://www.envoyproxy.io/docs/envoy/latest/api-docs/xds_protocol#xds-protocol) (* discovery service).


CP系统 VS. AP系统 来自于 CAP的概念——[（译）请不要再称数据库为 CP 和 AP](https://zhuanlan.zhihu.com/p/108002021)，[英文原文](https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html)

*   *一致性（Consistency）* 在 CAP 中是[可线性化](http://cs.brown.edu/~mph/HerlihyW90/p463-herlihy.pdf)的意思（linearizability）。这是一个非常特殊（而且非常强）的一致性。需要说明的是，虽然 ACID 中的 C 也是一致性（Consistency），但它和这里的一致性没有任何关系，我会在后面解释可线性化是什么意思。
*   *可用性（Availability）*在CAP中被定义为"**所有**工作中的[数据库]节点对于每一个收到的请求，都必须返回一个[非错误]的响应（response）"。只有部分节点能处理请求是不够的：所有工作中的节点都必须能正确处理请求。所以很多自称"高度可用"（如，系统总体很少宕机）的系统其实并没有满足这里的可用性的定义。
*   *分区容错（Partition Tolerance，这个名字误导性很强）* 基本上是说你在一个消息可能会被延迟发送或者干脆丢失的[异步网络](http://www.the-paper-trail.org/page/cap-faq/)中通信。互联网和数据中心都有这个[特性](https://aphyr.com/posts/288-the-network-is-reliable)，所以在这件事上我们没得选。

一致性漫画一则——[来源](https://www.zipcomic.com/invincible-2018-issue-1)

![](https://upload-images.jianshu.io/upload_images/3296949-51ea7c1ac34e6583.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### IaaS, PaaS, Saas

[SaaS vs PaaS vs IaaS: What’s The Difference & How To Choose](https://www.bmc.com/blogs/saas-vs-paas-vs-iaas-whats-the-difference-and-how-to-choose)

*   [Software as a Service (SaaS)](https://www.bmc.com/blogs/saas-vs-paas-vs-iaas-whats-the-difference-and-how-to-choose/#ref1)
*   [Platform as a Service (PaaS)](https://www.bmc.com/blogs/saas-vs-paas-vs-iaas-whats-the-difference-and-how-to-choose/#ref2)
*   [Infrastructure as a Service (IaaS)](https://www.bmc.com/blogs/saas-vs-paas-vs-iaas-whats-the-difference-and-how-to-choose/#ref3)

|Platform Type|Common Examples|
|---------|---------|
|SaaS|Google Workspace, Dropbox, Salesforce, Cisco WebEx, Concur, GoToMeeting|
|PaaS|AWS Elastic Beanstalk, Windows Azure, Heroku, Force.com, Google App Engine, Apache Stratos, OpenShift|
|IaaS|DigitalOcean, Linode, Rackspace, Amazon Web Services (AWS), Cisco Metapod, Microsoft Azure, Google Compute Engine (GCE)|

从以下维度有谁负责管理的角度区分这几个名词——

- Applications 应用
- Virtualization 可视化
- Servers 服务端
- Data 数据
- Storage 存储
- Runtime 运行时
- Middleware 中间件
- Networking 网络
- O/S 操作系统 

**On-Premises 私有化部署**
![On Premise](https://upload-images.jianshu.io/upload_images/3296949-192ed35452b9f4c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**IaaS**
![IaaS](https://upload-images.jianshu.io/upload_images/3296949-ab15834110de8e53.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**PasS**
![Paas](https://upload-images.jianshu.io/upload_images/3296949-1ae4ed670de5881b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**SasS**

![SaaS](https://upload-images.jianshu.io/upload_images/3296949-1a939847913ab6ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Network 

[二层网络和三层网络 ](https://www.sohu.com/a/297996753_100169323)

二层网络：交换机直接与终端通信。

>交换机根据MAC地址表进行数据包的转发。
>
>有则转发，无则泛洪，即将数据包广播发送到所有端口，如果目的终端收到给出回应，那么交换机就可以将该MAC地址添加到地址表中，这是交换机对MAC地址进行建立的过程。

扩展二层网络——

三层网络：加快大型局域网内部的数据交换。

包括：核心层、汇聚层和接入层

核心层：对接外部网络
汇聚层：实施安全功能（划分VLAN和配置ACL）、工作组整体接入功能、虚拟网络过滤功能。因此，汇聚层设备应采用三层交换机。
接入层：对接具体接入的终端设备

[两层网络、三层网络的理解](https://blog.csdn.net/cj2580/article/details/80107037) 实例演示

![](https://upload-images.jianshu.io/upload_images/3296949-8df98da14b1fc498.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 二层交换机：只识别MAC地址，不识别IP地址（IP地址由电脑负责转换），不能路由，所以叫二层交换机
- 三层交换机：不但能识别MAC地址，还能把MAC地址中的IP地址识别出来，进行路由。

实际做法：  
- 处于同一个局域网中的各个子网的互联以及局域网中VLAN间的路由，用三层交换机来代替路由器
- 局域网与公网互联之间要实现跨地域的网络访问时，使用专业路由器
