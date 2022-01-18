+++
title = "k8s practice"
description = "the way to kubernetes"
tags = [
    "kubernetes",
    "k8s"
]
date = "2020-08-23"
topics = [
    "kubernetes",
    "k8s"
]
draft = false
toc = true
+++

## Tasks

练习记录，[官方Tasks](https://kubernetes.io/docs/tasks/)，[中文版本对照](https://kubernetes.io/zh/docs/tasks/)跟着练习一遍之后，各个概念就清楚了。

### 更新DaemonSet 

[对 DaemonSet 执行滚动更新](https://kubernetes.io/zh/docs/tasks/manage-daemon/update-daemon-set/)，本地控制DO的集群，默认会超时，无法apply对象，提示

```
E1123 19:37:27.679680     511 request.go:1027] Unexpected error when reading response body: net/http: request canceled (Client.Timeout or context cancellation while reading body)
error: unexpected error when reading response body. Please retry. Original error: net/http: request canceled (Client.Timeout or context cancellation while reading body)
# 或者 
Unable to connect to the server: net/http: TLS handshake timeout
```

增加超时时间设置`k apply -f xxx.yaml  --request-timeout="50"`，还可以指定log级别`-v=6`，参考[【Kubernetes】Client.Timeout or context cancellation while reading bodyというエラーの解消方法](https://www.mtioutput.com/entry/kubectl-timeout-error)

- 设置DaemonSet内容，查看更新策略`kubectl get ds/fluentd-elasticsearch  -n kube-system -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}'` 查看方式也可以为 `-o yaml`
- 对 RollingUpdate DaemonSet 的 `.spec.template` 的任何更新都将触发滚动更新。例如增加了cpu和内存的限制

以下方式都可以更新——
```shell
# 声明式命令
kubectl apply -f https://k8s.io/examples/controllers/fluentd-daemonset-update.yaml
# 指令式命令 Edit a resource from the default editor.
kubectl edit ds/fluentd-elasticsearch -n kube-system #打开文件编辑，保持。远程不建议这样操作
# 只更新容器镜像
kubectl set image ds/fluentd-elasticsearch fluentd-elasticsearch=quay.io/fluentd_elasticsearch/fluentd:v2.6.0 -n kube-system
```

更新结果类似——
```
myu@Gebitang:~$ k apply -f fluentd-daemonset-update.yaml
daemonset.apps/fluentd-elasticsearch configured
myu@Gebitang:~$ kubectl rollout status ds/fluentd-elasticsearch -n kube-system
Waiting for daemon set "fluentd-elasticsearch" rollout to finish: 1 out of 3 new pods have been updated...
Waiting for daemon set "fluentd-elasticsearch" rollout to finish: 1 out of 3 new pods have been updated...
Waiting for daemon set "fluentd-elasticsearch" rollout to finish: 2 out of 3 new pods have been updated...
Waiting for daemon set "fluentd-elasticsearch" rollout to finish: 2 out of 3 new pods have been updated...
Waiting for daemon set "fluentd-elasticsearch" rollout to finish: 2 out of 3 new pods have been updated...
Waiting for daemon set "fluentd-elasticsearch" rollout to finish: 2 of 3 updated pods are available...
daemon set "fluentd-elasticsearch" successfully rolled out
```

从名字空间中删除 DaemonSet：`kubectl delete ds fluentd-elasticsearch -n kube-system`

### 管理内存、CPU、配额

[管理内存](https://kubernetes.io/zh/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/)，提示`looking up service account default-mem-example/default: serviceaccount "default" not found`，意味着没有服务账号(默认空间有默认的账号，但新创建的namespace下没有服务账户)，可以先执行任务[为pod配置服务账户](https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-service-account/)

```shell
kubectl create -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: default
  namespace: default-mem-example
EOF
# 然后再在对应空间下创建pod即可
```

执行删除namespace的命令后`k delete namespace default-mem-example`，一直没有返回结果，Ctrl+C强制结束，查看状态显示为`Terminating`，参考[Namespace "stuck" as Terminating, How I removed it](https://stackoverflow.com/questions/52369247/namespace-stuck-as-terminating-how-i-removed-it)，当前空间下还有其他资源？`kubectl api-resources --verbs=list --namespaced -o name  | xargs -n 1 kubectl get --show-kind --ignore-not-found -n default-mem-example`，删除对应的资源后，状态依然为`Terminating`

```shell
geb@Gebitang:~$ k delete limitranges --namespace=default-mem-example
error: resource(s) were provided, but no name was specified
geb@Gebitang:~$ k delete limitrange/mem-limit-range --namespace=default-mem-example
limitrange "mem-limit-range" deleted
geb@Gebitang:~$ k delete serviceaccount/default --namespace=default-mem-example
serviceaccount "default" deleted

# 查看空间信息 k get namespace default-mem-example -o json
```

```json
{
    "apiVersion": "v1",
    "kind": "Namespace",
    "metadata": {
        "creationTimestamp": "2021-11-23T07:17:26Z",
        "deletionTimestamp": "2021-11-23T07:55:36Z",
        "labels": {
            "kubernetes.io/metadata.name": "default-mem-example"
        },
        "name": "default-mem-example",
        "resourceVersion": "2161742",
        "uid": "7f0fd528-b681-45f7-887e-49820024333e"
    },
    "spec": {
        "finalizers": [
            "kubernetes"
        ]
    },
    "status": {
        "phase": "Terminating"
    }
}
```

调用api，将finalizers字段置空即可：

```shell
# 使用proxy，然后调用http server api
NAMESPACE=your-rogue-namespace
kubectl proxy &
kubectl get namespace $NAMESPACE -o json |jq '.spec = {"finalizers":[]}' >temp.json
curl -k -H "Content-Type: application/json" -X PUT --data-binary @temp.json 127.0.0.1:8001/api/v1/namespaces/$NAMESPACE/finalize

# 直接调用，将上述json内容的finalizers数组置为空，保存为 default-mem-example.json， 调用
 k replace --raw "/api/v1/namespaces/default-mem-example/finalize" -f default-mem-example.json
# 再查看空间时，此空间被彻底删除
```

其他资源类似


## start over 

### 虚拟机环境

- centOS7 http://isoredirect.centos.org/centos/7/isos/x86_64/
- debian11 https://www.debian.org/distrib/netinst
- ubuntu20.04 https://ubuntu.com/download/server

**其他事项** 
- 旧的vmware player无法支撑debian11版本，重新下载安装player 16，分别创建三个虚拟机系统
- 国内镜像源：阿里 https://developer.aliyun.com/mirror/， 清华 https://mirrors.tuna.tsinghua.edu.cn/， 163 http://mirrors.163.com/ ， 中科大 https://mirrors.ustc.edu.cn/
- 不同类型的镜像介绍： [Differences between distribution types](https://superuser.com/questions/968889/what-is-the-difference-between-live-bin-minimal-and-netinstall-versions-of-ce)
  - **Live** 类似可直接使用的USB系统
  - **Bin** 桌面版。包含了GUI环境和其他软件
  - **Minimal** 不包含GUI的版本
  - **Netinstall** 需要从镜像站下载安装的版本

纯手工搭建还是比较费劲，添加镜像源之后通过命令行搭建比较靠谱。

- [安装docker环境](https://docs.docker.com/engine/install/debian/)： 配置好源之后；安装工具；添加GPG key；添加stable源；安装8个包
- 安装三件套`kubeadm kubelet kubectl`
- 先确保kubelet[正常启动](https://stackoverflow.com/questions/52119985/kubeadm-init-shows-kubelet-isnt-running-or-healthy)

```
# create file /etc/docker/daemon.json with 
{
    "exec-opts": ["native.cgroupdriver=systemd"]
}
# do the following commands
 sudo systemctl daemon-reload
 sudo systemctl restart docker
 sudo systemctl restart kubelet
```

- 执行`sudo kubeadm init --v=5`开始安装
- 安装成功后，master节点会显示为NotReady状态，查看后因为没有CNI插件，可以安装[Deploying flannel manually](https://github.com/flannel-io/flannel#deploying-flannel-manually)，稍后恢复为ready状态


### 纯手工搭建环境

目前只有一台ubuntu 20.04的server机器。

先收工[安装docker环境](https://docs.docker.com/engine/install/ubuntu/)，从[下载页面](https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/)下载docker-ce docker-ce-cli containerd.io

**containerd.io** : daemon [containerd](https://containerd.io/). It [works](https://containerd.io/docs/getting-started/) independently on the docker packages, and it is required by the docker packages.

> containerd is available as a daemon for Linux and Windows. It manages the complete container lifecycle of its host system, from image transfer and storage to container execution and supervision to low-level storage to network attachments and beyond.

**docker-ce-cli** : command line interface for docker engine, community edition

**docker-ce** : [docker](https://docs.docker.com/get-docker/) engine, community edition. Requires docker-ce-cli.

>[containerd.io_1.4.12-1_amd64.deb](https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/containerd.io_1.4.12-1_amd64.deb) 2021-12-11 23:03:31 22.6 MiB  
>[docker-ce-cli_20.10.12~3-0~ubuntu-focal_amd64.deb](https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/docker-ce-cli_20.10.12~3-0~ubuntu-focal_amd64.deb)  2021-12-13 14:38:46 38.8 MiB  
>[docker-ce_20.10.12~3-0~ubuntu-focal_amd64.deb](https://download.docker.com/linux/ubuntu/dists/focal/pool/stable/amd64/docker-ce_20.10.12~3-0~ubuntu-focal_amd64.deb)  2021-12-13 14:38:48 20.3 MiB  

注意安装顺序——安装完成后会自动创建对应的systemd services

```
➜  deps sudo dpkg -i containerd.io_1.4.12-1_amd64.deb
[sudo] password for geb:
Selecting previously unselected package containerd.io.
(Reading database ... 109132 files and directories currently installed.)
Preparing to unpack containerd.io_1.4.12-1_amd64.deb ...
Unpacking containerd.io (1.4.12-1) ...
Setting up containerd.io (1.4.12-1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/containerd.service → /lib/systemd/system/containerd.service.
Processing triggers for man-db (2.9.1-1) ...
➜  deps sudo dpkg -i docker-ce_20.10.12_3-0_ubuntu-focal_amd64.deb
Selecting previously unselected package docker-ce.
(Reading database ... 109148 files and directories currently installed.)
Preparing to unpack docker-ce_20.10.12_3-0_ubuntu-focal_amd64.deb ...
Unpacking docker-ce (5:20.10.12~3-0~ubuntu-focal) ...
dpkg: dependency problems prevent configuration of docker-ce:
 docker-ce depends on docker-ce-cli; however:
  Package docker-ce-cli is not installed.

dpkg: error processing package docker-ce (--install):
 dependency problems - leaving unconfigured
Processing triggers for systemd (245.4-4ubuntu3.15) ...
Errors were encountered while processing:
 docker-ce
➜  deps sudo dpkg -i docker-ce-cli_20.10.12_3-0_ubuntu-focal_amd64.deb
Selecting previously unselected package docker-ce-cli.
(Reading database ... 109160 files and directories currently installed.)
Preparing to unpack docker-ce-cli_20.10.12_3-0_ubuntu-focal_amd64.deb ...
Unpacking docker-ce-cli (5:20.10.12~3-0~ubuntu-focal) ...
Setting up docker-ce-cli (5:20.10.12~3-0~ubuntu-focal) ...
Processing triggers for man-db (2.9.1-1) ...
➜  deps sudo dpkg -i docker-ce_20.10.12_3-0_ubuntu-focal_amd64.deb
(Reading database ... 109358 files and directories currently installed.)
Preparing to unpack docker-ce_20.10.12_3-0_ubuntu-focal_amd64.deb ...
Unpacking docker-ce (5:20.10.12~3-0~ubuntu-focal) over (5:20.10.12~3-0~ubuntu-focal) ...
Setting up docker-ce (5:20.10.12~3-0~ubuntu-focal) ...
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /lib/systemd/system/docker.service.
Created symlink /etc/systemd/system/sockets.target.wants/docker.socket → /lib/systemd/system/docker.socket.
Processing triggers for systemd (245.4-4ubuntu3.15) ...
```

配置安装源(本页面后续内容)后，[安装 kubeadm、kubelet 和 kubectl](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

- kubeadm：用来初始化集群的指令。
- kubelet：在集群中的每个节点上用来启动 Pod 和容器等。
- kubectl：用来与集群通信的命令行工具。

```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

#算上以来文件，实际上安装了8个应用
conntrack cri-tools ebtables kubeadm kubectl kubelet kubernetes-cni socat 
```

执行初始化动作`sudo kubeadm init --v=5`可以看到更详细的安装信息，根据log信息可以看到大致执行了哪些动作——

```
➜ sudo kubeadm init --v=5
I0116 11:17:10.928410   83074 initconfiguration.go:117] detected and using CRI socket: /var/run/dockershim.sock
I0116 11:17:10.928585   83074 interface.go:432] Looking for default routes with IPv4 addresses
I0116 11:17:10.928591   83074 interface.go:437] Default route transits interface "eno1"
I0116 11:17:10.928723   83074 interface.go:209] Interface eno1 is up
I0116 11:17:10.928767   83074 interface.go:257] Interface "eno1" has 4 addresses :[192.168.10.101/24 fd0d:7276:3cac::506/128 fd0d:7276:3cac:0:6202:92ff:fe39:bf83/64 fe80::6202:92ff:fe39:bf83/64].
I0116 11:17:10.928790   83074 interface.go:224] Checking addr  192.168.10.101/24.
I0116 11:17:10.928806   83074 interface.go:231] IP found 192.168.10.101
I0116 11:17:10.928828   83074 interface.go:263] Found valid IPv4 address 192.168.10.101 for interface "eno1".
I0116 11:17:10.928834   83074 interface.go:443] Found active IP 192.168.10.101
I0116 11:17:10.928859   83074 kubelet.go:217] the value of KubeletConfiguration.cgroupDriver is empty; setting it to "systemd"
I0116 11:17:10.931755   83074 version.go:186] fetching Kubernetes version from URL: https://dl.k8s.io/release/stable-1.txt
[init] Using Kubernetes version: v1.23.1
[preflight] Running pre-flight checks
I0116 11:17:11.640595   83074 checks.go:578] validating Kubernetes and kubeadm version
I0116 11:17:11.640657   83074 checks.go:171] validating if the firewall is enabled and active
I0116 11:17:11.654060   83074 checks.go:206] validating availability of port 6443
I0116 11:17:11.654160   83074 checks.go:206] validating availability of port 10259
I0116 11:17:11.654181   83074 checks.go:206] validating availability of port 10257
I0116 11:17:11.654200   83074 checks.go:283] validating the existence of file /etc/kubernetes/manifests/kube-apiserver.yaml
I0116 11:17:11.654217   83074 checks.go:283] validating the existence of file /etc/kubernetes/manifests/kube-controller-manager.yaml
I0116 11:17:11.654224   83074 checks.go:283] validating the existence of file /etc/kubernetes/manifests/kube-scheduler.yaml
I0116 11:17:11.654232   83074 checks.go:283] validating the existence of file /etc/kubernetes/manifests/etcd.yaml
I0116 11:17:11.654239   83074 checks.go:433] validating if the connectivity type is via proxy or direct
I0116 11:17:11.654253   83074 checks.go:472] validating http connectivity to first IP address in the CIDR
I0116 11:17:11.654266   83074 checks.go:472] validating http connectivity to first IP address in the CIDR
I0116 11:17:11.654271   83074 checks.go:107] validating the container runtime
I0116 11:17:11.722204   83074 checks.go:133] validating if the "docker" service is enabled and active
I0116 11:17:11.772580   83074 checks.go:332] validating the contents of file /proc/sys/net/bridge/bridge-nf-call-iptables
I0116 11:17:11.772659   83074 checks.go:332] validating the contents of file /proc/sys/net/ipv4/ip_forward
I0116 11:17:11.772693   83074 checks.go:654] validating whether swap is enabled or not
	[WARNING Swap]: swap is enabled; production deployments should disable swap unless testing the NodeSwap feature gate of the kubelet
I0116 11:17:11.772789   83074 checks.go:373] validating the presence of executable conntrack
I0116 11:17:11.772831   83074 checks.go:373] validating the presence of executable ip
I0116 11:17:11.772854   83074 checks.go:373] validating the presence of executable iptables
I0116 11:17:11.772877   83074 checks.go:373] validating the presence of executable mount
I0116 11:17:11.772911   83074 checks.go:373] validating the presence of executable nsenter
I0116 11:17:11.772960   83074 checks.go:373] validating the presence of executable ebtables
I0116 11:17:11.772987   83074 checks.go:373] validating the presence of executable ethtool
I0116 11:17:11.773018   83074 checks.go:373] validating the presence of executable socat
I0116 11:17:11.773048   83074 checks.go:373] validating the presence of executable tc
I0116 11:17:11.773077   83074 checks.go:373] validating the presence of executable touch
I0116 11:17:11.773100   83074 checks.go:521] running all checks
I0116 11:17:11.856347   83074 checks.go:404] checking whether the given node name is valid and reachable using net.LookupHost
I0116 11:17:11.856366   83074 checks.go:620] validating kubelet version
I0116 11:17:11.900767   83074 checks.go:133] validating if the "kubelet" service is enabled and active
I0116 11:17:11.919129   83074 checks.go:206] validating availability of port 10250
I0116 11:17:11.919197   83074 checks.go:206] validating availability of port 2379
I0116 11:17:11.919230   83074 checks.go:206] validating availability of port 2380
I0116 11:17:11.919259   83074 checks.go:246] validating the existence and emptiness of directory /var/lib/etcd
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
I0116 11:17:11.919362   83074 checks.go:842] using image pull policy: IfNotPresent
I0116 11:17:11.956674   83074 checks.go:859] pulling: k8s.gcr.io/kube-apiserver:v1.23.1
I0116 11:18:27.630052   83074 checks.go:859] pulling: k8s.gcr.io/kube-controller-manager:v1.23.1
I0116 11:19:43.197056   83074 checks.go:859] pulling: k8s.gcr.io/kube-scheduler:v1.23.1
I0116 11:20:58.742768   83074 checks.go:859] pulling: k8s.gcr.io/kube-proxy:v1.23.1
I0116 11:22:14.285969   83074 checks.go:859] pulling: k8s.gcr.io/pause:3.6
I0116 11:23:29.802417   83074 checks.go:859] pulling: k8s.gcr.io/etcd:3.5.1-0
I0116 11:24:45.383728   83074 checks.go:859] pulling: k8s.gcr.io/coredns/coredns:v1.8.6

由于上走的gcr（Google Cloud Container Registry)，所以后面就超时失败了


```

下载clash，`gunzip clashxx.gz`解压；下载配置文件；[下载mmdb](https://github.com/Dreamacro/clash/issues/854)；启动` ./clash -d conf -f clash.yml`走代理下载镜像；关闭swap `sudo swapoff -a` 还是提示超时。[参考](https://zhuanlan.zhihu.com/p/46341911)

执行`kubeadm config images list`获取需要安装的镜像列表，然后从国内站点拉取。更方便的方式是init时指定repository的地址`sudo kubeadm init --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers`，更多[参数参考](https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/kubeadm-init/) 

- --pod-network-cidr string， 指明 pod 网络可以使用的 IP 地址段。如果设置了这个参数，控制平面将会为每一个节点自动分配 CIDRs。
- --service-cidr string     默认值："10.96.0.0/12"。为服务的虚拟 IP 地址另外指定 IP 地址段

[关于CIDR](https://www.jianshu.com/p/71220cbcf86e)——Classless Inter-Domain Routing, 无类域间路由。基于变长子网掩码([VLSM](https://www.techtarget.com/searchnetworking/definition/variable-length-subnet-mask))，举例，IP4由32位组成，则`192.255.255.255/12`前12位是地址的网络部分，而最后20位是主机地址。[计算器](https://www.ipaddressguide.com/cidr)可计算合适的ip范围


```
#!/bin/bash

images=(  # 下面的镜像应该去除"k8s.gcr.io/"的前缀，版本换成上面获取到的版本
    kube-apiserver:v1.23.1
kube-controller-manager:v1.23.1
kube-scheduler:v1.23.1
kube-proxy:v1.23.1
pause:3.6
etcd:3.5.1-0
coredns:v1.8.6
)

for imageName in ${images[@]} ; do
sudo docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
   sudo  docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
   sudo  docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
done
```

下载完成后，还需要把coredns的tag重新命名一下：

```
#因为直接下载 coredns/coredns:v1.8.6时会找不到，需要下载coredns:v1.8.6，但kubeadm需要的是k8s.gcr.io/coredns/coredns:v1.8.6 
sudo docker tag k8s.gcr.io/coredns:v1.8.6 k8s.gcr.io/coredns/coredns:v1.8.6
sudo docker rmi k8s.gcr.io/coredns:v1.8.6
```

镜像OK，健康检查失败。的确是kubelet被退出。 

```
I0116 12:34:31.850670   95794 checks.go:851] image exists: k8s.gcr.io/kube-apiserver:v1.23.1
I0116 12:34:31.878709   95794 checks.go:851] image exists: k8s.gcr.io/kube-controller-manager:v1.23.1
I0116 12:34:31.905295   95794 checks.go:851] image exists: k8s.gcr.io/kube-scheduler:v1.23.1
I0116 12:34:31.933910   95794 checks.go:851] image exists: k8s.gcr.io/kube-proxy:v1.23.1
I0116 12:34:31.961095   95794 checks.go:851] image exists: k8s.gcr.io/pause:3.6
I0116 12:34:31.989404   95794 checks.go:851] image exists: k8s.gcr.io/etcd:3.5.1-0
I0116 12:34:32.015336   95794 checks.go:851] image exists: k8s.gcr.io/coredns/coredns:v1.8.6
[certs] Using certificateDir folder "/etc/kubernetes/pki"
I0116 12:34:32.015391   95794 certs.go:112] creating a new certificate authority for ca
[certs] Generating "ca" certificate and key
I0116 12:34:32.333727   95794 certs.go:522] validating certificate period for ca certificate
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local lee] and IPs [10.96.0.1 192.168.10.101]
[certs] Generating "apiserver-kubelet-client" certificate and key
I0116 12:34:32.801641   95794 certs.go:112] creating a new certificate authority for front-proxy-ca
[certs] Generating "front-proxy-ca" certificate and key
I0116 12:34:32.918912   95794 certs.go:522] validating certificate period for front-proxy-ca certificate
[certs] Generating "front-proxy-client" certificate and key
I0116 12:34:33.148388   95794 certs.go:112] creating a new certificate authority for etcd-ca
[certs] Generating "etcd/ca" certificate and key
I0116 12:34:33.468493   95794 certs.go:522] validating certificate period for etcd/ca certificate
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [lee localhost] and IPs [192.168.10.101 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [lee localhost] and IPs [192.168.10.101 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
I0116 12:34:34.288630   95794 certs.go:78] creating new public/private key files for signing service account users
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
I0116 12:34:34.899330   95794 kubeconfig.go:103] creating kubeconfig file for admin.conf
[kubeconfig] Writing "admin.conf" kubeconfig file
I0116 12:34:35.094669   95794 kubeconfig.go:103] creating kubeconfig file for kubelet.conf
[kubeconfig] Writing "kubelet.conf" kubeconfig file
I0116 12:34:35.256831   95794 kubeconfig.go:103] creating kubeconfig file for controller-manager.conf
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
I0116 12:34:35.520261   95794 kubeconfig.go:103] creating kubeconfig file for scheduler.conf
[kubeconfig] Writing "scheduler.conf" kubeconfig file
I0116 12:34:35.645722   95794 kubelet.go:65] Stopping the kubelet
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
I0116 12:34:36.020442   95794 manifests.go:99] [control-plane] getting StaticPodSpecs
I0116 12:34:36.020618   95794 certs.go:522] validating certificate period for CA certificate
I0116 12:34:36.020674   95794 manifests.go:125] [control-plane] adding volume "ca-certs" for component "kube-apiserver"
I0116 12:34:36.020680   95794 manifests.go:125] [control-plane] adding volume "etc-ca-certificates" for component "kube-apiserver"
I0116 12:34:36.020685   95794 manifests.go:125] [control-plane] adding volume "etc-pki" for component "kube-apiserver"
I0116 12:34:36.020690   95794 manifests.go:125] [control-plane] adding volume "k8s-certs" for component "kube-apiserver"
I0116 12:34:36.020694   95794 manifests.go:125] [control-plane] adding volume "usr-local-share-ca-certificates" for component "kube-apiserver"
I0116 12:34:36.020699   95794 manifests.go:125] [control-plane] adding volume "usr-share-ca-certificates" for component "kube-apiserver"
I0116 12:34:36.024333   95794 manifests.go:154] [control-plane] wrote static Pod manifest for component "kube-apiserver" to "/etc/kubernetes/manifests/kube-apiserver.yaml"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
I0116 12:34:36.024368   95794 manifests.go:99] [control-plane] getting StaticPodSpecs
I0116 12:34:36.024548   95794 manifests.go:125] [control-plane] adding volume "ca-certs" for component "kube-controller-manager"
I0116 12:34:36.024562   95794 manifests.go:125] [control-plane] adding volume "etc-ca-certificates" for component "kube-controller-manager"
I0116 12:34:36.024570   95794 manifests.go:125] [control-plane] adding volume "etc-pki" for component "kube-controller-manager"
I0116 12:34:36.024578   95794 manifests.go:125] [control-plane] adding volume "flexvolume-dir" for component "kube-controller-manager"
I0116 12:34:36.024585   95794 manifests.go:125] [control-plane] adding volume "k8s-certs" for component "kube-controller-manager"
I0116 12:34:36.024593   95794 manifests.go:125] [control-plane] adding volume "kubeconfig" for component "kube-controller-manager"
I0116 12:34:36.024600   95794 manifests.go:125] [control-plane] adding volume "usr-local-share-ca-certificates" for component "kube-controller-manager"
I0116 12:34:36.024608   95794 manifests.go:125] [control-plane] adding volume "usr-share-ca-certificates" for component "kube-controller-manager"
I0116 12:34:36.025297   95794 manifests.go:154] [control-plane] wrote static Pod manifest for component "kube-controller-manager" to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
[control-plane] Creating static Pod manifest for "kube-scheduler"
I0116 12:34:36.025321   95794 manifests.go:99] [control-plane] getting StaticPodSpecs
I0116 12:34:36.025475   95794 manifests.go:125] [control-plane] adding volume "kubeconfig" for component "kube-scheduler"
I0116 12:34:36.025888   95794 manifests.go:154] [control-plane] wrote static Pod manifest for component "kube-scheduler" to "/etc/kubernetes/manifests/kube-scheduler.yaml"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
I0116 12:34:36.026504   95794 local.go:65] [etcd] wrote Static Pod manifest for a local etcd member to "/etc/kubernetes/manifests/etcd.yaml"
I0116 12:34:36.026515   95794 waitcontrolplane.go:91] [wait-control-plane] Waiting for the API server to be healthy
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[kubelet-check] Initial timeout of 40s passed.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.

	Unfortunately, an error has occurred:
		timed out waiting for the condition

	This error is likely caused by:
		- The kubelet is not running
		- The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

	If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
		- 'systemctl status kubelet'
		- 'journalctl -xeu kubelet'

	Additionally, a control plane component may have crashed or exited when started by the container runtime.
	To troubleshoot, list all containers using your preferred container runtimes CLI.

	Here is one example how you may list all Kubernetes containers running in docker:
		- 'docker ps -a | grep kube | grep -v pause'
		Once you have found the failing container, you can inspect its logs with:
		- 'docker logs CONTAINERID'

couldn't initialize a Kubernetes cluster
k8s.io/kubernetes/cmd/kubeadm/app/cmd/phases/init.runWaitControlPlanePhase
	/workspace/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/cmd/kubeadm/app/cmd/phases/init/waitcontrolplane.go:118
k8s.io/kubernetes/cmd/kubeadm/app/cmd/phases/workflow.(*Runner).Run.func1
	/workspace/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/cmd/kubeadm/app/cmd/phases/workflow/runner.go:234
k8s.io/kubernetes/cmd/kubeadm/app/cmd/phases/workflow.(*Runner).visitAll
	/workspace/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/cmd/kubeadm/app/cmd/phases/workflow/runner.go:421
k8s.io/kubernetes/cmd/kubeadm/app/cmd/phases/workflow.(*Runner).Run
	/workspace/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/cmd/kubeadm/app/cmd/phases/workflow/runner.go:207
k8s.io/kubernetes/cmd/kubeadm/app/cmd.newCmdInit.func1
	/workspace/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/cmd/kubeadm/app/cmd/init.go:153
k8s.io/kubernetes/vendor/github.com/spf13/cobra.(*Command).execute
	/workspace/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/vendor/github.com/spf13/cobra/command.go:856
k8s.io/kubernetes/vendor/github.com/spf13/cobra.(*Command).ExecuteC
	/workspace/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/vendor/github.com/spf13/cobra/command.go:974
k8s.io/kubernetes/vendor/github.com/spf13/cobra.(*Command).Execute
	/workspace/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/vendor/github.com/spf13/cobra/command.go:902
k8s.io/kubernetes/cmd/kubeadm/app.Run
	/workspace/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/cmd/kubeadm/app/kubeadm.go:50
main.main
	_output/dockerized/go/src/k8s.io/kubernetes/cmd/kubeadm/kubeadm.go:25
runtime.main
	/usr/local/go/src/runtime/proc.go:255
runtime.goexit
	/usr/local/go/src/runtime/asm_amd64.s:1581
error execution phase wait-control-plane
k8s.io/kubernetes/cmd/kubeadm/app/cmd/phases/workflow.(*Runner).Run.func1
	/workspace/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/cmd/kubeadm/app/cmd/phases/workflow/runner.go:235
k8s.io/kubernetes/cmd/kubeadm/app/cmd/phases/workflow.(*Runner).visitAll
	/workspace/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/cmd/kubeadm/app/cmd/phases/workflow/runner.go:421
k8s.io/kubernetes/cmd/kubeadm/app/cmd/phases/workflow.(*Runner).Run
	/workspace/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/cmd/kubeadm/app/cmd/phases/workflow/runner.go:207
k8s.io/kubernetes/cmd/kubeadm/app/cmd.newCmdInit.func1
	/workspace/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/cmd/kubeadm/app/cmd/init.go:153
k8s.io/kubernetes/vendor/github.com/spf13/cobra.(*Command).execute
	/workspace/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/vendor/github.com/spf13/cobra/command.go:856
k8s.io/kubernetes/vendor/github.com/spf13/cobra.(*Command).ExecuteC
	/workspace/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/vendor/github.com/spf13/cobra/command.go:974
k8s.io/kubernetes/vendor/github.com/spf13/cobra.(*Command).Execute
	/workspace/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/vendor/github.com/spf13/cobra/command.go:902
k8s.io/kubernetes/cmd/kubeadm/app.Run
	/workspace/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/cmd/kubeadm/app/kubeadm.go:50
main.main
	_output/dockerized/go/src/k8s.io/kubernetes/cmd/kubeadm/kubeadm.go:25
runtime.main
	/usr/local/go/src/runtime/proc.go:255
runtime.goexit
	/usr/local/go/src/runtime/asm_amd64.s:1581
➜  ~
```

[kubeadm init shows kubelet isn't running or healthy](https://stackoverflow.com/questions/52119985/kubeadm-init-shows-kubelet-isnt-running-or-healthy)

提示Port 6443 is in use。需要先执行`sudo kubeadm reset`，重新执行init命令。终于OK了

```
[certs] Using certificateDir folder "/etc/kubernetes/pki"
I0116 13:13:30.061835    4969 certs.go:112] creating a new certificate authority for ca
[certs] Generating "ca" certificate and key
I0116 13:13:30.309186    4969 certs.go:522] validating certificate period for ca certificate
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local lee] and IPs [10.96.0.1 192.168.10.101]
[certs] Generating "apiserver-kubelet-client" certificate and key
I0116 13:13:30.726508    4969 certs.go:112] creating a new certificate authority for front-proxy-ca
[certs] Generating "front-proxy-ca" certificate and key
I0116 13:13:30.913920    4969 certs.go:522] validating certificate period for front-proxy-ca certificate
[certs] Generating "front-proxy-client" certificate and key
I0116 13:13:31.049464    4969 certs.go:112] creating a new certificate authority for etcd-ca
[certs] Generating "etcd/ca" certificate and key
I0116 13:13:31.233385    4969 certs.go:522] validating certificate period for etcd/ca certificate
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [lee localhost] and IPs [192.168.10.101 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [lee localhost] and IPs [192.168.10.101 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
I0116 13:13:32.462845    4969 certs.go:78] creating new public/private key files for signing service account users
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
I0116 13:13:32.535549    4969 kubeconfig.go:103] creating kubeconfig file for admin.conf
[kubeconfig] Writing "admin.conf" kubeconfig file
I0116 13:13:32.680926    4969 kubeconfig.go:103] creating kubeconfig file for kubelet.conf
[kubeconfig] Writing "kubelet.conf" kubeconfig file
I0116 13:13:32.758276    4969 kubeconfig.go:103] creating kubeconfig file for controller-manager.conf
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
I0116 13:13:32.880393    4969 kubeconfig.go:103] creating kubeconfig file for scheduler.conf
[kubeconfig] Writing "scheduler.conf" kubeconfig file
I0116 13:13:32.957077    4969 kubelet.go:65] Stopping the kubelet
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
I0116 13:13:33.422382    4969 manifests.go:99] [control-plane] getting StaticPodSpecs
I0116 13:13:33.422598    4969 certs.go:522] validating certificate period for CA certificate
I0116 13:13:33.422669    4969 manifests.go:125] [control-plane] adding volume "ca-certs" for component "kube-apiserver"
I0116 13:13:33.422680    4969 manifests.go:125] [control-plane] adding volume "etc-ca-certificates" for component "kube-apiserver"
I0116 13:13:33.422686    4969 manifests.go:125] [control-plane] adding volume "etc-pki" for component "kube-apiserver"
I0116 13:13:33.422691    4969 manifests.go:125] [control-plane] adding volume "k8s-certs" for component "kube-apiserver"
I0116 13:13:33.422696    4969 manifests.go:125] [control-plane] adding volume "usr-local-share-ca-certificates" for component "kube-apiserver"
I0116 13:13:33.422702    4969 manifests.go:125] [control-plane] adding volume "usr-share-ca-certificates" for component "kube-apiserver"
I0116 13:13:33.424792    4969 manifests.go:154] [control-plane] wrote static Pod manifest for component "kube-apiserver" to "/etc/kubernetes/manifests/kube-apiserver.yaml"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
I0116 13:13:33.424817    4969 manifests.go:99] [control-plane] getting StaticPodSpecs
I0116 13:13:33.425021    4969 manifests.go:125] [control-plane] adding volume "ca-certs" for component "kube-controller-manager"
I0116 13:13:33.425031    4969 manifests.go:125] [control-plane] adding volume "etc-ca-certificates" for component "kube-controller-manager"
I0116 13:13:33.425037    4969 manifests.go:125] [control-plane] adding volume "etc-pki" for component "kube-controller-manager"
I0116 13:13:33.425043    4969 manifests.go:125] [control-plane] adding volume "flexvolume-dir" for component "kube-controller-manager"
I0116 13:13:33.425049    4969 manifests.go:125] [control-plane] adding volume "k8s-certs" for component "kube-controller-manager"
I0116 13:13:33.425054    4969 manifests.go:125] [control-plane] adding volume "kubeconfig" for component "kube-controller-manager"
I0116 13:13:33.425060    4969 manifests.go:125] [control-plane] adding volume "usr-local-share-ca-certificates" for component "kube-controller-manager"
I0116 13:13:33.425066    4969 manifests.go:125] [control-plane] adding volume "usr-share-ca-certificates" for component "kube-controller-manager"
I0116 13:13:33.425761    4969 manifests.go:154] [control-plane] wrote static Pod manifest for component "kube-controller-manager" to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
[control-plane] Creating static Pod manifest for "kube-scheduler"
I0116 13:13:33.425780    4969 manifests.go:99] [control-plane] getting StaticPodSpecs
I0116 13:13:33.425961    4969 manifests.go:125] [control-plane] adding volume "kubeconfig" for component "kube-scheduler"
I0116 13:13:33.428424    4969 manifests.go:154] [control-plane] wrote static Pod manifest for component "kube-scheduler" to "/etc/kubernetes/manifests/kube-scheduler.yaml"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
I0116 13:13:33.429151    4969 local.go:65] [etcd] wrote Static Pod manifest for a local etcd member to "/etc/kubernetes/manifests/etcd.yaml"
I0116 13:13:33.429175    4969 waitcontrolplane.go:91] [wait-control-plane] Waiting for the API server to be healthy
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 15.501883 seconds
I0116 13:13:48.932031    4969 uploadconfig.go:110] [upload-config] Uploading the kubeadm ClusterConfiguration to a ConfigMap
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
I0116 13:13:48.954852    4969 uploadconfig.go:124] [upload-config] Uploading the kubelet component config to a ConfigMap
[kubelet] Creating a ConfigMap "kubelet-config-1.23" in namespace kube-system with the configuration for the kubelets in the cluster
NOTE: The "kubelet-config-1.23" naming of the kubelet ConfigMap is deprecated. Once the UnversionedKubeletConfigMap feature gate graduates to Beta the default name will become just "kubelet-config". Kubeadm upgrade will handle this transition transparently.
I0116 13:13:49.004597    4969 uploadconfig.go:129] [upload-config] Preserving the CRISocket information for the control-plane node
I0116 13:13:49.004634    4969 patchnode.go:31] [patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock" to the Node API object "lee" as an annotation
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node lee as control-plane by adding the labels: [node-role.kubernetes.io/master(deprecated) node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node lee as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: sb4nsd.o4c2svxc6ey18vzv
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
I0116 13:13:50.288207    4969 clusterinfo.go:47] [bootstrap-token] loading admin kubeconfig
I0116 13:13:50.288795    4969 clusterinfo.go:58] [bootstrap-token] copying the cluster from admin.conf to the bootstrap kubeconfig
I0116 13:13:50.289032    4969 clusterinfo.go:70] [bootstrap-token] creating/updating ConfigMap in kube-public namespace
I0116 13:13:50.295851    4969 clusterinfo.go:84] creating the RBAC rules for exposing the cluster-info ConfigMap in the kube-public namespace
I0116 13:13:50.336758    4969 kubeletfinalize.go:90] [kubelet-finalize] Assuming that kubelet client certificate rotation is enabled: found "/var/lib/kubelet/pki/kubelet-client-current.pem"
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
I0116 13:13:50.338234    4969 kubeletfinalize.go:134] [kubelet-finalize] Restarting the kubelet to enable client certificate rotation
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.10.101:6443 --token sb4nsd.o4c2svxc6ey18vzv \
	--discovery-token-ca-cert-hash sha256:a21fa42fb12f65c33dab7dd37cc2341c07ba4b90675500a76b52a39e1507e082
```

默认添加的node节点没有roles的描述信息，可以通过`label`命令指定或重新，[参考](https://stackoverflow.com/questions/48854905/how-to-add-roles-to-nodes-in-kubernetes)

```
kubectl label nodes <your_node> kubernetes.io/role=<your_label>
kubectl label --overwrite nodes <your_node> kubernetes.io/role=<your_new_label>
```

### wsl2 时间同步

按照[Kubernetes Tutorial - Step by Step Introduction to Basic Concepts](https://auth0.com/blog/kubernetes-tutorial-step-by-step-introduction-to-basic-concepts/)介绍在DO上创建免费的k8s集群，在wsl2里进行访问时提示错误：`Unable to connect to the server: x509: certificate has expired or is not yet valid: current time 2021-11-09T02:59:51+08:00 is before 2021-11-09T07:47:25Z`查看本地时间，显示比实际实际慢

执行`sudo hwclock -s `同步时间依然没有变化，官方[issue 4149](https://github.com/microsoft/WSL/issues/4149)中提供了临时解决方案——netdate。立即生效，不影响任何进程

```
# 按照ntpdate set the date and time via  Network Time Protocol (NTP)
sudo apt install ntpdate
# 同步时间
sudo ntpdate -sb time.nist.gov
# 或执行
sudo ntpdate time.windows.com

myu@Gebitang:~$ sudo ntpdate time.windows.com
 9 Nov 16:09:43 ntpdate[1727]: step time server 52.231.114.183 offset 46499.066385 sec
```

同步时间之后，可以跟集群正常交互

### Kubernetes Tutorial

上面提到的文章有点老，遇到的问题记录一下——

- 通过promote code注册DO需要手动提交个ticket，否则无法创建k8s集群，提示droplet超限
- 部署`NGINX ingress controller`的yaml文件地址已失效，参考[ingress-nginx](https://kubernetes.github.io/ingress-nginx/deploy/#digital-ocean)中的说明，使用`https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.0.4/deploy/static/provider/do/deploy.yaml`文件

```
myu@Gebitang:~$ k get pods -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-n2dgj        0/1     Completed   0          87s
ingress-nginx-admission-patch-jldqd         0/1     Completed   2          86s
ingress-nginx-controller-5c8d66c76d-fgvpb   1/1     Running     0          90s
```

- 最终部署完service之后，获取外网ip的步骤也失效了，通过`k get svc -n ingress-nginx`可以看到外网ip地址，尽管访问服务时提示404，至少表示nginx启动起来了

### K8S中的CRD开发

CRD: `CustomResourceDefinition`, Custom code that defines a resource to add to your Kubernetes API server without building a complete custom server.

[2021-05-04版本](http://bingerambo.com/posts/2021/05/k8s%E4%B8%AD%E7%9A%84crd%E5%BC%80%E5%8F%91)流程。依赖[code-generator](https://github.com/kubernetes/code-generator)，官方示例[sample-controller](https://github.com/kubernetes/sample-controller)，也可以参考[K8s Client代码自动生成](http://ljchen.net/2019/06/14/K8s-client%E4%BB%A3%E7%A0%81%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90/)

使用代码生成器生成的api兼容标准的k8s api，操作动作相同。


### setup with dashboard

[download binary](https://www.downloadkubernetes.com/)

[Install kubectl on Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

关于UI管理使用的[kubernetes/dashboard](https://github.com/kubernetes/dashboard/tree/master/docs)，参考具体的[doc: creating sample user](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md)

关于用户的步骤——

- 基于RBAC的方式创建`ServiceAccount`
- 将创建的SA进行`CluserRoleBinding`

示例中如果将namespace指定为`cluser-admin`，将拥有管理员权限。(通过`k get clusterroles`查看集群中都有哪些角色)

```
chmod +x kubectl
mkdir -p ~/.local/bin/kubectl
mv ./kubectl ~/.local/bin/kubectl
# and then add ~/.local/bin/kubectl to $PATH
```

自动完成Enable kubectl autocompletion

```shell
source <(kubectl completion bash)
alias k=kubectl
complete -F __start_kubectl k

#shell生效
echo 'source <(kubectl completion bash)' >>~/.bashrc
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -F __start_kubectl k' >>~/.bashrc
```

使用`kind`创建集群 [quick start](https://kind.sigs.k8s.io/docs/user/quick-start/)
```shell
# 创建一个3节点集群的配置文件
cat << EOF > kind-3nodes.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
EOF
# 使用配置文件创建新的集群
kind create cluster --name k3s --config ./kind-3nodes.yaml
# 获取集群节点
kubectl get nodes
# 删除集群
kind delete cluster --name k3s
```
### WSL+Docker的限制

[WSL+Docker: Kubernetes on the Windows Desktop](https://kubernetes.io/blog/2020/05/21/wsl-docker-kubernetes-on-the-windows-desktop/) 

从WSL2里的ubuntu20.04镜像里使用kind创建的k8s cluster，无法cluster的node的ip，网络不通。——这看起来是个已知的问题：[Networking features in Docker Desktop for Windows](https://docs.docker.com/desktop/windows/networking/)——

- **There is no docker0 bridge on Windows**  
Because of the way networking is implemented in Docker Desktop for Windows, you cannot see a docker0 interface on the host. This interface is actually within the virtual machine.

- **I cannot ping my containers**  
Docker Desktop for Windows can’t route traffic to Linux containers. However, you can ping the Windows containers.

- **Per-container IP addressing is not possible** 
The docker (Linux) bridge network is not reachable from the Windows host. However, it works with Windows containers.


跟着[演示](https://www.qikqiak.com/post/deploy-k8s-on-win-use-wsl2)直到部署三个节点的cluster都正常，安装一个 Kubernetes Dashboard时(`kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.1/aio/deploy/recommended.yaml`)，`kubectl get all -n kubernetes-dashboard`命令显示，一直处于"ContainerCreating"状态。感觉应该是我本地网络原因？ [issue 2863](https://github.com/kubernetes/dashboard/issues/2863)


多个配置文件切换[Configure Access to Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

```shell
# online.config.yaml for pro, dev.config.yaml for dev
kubectl config --kubeconfig=online.config.yaml view

# export
export KUBECONFIG=$KUBECONFIG:$HOME/.kube/config
export KUBECONFIG_SAVED=$KUBECONFIG
kubectl config view
# clean up 
export KUBECONFIG=$KUBECONFIG_SAVED
```

```
geb@Gebitang:~$ kubectl get all -n kubernetes-dashboard
NAME                                             READY   STATUS              RESTARTS   AGE
pod/dashboard-metrics-scraper-5594697f48-wgpst   0/1     ContainerCreating   0          14m
pod/kubernetes-dashboard-bc8f4fc6f-f2mqg         0/1     ContainerCreating   0          14m

NAME                                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/dashboard-metrics-scraper   ClusterIP   10.96.21.147   <none>        8000/TCP   14m
service/kubernetes-dashboard        ClusterIP   10.96.173.37   <none>        443/TCP    14m

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dashboard-metrics-scraper   0/1     1            0           14m
deployment.apps/kubernetes-dashboard        0/1     1            0           14m

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/dashboard-metrics-scraper-5594697f48   1         1         0       14m
replicaset.apps/kubernetes-dashboard-bc8f4fc6f         1         1         0       14m

#  kubectl get pods --all-namespaces
NAMESPACE              NAME                                                      READY   STATUS              RESTARTS   AGE
kube-system            coredns-558bd4d5db-57bh5                                  1/1     Running             0          16m
kube-system            coredns-558bd4d5db-sfsfp                                  1/1     Running             0          16m
kube-system            etcd-wslkindmultinodes-control-plane                      1/1     Running             0          17m
kube-system            kindnet-2k9fg                                             1/1     Running             0          16m
kube-system            kindnet-ssl4j                                             1/1     Running             0          16m
kube-system            kindnet-z5mgq                                             1/1     Running             0          16m
kube-system            kube-apiserver-wslkindmultinodes-control-plane            1/1     Running             0          17m
kube-system            kube-controller-manager-wslkindmultinodes-control-plane   1/1     Running             1          17m
kube-system            kube-proxy-b4r8b                                          1/1     Running             0          16m
kube-system            kube-proxy-mwlms                                          1/1     Running             0          16m
kube-system            kube-proxy-smhm5                                          1/1     Running             0          16m
kube-system            kube-scheduler-wslkindmultinodes-control-plane            1/1     Running             1          17m
kubernetes-dashboard   dashboard-metrics-scraper-5594697f48-wgpst                0/1     ContainerCreating   0          15m
kubernetes-dashboard   kubernetes-dashboard-bc8f4fc6f-f2mqg                      0/1     ContainerCreating   0          15m
local-path-storage     local-path-provisioner-547f784dff-dngw5                   1/1     Running             0          16m
```

需要先安装网络—— 

[参考这个comment](https://github.com/kubernetes/dashboard/issues/2863#issuecomment-368785304)

```
# 可以将url内容保存到文件进行执行
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml
```

执行之后，`kubectl get all -n kubernetes-dashboard`可以看到对应的pod都正常运行起来了

```
geb@Gebitang:~$ kubectl get all -n kubernetes-dashboard
NAME                                             READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-5594697f48-wgpst   1/1     Running   0          33m
pod/kubernetes-dashboard-bc8f4fc6f-f2mqg         1/1     Running   0          33m

NAME                                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/dashboard-metrics-scraper   ClusterIP   10.96.21.147   <none>        8000/TCP   33m
service/kubernetes-dashboard        ClusterIP   10.96.173.37   <none>        443/TCP    33m

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dashboard-metrics-scraper   1/1     1            1           33m
deployment.apps/kubernetes-dashboard        1/1     1            1           33m

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/dashboard-metrics-scraper-5594697f48   1         1         1       33m
replicaset.apps/kubernetes-dashboard-bc8f4fc6f         1         1         1       33m
```


proxy启动后，使用如下方式获取token，
```
# 创建一个新的 ServiceAccount
kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF
# 将上面的 SA 绑定到系统的 cluster-admin 这个集群角色上
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF

# ----------actual result-------
geb@Gebitang:~$ kubectl apply -f - <<EOF
> apiVersion: v1
> kind: ServiceAccount
etadata:> metadata:
>   name: admin-user
namespac>   namespace: kubernetes-dashboard
> EOF
serviceaccount/admin-user created
geb@Gebitang:~$ kubectl apply -f - <<EOF
iVersio> apiVersion: rbac.authorization.k8s.io/v1
> kind: ClusterRoleBinding
> metadata:
>   name: admin-user
> roleRef:
>   apiGroup: rbac.authorization.k8s.io
>   kind: ClusterRole
>   name: cluster-admin
> subjects:
> - kind: ServiceAccount
>   name: admin-user
>   namespace: kubernetes-dashboard
> EOF
clusterrolebinding.rbac.authorization.k8s.io/admin-user created
geb@Gebitang:~$
geb@Gebitang:~$ kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
Name:         admin-user-token-d4mkv
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: b66a6322-81a3-4bec-af6a-c329c4702ef8

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IjdnaktTQ2Nuc1RfbGhWaWs3NHYtRzF2VFhIU2ZTX3g4aFBHUkNfYkROOU0ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWQ0bWt2Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJiNjZhNjMyMi04MWEzLTRiZWMtYWY2YS1jMzI5YzQ3MDJlZjgiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.pSiZrLue_t9R8qXN7Dm6jaOblVnLr_qI0NvkL25N3TB78rvR3O2zIzZ0cJ5ylvl7hdxIRXoKQxZUvY8cCeFRosBdGp_gaevd_33vlLNfcj0UM3x_9IQeS4cqPlI0EkWcrVXbPbQQyCLKBSgaaCNQrijtiQXUgaK2eLh8FCmvFIIE5U2vs-GqlmTy1YNpGr28WjtML-bFvQIsL7BpZieWzBNb7VfnsNMRPjpmUUE6iurVApwZSNTesFcKQP-dW7trN0B8hI7SllRMN64rfUnNwZX58eI5esgicQ2STJ64WkqTRYbGs1up__Iz_xvoRYXQq4bN_SSVnCnLOtf8jpVuyA
geb@Gebitang:~$
```

访问`http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/`选择token方式，将上面获取到的token输入后，就可以看到UI页面

删除集群——

```
# 查找集群
kind get cluster 
# 删除现在的集群
kind delete cluster --name wslkindmultinodes
```
## archive 
### 术语列表


{{< fluid_imgs
  "pure-u-1-1|https://static001.geekbang.org/resource/image/8e/67/8ee9f2fa987eccb490cfaa91c6484f67.png|Kubernetes项目架构|https://time.geekbang.org/column/article/23132"
>}}

- OCI: Open Container Initiative
- CNI: Container Network Interface
- CRI: Container Runtime Interface
- CSI: Container Storage Interface

使用环境Windows。

### WSL2 + Kubernetes

[在 Windows 下使用 WSL2 搭建 Kubernetes 集群](https://www.qikqiak.com/post/deploy-k8s-on-win-use-wsl2)

- 安装商店里的`windows terminal`
- 更新ubuntu的软件源

```
cp /etc/apt/sources.list /etc/apt/sources.list.bak
root@k8s:~# echo "deb http://mirrors.aliyun.com/ubuntu/ focal main restricted
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted
deb http://mirrors.aliyun.com/ubuntu/ focal universe
deb http://mirrors.aliyun.com/ubuntu/ focal-updates universe
deb http://mirrors.aliyun.com/ubuntu/ focal multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted
deb http://mirrors.aliyun.com/ubuntu/ focal-security universe
deb http://mirrors.aliyun.com/ubuntu/ focal-security multiverse" > /etc/apt/sources.list
```

- kind可以从github上下载后手动安装——本身是go编译的二进制文件
- 但kind安装kubernetes集群时使用的是编译发布在hub.docker.com上的镜像，WSL2里遇到网络问题

>This will bootstrap a Kubernetes cluster using a pre-built [node image](https://kind.sigs.k8s.io/docs/design/node-image). Prebuilt images are hosted at[`kindest/node`](https://hub.docker.com/r/kindest/node/)

```
root@Gebitang:/home/geb# kind create cluster --name wslk8s
Creating cluster "wslk8s" ...
 ✓ Ensuring node image (kindest/node:v1.21.1) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-wslk8s"
You can now use your cluster with:

kubectl cluster-info --context kind-wslk8s

Thanks for using kind! 😊
```

重新使用`kubeadm`进行安装：

- 需要先将apt-key.gpg证书导入`curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -`。可以先将文件下载到本地，然后`cat apt-get.gpg | apt-key add -`
- 添加镜像源到列表：在`/etc/apt/sources.list.d/`目录下创建`kubernetes.list`文件，内容为`deb http://mirrors.ustc.edu.cn/kubernetes/apt kubernetes-xenial main`
- 执行`apt-get update && apt-get install -y kubeadm`

执行`kubeadm version`返回信息。

### Errors were encountered while processing:  ubuntu-advantage-tools

ubuntu20.04执行`apt-get upgrade`时遇到类似上面的问题，看起来是个[bug](https://bugs.launchpad.net/ubuntu/+source/ubuntu-advantage-tools/+bug/1938097)，官方提示修复方式：

- 编辑`/var/lib/dpkg/info/ubuntu-advantage-tools.postinst`
- 执行`dpkg --configure -a`
```
I think the problem is in the postinst script, line 295:

    cloud_id=""
    if command -v "cloud-id" > /dev/null ; then
      cloud_id=$(cloud-id)
    fi

The third line should rather be:

      cloud_id=$(cloud-id || true)
```


根据[云原生技术基础](https://edu.aliyun.com/roadmap/cloudnative)的[第三课的demo](https://edu.aliyun.com/lesson_1651_16894)

### 前提环境

三件套：VirturalBox、kubectl、minikube。

- 安装 VirtualBox： [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)

- 安装[kubectl on windows](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows)

下载、添加PATH、验证版本

1.  Download the latest release v1.18.0 from [this link](https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/windows/amd64/kubectl.exe).

    Or if you have `curl` installed, use this command:

    ```
    curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/windows/amd64/kubectl.exe
    ```

    To find out the latest stable version (for example, for scripting), take a look at [https://storage.googleapis.com/kubernetes-release/release/stable.txt](https://storage.googleapis.com/kubernetes-release/release/stable.txt).

2.  Add the binary in to your PATH.

3.  Test to ensure the version of `kubectl` is the same as downloaded:

    ```
    kubectl version --client
    ```


- 安装minikube，[原生安装](https://kubernetes.io/docs/tasks/tools/install-minikube/)，[aliyun: Minikube - Kubernetes本地实验环境](https://developer.aliyun.com/article/221687)

先进行系统检测`systeminfo` 

```
Hyper-V Requirements:     A hypervisor has been detected. Features required for Hyper-V will not be displayed.
Hyper-V 要求:     已检测到虚拟机监控程序。将不显示 Hyper-V 所需的功能。
```

---

启动命令： `minikube start --driver virtualbox`
```
D:\>minikube start --driver=virtualbox
* Microsoft Windows 10 Education 10.0.18363 Build 18363 上的 minikube v1.12.3
* 根据现有的配置文件使用 virtualbox 驱动程序
* Starting control plane node minikube in cluster minikube
* Creating virtualbox VM (CPUs=4, Memory=6000MB, Disk=20000MB) ...
! StartHost failed, but will try again: creating host: create: precreate: This computer is running Hyper-V. VirtualBox won't boot a 64bits VM when Hyper-V is activated. Either use Hyper-V as a driver, or disable the Hyper-V hypervisor. (To skip this check, use --virtualbox-no-vtx-check)
* Creating virtualbox VM (CPUs=4, Memory=6000MB, Disk=20000MB) ...
* Failed to start virtualbox VM. "minikube start" may fix it: creating host: create: precreate: This computer is running Hyper-V. VirtualBox won't boot a 64bits VM when Hyper-V is activated. Either use Hyper-V as a driver, or disable the Hyper-V hypervisor. (To skip this check, use --virtualbox-no-vtx-check)
*
E0821 17:24:38.091674   11324 exit.go:76] &{ID:VBOX_HYPERV_64_BOOT Err:This computer is running Hyper-V. VirtualBox won't boot a 64bits VM when Hyper-V is activated. Either use Hyper-V as a driver, or disable the Hyper-V hypervisor. (To skip this check, use --virtualbox-no-vtx-check)
precreate
k8s.io/minikube/pkg/minikube/machine.(*LocalClient).Create
        /app/pkg/minikube/machine/client.go:225
k8s.io/minikube/pkg/minikube/machine.timedCreateHost.func2
        /app/pkg/minikube/machine/start.go:188
runtime.goexit
        /usr/local/go/src/runtime/asm_amd64.s:1373
create
k8s.io/minikube/pkg/minikube/machine.timedCreateHost
        /app/pkg/minikube/machine/start.go:197
k8s.io/minikube/pkg/minikube/machine.createHost
        /app/pkg/minikube/machine/start.go:163
k8s.io/minikube/pkg/minikube/machine.StartHost
        /app/pkg/minikube/machine/start.go:86
k8s.io/minikube/pkg/minikube/node.startHost
        /app/pkg/minikube/node/start.go:401
k8s.io/minikube/pkg/minikube/node.startMachine
        /app/pkg/minikube/node/start.go:345
k8s.io/minikube/pkg/minikube/node.Provision
        /app/pkg/minikube/node/start.go:234
k8s.io/minikube/cmd/minikube/cmd.provisionWithDriver
        /app/cmd/minikube/cmd/start.go:275
k8s.io/minikube/cmd/minikube/cmd.runStart
        /app/cmd/minikube/cmd/start.go:169
github.com/spf13/cobra.(*Command).execute
        /go/pkg/mod/github.com/spf13/cobra@v1.0.0/command.go:846
github.com/spf13/cobra.(*Command).ExecuteC
        /go/pkg/mod/github.com/spf13/cobra@v1.0.0/command.go:950
github.com/spf13/cobra.(*Command).Execute
        /go/pkg/mod/github.com/spf13/cobra@v1.0.0/command.go:887
k8s.io/minikube/cmd/minikube/cmd.Execute
        /app/cmd/minikube/cmd/root.go:106
main.main
        /app/cmd/minikube/main.go:71
runtime.main
        /usr/local/go/src/runtime/proc.go:203
runtime.goexit
        /usr/local/go/src/runtime/asm_amd64.s:1373
creating host
k8s.io/minikube/pkg/minikube/machine.createHost
        /app/pkg/minikube/machine/start.go:164
k8s.io/minikube/pkg/minikube/machine.StartHost
        /app/pkg/minikube/machine/start.go:86
k8s.io/minikube/pkg/minikube/node.startHost
        /app/pkg/minikube/node/start.go:401
k8s.io/minikube/pkg/minikube/node.startMachine
        /app/pkg/minikube/node/start.go:345
k8s.io/minikube/pkg/minikube/node.Provision
        /app/pkg/minikube/node/start.go:234
k8s.io/minikube/cmd/minikube/cmd.provisionWithDriver
        /app/cmd/minikube/cmd/start.go:275
k8s.io/minikube/cmd/minikube/cmd.runStart
        /app/cmd/minikube/cmd/start.go:169
github.com/spf13/cobra.(*Command).execute
        /go/pkg/mod/github.com/spf13/cobra@v1.0.0/command.go:846
github.com/spf13/cobra.(*Command).ExecuteC
        /go/pkg/mod/github.com/spf13/cobra@v1.0.0/command.go:950
github.com/spf13/cobra.(*Command).Execute
        /go/pkg/mod/github.com/spf13/cobra@v1.0.0/command.go:887
k8s.io/minikube/cmd/minikube/cmd.Execute
        /app/cmd/minikube/cmd/root.go:106
main.main
        /app/cmd/minikube/main.go:71
runtime.main
        /usr/local/go/src/runtime/proc.go:203
runtime.goexit
        /usr/local/go/src/runtime/asm_amd64.s:1373
Failed to start host
k8s.io/minikube/pkg/minikube/node.startMachine
        /app/pkg/minikube/node/start.go:347
k8s.io/minikube/pkg/minikube/node.Provision
        /app/pkg/minikube/node/start.go:234
k8s.io/minikube/cmd/minikube/cmd.provisionWithDriver
        /app/cmd/minikube/cmd/start.go:275
k8s.io/minikube/cmd/minikube/cmd.runStart
        /app/cmd/minikube/cmd/start.go:169
github.com/spf13/cobra.(*Command).execute
        /go/pkg/mod/github.com/spf13/cobra@v1.0.0/command.go:846
github.com/spf13/cobra.(*Command).ExecuteC
        /go/pkg/mod/github.com/spf13/cobra@v1.0.0/command.go:950
github.com/spf13/cobra.(*Command).Execute
        /go/pkg/mod/github.com/spf13/cobra@v1.0.0/command.go:887
k8s.io/minikube/cmd/minikube/cmd.Execute
        /app/cmd/minikube/cmd/root.go:106
main.main
        /app/cmd/minikube/main.go:71
runtime.main
        /usr/local/go/src/runtime/proc.go:203
runtime.goexit
        /usr/local/go/src/runtime/asm_amd64.s:1373 Advice:VirtualBox and Hyper-V are having a conflict. Use '--driver=hyperv' or disable Hyper-V using: 'bcdedit /set hypervisorlaunchtype off' URL: Issues:[4051 4783] ShowIssueLink:false}
* [VBOX_HYPERV_64_BOOT] error provisioning host Failed to start host: creating host: create: precreate: This computer is running Hyper-V. VirtualBox won't boot a 64bits VM when Hyper-V is activated. Either use Hyper-V as a driver, or disable the Hyper-V hypervisor. (To skip this check, use --virtualbox-no-vtx-check)
* 建议：VirtualBox and Hyper-V are having a conflict. Use '--driver=hyperv' or disable Hyper-V using: 'bcdedit /set hypervisorlaunchtype off'
* 相关问题：
  - https://github.com/kubernetes/minikube/issues/4051
  - https://github.com/kubernetes/minikube/issues/4783

```

报错信息还是很清楚，运行时冲突：
- 使用管理员权限执行`bcdedit /set hypervisorlaunchtype off`，成功后，还是提示相同的问题。
- 使用`--virtualbox-no-vtx-check` 提示这个`Error: unknown flag: --virtualbox-no-vtx-check` 可以See `minikube start --help' for usage.`

果然，现在的参数是`--no-vtx-check=true`。所以完整的参数应该是`minikube start --driver=virtualbox --no-vtx-check=true`

不幸的时，还是遇到问题了

```
PS D:\> minikube start --driver=virtualbox --no-vtx-check=true
* Microsoft Windows 10 Education 10.0.18363 Build 18363 上的 minikube v1.12.3
* 根据现有的配置文件使用 virtualbox 驱动程序
* Starting control plane node minikube in cluster minikube
* Creating virtualbox VM (CPUs=4, Memory=6000MB, Disk=20000MB) ...
* 正在 Docker 19.03.12 中准备 Kubernetes v1.18.3…
* Unable to load cached images: loading cached images: transferring cached image: sudo test -d /var/lib/minikube/images && sudo scp -t /var/lib/minikube/images && sudo touch -d "2020-08-21 17:31:20.5418176 +0800" /var/lib/minikube/images/etcd_3.4.3-0: wait: remote command exited without exit status or exit signal
output:
...
stdout:

stderr:
fatal error: index out of range
runtime: panic before malloc heap initialized

```

执行`bcdedit /set hypervisorlaunchtype off`之后出现docker无法启动情况？管理员权限执行`bcdedit /Set {current} hypervisorlaunchtype auto`恢复初始值，执行`bcdedit`可看到全部配置信息。

自动提示链接为[VIRTUALIZATION MUST BE ENABLED](https://docs.docker.com/docker-for-windows/troubleshoot/#virtualization-must-be-enabled)，但检查是符合要求的。重启后再看吧。接着先往下学课程。


