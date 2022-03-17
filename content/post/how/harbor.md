+++
title = "harbor basic"
description = "harbor"
tags = [
    "harbor"
]
date = "2022-03-17"
topics = [
    "harbor"
]
draft = false
toc = true
+++

## 云原生制品那些事(artifact)

[Harbor权威指南：容器镜像、Helm Chart等云原生制品的管理与实践](https://book.douban.com/subject/35220390/)，2020年11月出版

- 第一篇：容器镜像的结构(=1.4.1~1.4.4)
- 第二篇：OCI 镜像规范(=1.4.5)
- 第三篇：OCI 制品(=1.5.2~1.5.3)
- 第四篇：Registry 的作用原理(=1.6.1)

- [云原生制品那些事(1)：容器镜像](https://mp.weixin.qq.com/s/ScEw2-y7Gea01pEekYXkgw)
- [云原生制品那些事(2)：OCI 镜像规范](https://mp.weixin.qq.com/s/TVIz8p8nOj4ffYaECW7gVg)
- [云原生制品那些事(3)：OCI 制品Artifact](https://mp.weixin.qq.com/s/okFbunMWFLWfjwCjFbceZw)
- [云原生制品那些事(4)：Registry作用原理](https://mp.weixin.qq.com/s/r3xNEw6tT34TB_1Qr_z9lQ)

OCI 定义的镜像包括4个部分—— 
- 镜像索引（Image Index）：可选内容，指向镜像不同平台的版本，如 i386 和 arm64v8、Linux 和Windows
- 清单（Manifest）：说明镜像包含的配置和内容的文件
- 配置（Configuration）：描述容器的根文件系统和容器运行时使用的执行参数，还有一些镜像的元数据。
- 层文件（Layers）： 镜像的根文件系统由多个层文件叠加而成。每个层文件在分发时都必须被打包成一个tar文件，可选择压缩或者非压缩的方式，压缩工具可以是gzip或者zstd。



```
# docker manifest inspect hello-world
{
	"schemaVersion": 2,
	"mediaType": "application/vnd.docker.distribution.manifest.v2+json",
	"config": {
			"mediaType": "application/vnd.docker.container.image.v1+json",
			"size": 1520,
			"digest": "sha256:1815c82652c03bfd8644afda26fb184f2ed891d921b20a0703b46768f9755c57"
	},
	"layers": [
			{
				"mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
				"size": 972,
				"digest": "sha256:b04784fba78d739b526e27edc02a5a8cd07b1052e9283f5fc155828f4b614c28"
			}
	]
}

# Image Index example
{

  "schemaVersion": 2,
  "manifests": [
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "size": 8342,
      "digest": "sha256:d81ae89b30523f5152fe646c1f9d178e5d10f28d00b70294fca965b7b96aa3db",
      "platform": {
        "architecture": "arm64v8",
        "os": "linux"
      }
    },
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "size": 6439,
      "digest": "sha256:2ef4e3904905353a0c4544913bc0caa48d95b746ef1f2fe9b7c85b3badff987e",
      "platform": {
        "architecture": "amd64",
        "os": "linux"
      }
    }
  ],
  "annotations": {
    "io.harbor.key1": "value1",
    "io.harbor.key2": "value2"
  }
}
# annotations 键值对形式的附加信息（可选项）。
```
## harbor系统
### 前世今生

Harbor Registry（又称 Harbor 云原生制品仓库或 Harbor 镜像仓库）由 VMware 公司中国研发中心云原生实验室原创，并于 2016 年 3 月开源。确切说是中心的[ATC团队(Advanced Technology Center)](https://mp.weixin.qq.com/s/26HiJb300yHFAbrLxBo6EQ)发起，由张海宁带队发起。满足核心功能MVP：镜像pull && push。然后添加了四个核心功能——

- 1）RBAC ，支持 LDAP/AD 认证
- 2）日志审计 （操作可追溯性）
- 3）镜像复制（多数据中心或云环境之间的镜像自动同步）
- 4）图形化管理界面（几乎是企业应用必备）

看起来核心功能完全可以满足我们目前的需求，不需要升级到2.0版本哈。Docker依赖的基础技术早就存在，Docker的成功来自Docker Image的成功——成为事实标准之后，发展出后来的OPI。而Docker Engine大家都可以进行替代。

### 架构1.0 

- 2017年4月发布1.1版本
- 2019年5月Harbor 1.8发布(巴塞罗那举行的KubeCon欧洲大会上的主题演讲上)： OIDC  SSO 认证，异构镜像仓库复制，CI/CD 自动化集成和健康监控
- 2019年10月Harbor 1.9发布：tag 保留和配额、可与 CI/CD 工具集成的 Webhook 通知、数据复制、Syslog 集成以及 CVE 例外策略等安全功能

![](https://ivanzz1001.github.io/records/assets/img/docker/docker_harbor_arch.png)

Harbor在架构上主要由五个组件构成：

- **Proxy：** Harbor的registry, UI, token等服务，通过一个前置的反向代理统一接收浏览器、Docker客户端的请求，并将请求转发给后端不同的服务。

- **Registry：** 负责储存Docker镜像，并处理dockerpush/pull 命令。由于我们要对用户进行访问控制，即不同用户对Docker image有不同的读写权限，Registry会指向一个token服务，强制用户的每次docker pull/push请求都要携带一个合法的token,Registry会通过公钥对token 进行解密验证。——使用S3代替本地存储的镜像内容

- **Core services：** 这是Harbor的核心功能，主要提供以下服务：

  - UI：提供图形化界面，帮助用户管理registry上的镜像（image）, 并对用户进行授权。
  - webhook：为了及时获取registry 上image状态变化的情况， 在Registry上配置webhook，把状态变化传递给UI模块。
  - token服务：负责根据用户权限给每个docker push/pull命令签发token.Docker 客户端向Regiøstry服务发起的请求,如果不包含token，会被重定向到这里，获得token后再重新向Registry进行请求。
  
- **Database：** 为coreservices提供数据库服务，负责储存用户权限、审计日志、Docker image分组信息等数据。
- **Job services:** 主要用于镜像复制，本地镜像可以被同步到远程Harbor实例上。
- **Log collector：** 为了帮助监控Harbor运行，负责收集其他组件的log，供日后进行分析。

### 架构2.0

- 2020年5月发布2.0版本，Harbor 2.0 成为符合 OCI（Open Container Initiatives）规范的开源镜像仓库，能够存储多种云原生工件（Artifacts），例如，容器镜像、Helm Chart、OPA、Singularity 等等。

![](https://static001.geekbang.org/infoq/2c/2c739f94956f8c792607ba0438e31f74.png)

- **代理层：** 代理层实质上是一个 Nginx 反向代理，负责接收不同类型的客户端请求，包括浏览器、用户脚本、Docker 等，并根据请求类型和 URI 转发给不同的后端服务进行处理。

- **功能层：**
  - Portal：是一个基于 Argular 的前端应用，提供 Harbor 用户访问的界面。
  - Core：是 Harbor 中的核心组件，封装了 Harbor 绝大部分的业务逻辑。
  - JobService：异步任务组件，负责 Harbor 中很多比较耗时的功能，比如 Artifact 复制、扫描、垃圾回收等。
  - Docker Distribution：Harbor 通过 Distribution 实现 Artifact 的读写和存取等功能。
  - RegistryCtl：Docker Distribution 的控制组件。
  - Notary（可选）：基于 TUF 提供镜像签名管理的功能。
  - 扫描工具（可选）：镜像的漏洞检测工具。
  - ChartMuseum（可选）：提供 API 管理非 OCI 规范的 Helm Chart，随着兼容 OCI(Open Container Initiative) 规范的 Helm Chart 在社区上被更广泛地接受，Helm Chart 能以 Artifact 的形式在 Harbor 中存储和管理，不再依赖 ChartMuseum，因此 Harbor 可能会在后续版本中移除对 ChartMuseum 的支持。——最新版本应该已经去除ChartMuseum，都已OCI方式进行管理

- **数据层：**
  - Redis：主要作为缓存服务存储一些生命周期较短的数据，同时对于 JobService 还提供了类似队列的功能。
  - PostgreSQL：存储 Harbor 的应用数据，比如项目信息、用户与项目的关系、管理策略、配置信息、Artifact 的元数据等等。
  - Artifact 存储：存储 Artifact 本身的内容，也就是每次推送镜像、Helm Chart 或其他 Artifact 时，数据最终存储的地方。默认情况下，Harbor 会把 Artifact 写入本地文件系统中。用户也可以修改配置，将 Artifact 存储在外部存储中，例如亚马逊的对象存储 S3、谷歌云存储 GCS、阿里云的对象存储 OSS 等等。

更新点：
1. 符合 OCI（Open Container Initiatives）规范的开源镜像仓库
2. Aqua Trivy 作为默认漏洞扫描器 
3. 机器人帐户功能（robot account）

2020年9月发布2.1版本更新：  
- 镜像的代理和缓存
- 非阻塞的镜像垃圾回收
- P2P镜像分发的预热
- 机器学习模型的管理
- Sysdig镜像扫描器

2021年3月发布了 v2.2 版本  
- 系统级（跨项目）机器人帐号
- Prometheus 的支持
- 镜像的代理和缓存支持更多的公有云Registry，包括 AWS 的 ECR，谷歌云的GCR，Azure的 ACR 以及 Quay，避免 Docker Hub 的流量限制
- OIDC 认证支持管理组 （与 LDAP 类似）
- Aqua CSP 企业级扫描器集成
- Dell EMC ECS S3 存储支持

{{< fluid_imgs
  "pure-u-1-1|https://raw.githubusercontent.com/goharbor/harbor/release-2.0.0/docs/img/architecture/architecture.png|harbor architecture|https://github.com/goharbor/harbor/wiki/Architecture-Overview-of-Harbor"
>}}


### 升级

[生产系统中升级 Harbor 的完整流程](https://mp.weixin.qq.com/s/dVCxvJRmOs47y0QaxkcGzQ)。如果要升级到2.x版本，需要先从1.x升级到

>Harbor 2.0 中只支持以 1.9.x 或 1.10.x 两个版本为起点的升级迁移，所以需要先升级到这两个版本

目前环境使用的至少harbor的核心功能，push&&pull。其他类似镜像复制、漏洞扫描、镜像签名、角色控制、垃圾清理等功能都没被用到。暂时还没啥直接升级的必要。需要的话，直接迁移使用2.x版本是更好的选择。

### 其他

-  关于[OCI](https://opencontainers.org/)： 成立于2015年，Linux 基金会旗下的合作项目，以开放治理的方式制定操作系统虚拟化（特别是 Linux 容器）的开放工业标准。制定：容器运行时（runtime）规范、容器镜像(image)规范、以及处于讨论中的镜像分发（distribution）规范

- [OPA](https://www.openpolicyagent.org/)：aka OPENPOLICYAGENT, Policy-based control for cloud native environments。The Open Policy Agent (OPA, pronounced “oh-pa”) is an open source, general-purpose policy engine that unifies policy enforcement across the stack. 

- [Singularity](http://apptainer.org/): Apptainer, the container system for high-performance computing (HPC) formerly known as Singularity, has joined the Linux Foundation。变化真快哈~ 
还有一个[sylabs](https://github.com/sylabs/singularity)公司，专门做Singularity相关服务，类似red hat vs. linux??

### 参考资料

- [Harbor创始人Henry](https://mp.weixin.qq.com/s/XPDaI9XXnTH9kYv4dCTLwA) 
- [巨轮起航，心怀诗和远方的港口--为什么做Harbor开源企业级Registry？](https://mp.weixin.qq.com/s/26HiJb300yHFAbrLxBo6EQ)
- [Harbor：开源企业级容器Registry架构简介](https://mp.weixin.qq.com/s/aGRMZ5W_eUEJ9UF2XhfASw)
- [Harbor开源镜像仓库的设计理念](https://mp.weixin.qq.com/s/8KEHnd4LER-M9VhVQ478UA)
- [Harbor 1.8开拓镜像管理崭新应用场景](https://mp.weixin.qq.com/s/BEpfR0L2n1f5SdlRqgEWTA)
- [Harbor 1.9 新增多项企业级功能](https://mp.weixin.qq.com/s/qS_ehznkkNDBlt_tqOQrGA)
- [Harbor 2.0的飞跃: OCI 兼容的工件仓库](https://mp.weixin.qq.com/s/fJMbr3TQOctVdg8Y9PyALA)
- [Harbor整体架构](https://ivanzz1001.github.io/records/post/docker/2018/04/20/docker-harbor-architecture)
- [Habor2.x 入门指南](https://xie.infoq.cn/article/c54645b4c5f6857b66beec488)
- [Harbor 2.1发布，工程师的发际线有救了](https://mp.weixin.qq.com/s/BYhSyVeVROYMwIAdO3SbmA)
- [Apptainer, a Container System for High-Performance Computing](https://thenewstack.io/apptainer-a-container-system-for-high-performance-computing/)
- [Architecture Overview of Harbor](https://github.com/goharbor/harbor/wiki/Architecture-Overview-of-Harbor)