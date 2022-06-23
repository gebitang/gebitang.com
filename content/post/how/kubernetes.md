+++
title = "k8s practice"
description = "the way to kubernetes"
tags = [
    "kubernetes",
    "k8s"
]
date = "2022-03-25"
topics = [
    "kubernetes",
    "k8s"
]
draft = false
toc = true
+++



{{< fluid_imgs
  "pure-u-1-1|https://static001.geekbang.org/resource/image/8e/67/8ee9f2fa987eccb490cfaa91c6484f67.png|Kubernetes项目架构|https://time.geekbang.org/column/article/23132"
>}}

- OCI: Open Container Initiative
- CNI: Container Network Interface
- CRI: Container Runtime Interface
- CSI: Container Storage Interface

[OCI运行时规范](https://mp.weixin.qq.com/s/NYhUQqaN9v1s36EHqpw-kA)

- 符合 CRI 标准的 containerd，以及底层的 runC，都是从Docker 项目中分拆出来。
  - 2015 年OCI成立之初，Docker 的 libcontainer 项目被捐赠给OCI，成为独立的容器运行时项目 runC
  - 2017年，Docker公司高层容器运行时的功能集中到containerd项目里，捐赠给云原生计算基金会。
  - Docker 引擎在发布时是一个单体应用，后来按功能分拆成 runC 和 containerd 两个不同层次的运行时。
- CRI-O是替代Docker或者containerd的高效且轻量级的容器运行时方案，是CRI的一个实现，能够运行符合OCI规范的容器，所以被称为CRI-O。

{{< fluid_imgs
  "pure-u-1-1|https://s3-img.meituan.net/v1/mss_3d027b52ec5a4d589e68050845611e68/ff/n0/0n/34/dp_301168.jpg|OCI"
>}}

## k8s 恢复

[ETCD 集群的备份和恢复](https://blog.csdn.net/qq_34556414/article/details/113882631)



```
1.停止所有 Master 上 kube-apiserver 服务

systemctl stop kube-apiserver

2.停止集群中所有 ETCD 服务
systemctl stop etcd

3.移除所有 ETCD 存储目录下数据
mv /etcd/data /etcd/data.bak

4.从备份文件中恢复数据
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot-xx.db  

5.启动 etcd
systemctl start etcd

6.启动 apiserver
systemctl start kube-apiserver

7.检查服务是否正常
```

## k8s 搭建

在CentOS机器上安装，配置1个master+2个node机器

### 前置配置

- 关闭防火墙 （保证网络通畅）
- 关闭`swap`分区（防止kubelet重启失败）
- 设置主机名（master节点可识别到各个node名称）
- 添加hosts解析 （将集群所有的ip和对应的主机名添加到各自的hosts文件中国）
- 打开`ipv6`流量转发
- 配置国内镜像源
- 时间同步服务

操作过程——

```shell
# close firewall
systemctl disable --now firewalld
setenforce 0
sed -i 's/enforcing/disabled/' /etc/selinux/config 

# close swap 
# comment the line contain swap config in 
swapoff -a
sed -i.bak 's/^.*centos-swap/#&/g' /etc/fstab

# set host name
hostnamectl set-hostname master

# add file /etc/sysctl.d/k8s.conf  with following content
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
# make it work at once
sysctl --system 

# config native yum repo.
mv /etc/yum.repos.d/* /tmp # back up first
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

# sync time
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
yum install dnf ntpdate -y # install dnf 
dnf makecache
ntpdate ntp.aliyun.com
```

#### dnf 

DNF(Dandified Yum)是新一代的RPM软件包管理器

```
# 列出所有 RPM 包
dnf list

# 安装软件包
dnf install wget

# 删除软件包
dnf remove wget

# 查看所有的软件包组
dnf grouplist

# 安装一个软件包组
dnf groupinstall ‘安全性工具’

# 查看系统中可用的 DNF 软件库
dnf repolist

# 查看系统中可用和不可用的所有的 DNF 软件库
dnf repolist all

# 列出所有安装了的 RPM 包
dnf list installed

# 列出所有可供安装的 RPM 包
dnf list available

# 搜索软件库中的 RPM 包
dnf search wget

# 查找某一文件的提供者
dnf provides /bin/bash

# 查看软件包详情
dnf info wget

# 删除无用孤立的软件包
dnf autoremove

# 删除缓存的无用软件包
dnf clean all

# 获取有关某条命令的使用帮助
dnf help clean

# 查看 DNF 命令的执行历史
dnf history

# 从特定的软件包库安装特定的软件
dnf -enablerepo=epel install nginx

# 重新安装特定软件包
dnf reinstall wget
```

### 安装docker服务

- 配置国内源
- 安装并设置开机启动
- 检查版本
- 配置docker的镜像仓库

```shell
# add native repo
curl -o /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# install by dnf 
dnf install -y  docker-ce docker-ce-cli

# config auto start
systemctl enable --now docker

#check version
docker --version 

# add customed repo for docker. 
cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://f1bhsuge.mirror.aliyuncs.com"]
}
EOF

systemctl restart docker
```

### 安装kubernetes服务

- 配置国内源
- 安装组件kubeadm、kubelet、kubectl
- 设置开始启动 `systemctl enable kubelet`

```shell
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# show version
dnf list kubeadm --showduplicates
# packename+version name
dnf install -y kubelet-1.20.10 kubeadm-1.20.10 kubectl-1.20.10

# auto start
systemctl enable kubelet
```

### 准备脚本

上述环境下，root用户执行如下脚步一次性完工——（除啦hosts文件未更新）

```shell
#!/bin/bash

function closeFireWall() {
	systemctl disable --now firewalld
	setenforce 0
	sed -i 's/enforcing/disabled/' /etc/selinux/config
}

function closeSwap() {
	swapoff -a
	# sed -ri 's/.*swap.*/#&/' /etc/fstab
	sed -ri 's/.*swap.*/#&/' /etc/fstab 
}

function setHostName() {
    hostnamectl set-hostname master
}

function supportIpv6Traffic() {
	echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.d/k8s.conf
	echo 'net.bridge.bridge-nf-call-ip6tables = 1' >> /etc/sysctl.d/k8s.conf
	echo 'net.bridge.bridge-nf-call-iptables = 1' >> /etc/sysctl.d/k8s.conf
	sysctl --system
}

function updateYumRepo() {
	mkdir -p /home/sa/repo.bak
	mv /etc/yum.repos.d/* /home/sa/repo.bak/
	curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
	curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
}

function syncTime() {
	ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
	yum install dnf ntpdate -y # install dnf
	dnf makecache
	ntpdate ntp.aliyun.com
}

function installDocker() {
	# add native repo
	curl -o /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
	# install by dnf
	dnf install -y  docker-ce docker-ce-cli

	# add customed repo for docker.
	mkdir -p /etc/docker
	touch /etc/docker/daemon.json
	cat > /etc/docker/daemon.json << EOF
	{
	  "registry-mirrors": ["https://f1bhsuge.mirror.aliyuncs.com"]
	}
EOF
	# config auto start
	systemctl enable --now docker

	#check version
	docker --version
	systemctl restart docker
}

function installK8sComponent() {
	cat > /etc/yum.repos.d/kubernetes.repo << EOF
	[kubernetes]
	name=Kubernetes
	baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
	enabled=1
	gpgcheck=0
	repo_gpgcheck=0
	gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
	# show version
	dnf list kubeadm --showduplicates
	# packename+version name
	dnf install -y kubelet-1.17.0 kubeadm-1.17.0 kubectl-1.17.0
    systemctl enable kubelet
}

function installLxcfs() {
	# insall lxcfs 
	yum -y install fuse-devel fuse  lxc-templates
	wget  --no-check-certificate  https://copr-be.cloud.fedoraproject.org/results/ganto/lxc3/epel-7-x86_64/01041891-lxcfs/lxcfs-3.1.2-0.2.el7.x86_64.rpm
	rpm -ivh lxcfs-3.1.2-0.2.el7.x86_64.rpm

	systemctl enable lxcfs
	systemctl start lxcfs
	# 运⾏lxcfs命令，指定 /proc ⽬录所在位置：
	sudo mkdir -p /var/lib/lxcfs
	sudo lxcfs /var/lib/lxcfs

	# 需要确保lxcfs服务正常启动
	systemctl status lxcfs 
}

closeFireWall
closeSwap
# setHostName
supportIpv6Traffic
updateYumRepo
syncTime
installDocker
installK8sComponent
installLxcfs
```

### 部署集群

- 生成预处理文件
- 预拉取镜像
- 部署主节点
- 生成加入节点命令

```shell
# pre-config 
kubeadm config print init-defaults > kubeadm-init.yaml

# pull image first 
kubeadm config images pull --config kubeadm-init.yaml

# start buildup master
kubeadm init --config kubeadm-init.yaml

# To print a join command for worker/slave node
kubeadm token create --print-join-command
```

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.50.200 #address						     
  bindPort:  6443 #port
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: master
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers   #阿里云的镜像站点
kind: ClusterConfiguration
kubernetesVersion: v1.20.10			#kubernetes版本号
networking:
  dnsDomain: cluster.local	
  serviceSubnet: 192.168.99.0/24		#选择默认即可，当然也可以自定义CIDR
  podSubnet: 192.168.98.0/24			#添加pod网段
scheduler: {}
```

部署完成后，必须有对应的网络插件才能让node直接正常通信，否则一直处于notReady状态，提示——

>KubeletNotReady              
>runtime network not ready: NetworkReady=false 
>reason:NetworkPluginNotReady 
>message:docker: network plugin is not ready: cni config uninitialized

### 安装网络插件

没有网络插件的集群基本不可用，通常使用calico，测试集群node小于50可直接使用[官方手册](https://projectcalico.docs.tigera.io/getting-started/kubernetes/self-managed-onprem/onpremises)

```shell
# download
curl https://projectcalico.docs.tigera.io/manifests/calico.yaml -O
# apply
kubectl apply -f calico.yaml
```


### 架构检查


{{< fluid_imgs
  "pure-u-1-1|https://s3-img.meituan.net/v1/mss_3d027b52ec5a4d589e68050845611e68/ff/n0/0n/5f/sa_507404.jpg|kubernetes"
>}}


安装完成后，集群中启动的容器大概包括以下内容——除了网络相关的内容，可以看到架构图上的内容都有对应的容器

```shell
# docker ps = docker container ls
CONTAINER ID   IMAGE           COMMAND                  CREATED       STATUS       PORTS     NAMES
7681553c1375   0cae8d5cc64c    "kube-apiserver --ad…"   2 hours ago   Up 2 hours             k8s_kube-apiserver_kube-apiserver-master_kube-system_3595fcec3afd46524fa623e9083e8e2b_88
1f6af2af6504   303ce5db0e90    "etcd --advertise-cl…"   2 hours ago   Up 2 hours             k8s_etcd_etcd-master_kube-system_3c0c04a4c2e1acb0e625dcf83a547310_108
e6250d39a65c   78c190f736b1    "kube-scheduler --au…"   3 hours ago   Up 3 hours             k8s_kube-scheduler_kube-scheduler-master_kube-system_75516e998e1ab97384d969d8ccd139db_55
95fb0ecc7f06   5eb3b7486872    "kube-controller-man…"   3 hours ago   Up 3 hours             k8s_kube-controller-manager_kube-controller-manager-master_kube-system_ab2c4278e88f8bade2b4f3742e6fba77_57
18c4a40b1c4f   pause:3.1       "/pause"                 3 hours ago   Up 3 hours             k8s_POD_kube-scheduler-master_kube-system_75516e998e1ab97384d969d8ccd139db_3
b7b9eb44f106   pause:3.1       "/pause"                 3 hours ago   Up 3 hours             k8s_POD_kube-controller-manager-master_kube-system_ab2c4278e88f8bade2b4f3742e6fba77_3
1e5402dc33eb   pause:3.1       "/pause"                 3 hours ago   Up 3 hours             k8s_POD_kube-apiserver-master_kube-system_3595fcec3afd46524fa623e9083e8e2b_3
0ae0fee45898   pause:3.1       "/pause"                 3 hours ago   Up 3 hours             k8s_POD_etcd-master_kube-system_3c0c04a4c2e1acb0e625dcf83a547310_3

# kc get pods --all-namespaces
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   etcd-master                      1/1     Running   108        4h1m
kube-system   kube-apiserver-master            1/1     Running   88         4h
kube-system   kube-controller-manager-master   1/1     Running   57         4h1m
kube-system   kube-scheduler-master            1/1     Running   55         4h1m

# docker images 
kube-proxy                   v1.17.0   7d54289267dc   2 years ago    116MB
kube-scheduler               v1.17.0   78c190f736b1   2 years ago    94.4MB
kube-apiserver               v1.17.0   0cae8d5cc64c   2 years ago    171MB
kube-controller-manager      v1.17.0   5eb3b7486872   2 years ago    161MB
coredns                      1.6.5     70f311871ae1   2 years ago    41.6MB
etcd                         3.4.3-0   303ce5db0e90   2 years ago    288MB
pause                        3.1       da86e6ba6ca1   4 years ago    742kB
calico/kube-controllers      v3.22.1   c0c6672a66a5   2 months ago   132MB
calico/cni                   v3.22.1   2a8ef6985a3e   2 months ago   236MB
calico/pod2daemon-flexvol    v3.22.1   17300d20daf9   2 months ago   19.7MB
calico/node                  v3.22.1   7a71aca7b60f   2 months ago   198MB
```

容器插件还没有安装，此时原生自带的资源包括——

```
NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND
bindings                                                                      true         Binding
componentstatuses                 cs                                          false        ComponentStatus
configmaps                        cm                                          true         ConfigMap
endpoints                         ep                                          true         Endpoints
events                            ev                                          true         Event
limitranges                       limits                                      true         LimitRange
namespaces                        ns                                          false        Namespace
nodes                             no                                          false        Node
persistentvolumeclaims            pvc                                         true         PersistentVolumeClaim
persistentvolumes                 pv                                          false        PersistentVolume
pods                              po                                          true         Pod
podtemplates                                                                  true         PodTemplate
replicationcontrollers            rc                                          true         ReplicationController
resourcequotas                    quota                                       true         ResourceQuota
secrets                                                                       true         Secret
serviceaccounts                   sa                                          true         ServiceAccount
services                          svc                                         true         Service
mutatingwebhookconfigurations                  admissionregistration.k8s.io   false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io   false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io           false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io         false        APIService
controllerrevisions                            apps                           true         ControllerRevision
daemonsets                        ds           apps                           true         DaemonSet
deployments                       deploy       apps                           true         Deployment
replicasets                       rs           apps                           true         ReplicaSet
statefulsets                      sts          apps                           true         StatefulSet
tokenreviews                                   authentication.k8s.io          false        TokenReview
localsubjectaccessreviews                      authorization.k8s.io           true         LocalSubjectAccessReview
selfsubjectaccessreviews                       authorization.k8s.io           false        SelfSubjectAccessReview
selfsubjectrulesreviews                        authorization.k8s.io           false        SelfSubjectRulesReview
subjectaccessreviews                           authorization.k8s.io           false        SubjectAccessReview
horizontalpodautoscalers          hpa          autoscaling                    true         HorizontalPodAutoscaler
cronjobs                          cj           batch                          true         CronJob
jobs                                           batch                          true         Job
certificatesigningrequests        csr          certificates.k8s.io            false        CertificateSigningRequest
leases                                         coordination.k8s.io            true         Lease
endpointslices                                 discovery.k8s.io               true         EndpointSlice
events                            ev           events.k8s.io                  true         Event
ingresses                         ing          extensions                     true         Ingress
ingresses                         ing          networking.k8s.io              true         Ingress
networkpolicies                   netpol       networking.k8s.io              true         NetworkPolicy
runtimeclasses                                 node.k8s.io                    false        RuntimeClass
poddisruptionbudgets              pdb          policy                         true         PodDisruptionBudget
podsecuritypolicies               psp          policy                         false        PodSecurityPolicy
clusterrolebindings                            rbac.authorization.k8s.io      false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io      false        ClusterRole
rolebindings                                   rbac.authorization.k8s.io      true         RoleBinding
roles                                          rbac.authorization.k8s.io      true         Role
priorityclasses                   pc           scheduling.k8s.io              false        PriorityClass
csidrivers                                     storage.k8s.io                 false        CSIDriver
csinodes                                       storage.k8s.io                 false        CSINode
storageclasses                    sc           storage.k8s.io                 false        StorageClass
volumeattachments                              storage.k8s.io                 false        VolumeAttachment
```

#### pause容器

- 镜像非常小，目前在 700KB 左右
- 永远处于 Pause (暂停) 状态
- 在 pod 中担任 Linux 命名空间共享的基础；
- 启用 pid 命名空间，开启 init 进程。

完成pod里的多个容器的网络共享：使用统一个网络、同一个ip

参考[Pause 容器](https://jimmysong.io/kubernetes-handbook/concepts/pause-container.html)或[The Almighty Pause Container](https://www.ianlewis.org/en/almighty-pause-container)



### 后置检查检查

确保对应点docker、kubelet服务正常启动，防火墙关闭，swap分区关闭(防止kubelet启动失败)、端口没有占用。查看对应服务日志，例如`journalctl -xeu kubelet`

### 删除集群

在所有节点上执行以下动作：

- 重置集群
- 删除对应配置

```shell
# reset 
kubeadm reset

# delete related files
rm -rf /var/lib/kubelet
rm -rf /etc/kubernetes
rm -rf  /etc/cni/net.d
```

```shell
[root@master ~]# kubeadm reset
[reset] Reading configuration from the cluster...
[reset] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
W0517 17:57:47.281074   10646 reset.go:99] [reset] Unable to fetch the kubeadm-config ConfigMap from cluster: failed to get config map: configmaps "kubeadm-config" not found
[reset] WARNING: Changes made to this host by 'kubeadm init' or 'kubeadm join' will be reverted.
[reset] Are you sure you want to proceed? [y/N]: y
[preflight] Running pre-flight checks
W0517 17:58:00.257625   10646 removeetcdmember.go:79] [reset] No kubeadm config, using etcd pod spec to get data directory
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
[reset] Deleting contents of config directories: [/etc/kubernetes/manifests /etc/kubernetes/pki]
[reset] Deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]
[reset] Deleting contents of stateful directories: [/var/lib/etcd /var/lib/kubelet /var/lib/dockershim /var/run/kubernetes /var/lib/cni]

The reset process does not clean CNI configuration. To do so, you must remove /etc/cni/net.d

The reset process does not reset or clean up iptables rules or IPVS tables.
If you wish to reset iptables, you must do so manually by using the "iptables" command.

If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)
to reset your system's IPVS tables.

The reset process does not clean your kubeconfig files and you must remove them manually.
Please, check the contents of the $HOME/.kube/config file.
```

## k8s practice

### 断电重启问题

表明上提示`The connection to the server master:6443 was refused - did you specify the right host or port?`，最终实际上是因为apiserver无法连接etcd导致容器退出。


### 创建configMap 扩容deploy


```shell
# kc create configmap --help查看
# 生成configmap文件
kubectl create configmap --dry-run=true saber.yml --from-file=./saber.yml --output yaml

kubectl create configmap --dry-run=true pool.conf --from-file=./nginx.conf --output yaml > bridgeconfig.yml

#扩容
kc scale deploy  nginx -n nginx-deploy --replicas=6

# 编辑部署文件
kc edit deploy nginx-deploy -n nginx-deploy
```

### Troubleshooting a Deployment

部署deployment后，pod一直未创建。处理流程——

- `kc get deploy -n nspace`查看部署信息
- `kc describe deploy deploy-name`查看描述信息，只显示` Normal  ScalingReplicaSet  8m48s  deployment-controller  Scaled up replica set oops-ssh-server-sshd-74cd68dd8 to 1`
- `kc get replicaset -n nspace`查看对应的replicaset信息
- `kc describe replicaset name-of-replicaset`可以从events中看到对应的提示信息，例如`serviceaccount "oops-ssh-server" not found`

### 证书管理

通过 kubeadm 安装的 Kubernetes，所有证书都存放在 `/etc/kubernetes/pki` 目录下

- [手动轮换 CA 证书](https://kubernetes.io/zh/docs/tasks/tls/manual-rotation-of-ca-certificates/)
- [PKI 证书和要求](https://kubernetes.io/zh/docs/setup/best-practices/certificates)


### 安装lxcfs服务

```shell
# 安装
yum -y install fuse-devel fuse  lxc-templates
wget  --no-check-certificate  https://copr-be.cloud.fedoraproject.org/results/ganto/lxc3/epel-7-x86_64/01041891-lxcfs/lxcfs-3.1.2-0.2.el7.x86_64.rpm
rpm -ivh lxcfs-3.1.2-0.2.el7.x86_64.rpm

systemctl enable lxcfs
systemctl start lxcfs
# 运⾏lxcfs命令，指定 /proc ⽬录所在位置：
mkdir -p /var/lib/lxcfs
lxcfs /var/lib/lxcfs

# 需要确保lxcfs服务正常启动
systemctl status lxcfs 

```

### 强制删除pod 

[Pods stuck in Terminating status](https://stackoverflow.com/questions/35453792/pods-stuck-in-terminating-status)，状态一直处于Terminating，强制删除

```shell
#删除单个pod
kubectl delete pod {podname} -n {namespace}
# 配合必要的 ns参数 -n kube-system
ns=kube-system
for p in $(kubectl get pods -n $ns | grep Terminating | awk '{print $1}'); do kubectl delete pod $p -n $ns --grace-period=0 --force;done
```

提示告警：
>warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.


### deployment的字段含义

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
  labels:
    app: my-nginx # LABEL-A: <--this label is to manage the deployment itself. this may be used to filter the deployment based on this label. 
spec:              
  replicas: 2
  selector:
    matchLabels:
      app: my-nginx    #LABEL-B:  <--  field defines how the Deployment finds which Pods to manage.
  template:
    metadata:
      labels:
        app: my-nginx   #LABEL-C: <--this is the label of the pod, this must be same as LABEL-B
    spec:                
      containers:
      - name: my-nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
        resources:
          limits:
            memory: "128Mi" #128 MB
            cpu: "200m" #200 millicpu (.2 cpu or 20% of the cpu)
```

### 拉取镜像卡住

[Pods get stuck in ContainerCreating state when pulling image takes long #83471](https://github.com/kubernetes/kubernetes/issues/83471) 看起来都遇到过这个问题。现象为kubelet一直处于拉取镜像状态。

- 手动在对应的node节点上使用docker pull拉取对应的镜像OK
- 在对应的node上检查kubelet对应时段的log信息`journalctl -u kubelet --since "2022-04-12 14:00" --until "2022-04-12 14:20"`
- 检查对应的镜像是否过大
- kubelet开启并行下载，同时部署其他的镜像是否也会出现卡住现象？`--serialize-image-pulls=false`


### 问题排查流程

- 获取非running状态的pod   
`kc get pods -n qa-test |grep -v Running`

- 查看问题pod状态：查看对应事件  
`kc describe pod pod-name -n group-name `

- 查看pod的log信息  
`kc logs pod-name -n group-name `

- 删除问题pod  
`kc delete pod pod-name -n group-name`



```shell
kc get pod -n medusa |grep oops
kc get daemonset -n medusa-system
kc get daemonset secrets-daemon -n medusa-system -o yaml
kc get daemonset -n medusa-system -o wide
kc get pods -n medusa-system -o wide | grep secrets-daemon
kc get daemonset secrets-daemon -n medusa-system -o yaml 
kc exec -it secrets-daemon-zrg5w -n medusa-system /bin/bash
kc get pod secrets-daemon-zrg5w -n medusa-system -o yaml
kc describe pod secrets-daemon-zrg5w -n medusa-system
kc logs secrets-daemon-zrg5w -n medusa-system  --tail=10

history 10
```

### kubectl cp --help

Copy files and directories to and from containers.

### 获取网关信息： 

kc get pod -n istio-system -o wide | grep frontier
kc get pod -n medusa-system -o wide | grep nginx

### exec 多容器

多个容器可自定义名称，然后指定时使用`-c container-name`即可。例如`kc exec -it pod-name -c container-name -- ls /etc/`

### deployment

平台上的实例名对应集群中的 deployment的name；

`kc describe deployment unit-test-platform -n qa-test`

### display all kubernetes objects

- 列出所有对象 `kubectl api-resources -o wide`
- 查看对象信息 `kc explain name-of-object`
- 常用手册查询： [kubectl cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [k8s API Conventions](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md)
- [ingress](https://kubernetes.io/zh/docs/concepts/services-networking/ingress/)

获取 `api-resources`时，提示`error: unable to retrieve the complete list of server APIs: metrics.k8s.io/v1beta1: the server is currently unable to handle the request`，相当于对于`metrics.k8s.io/v1beta1`这个apigroup无法返回对应的类型。其中包含`nodes`和`pods`

```
NAME                              SHORTNAMES     APIGROUP                       NAMESPACED   KIND
nodes                                            metrics.k8s.io                 false        NodeMetrics
pods                                             metrics.k8s.io                 true         PodMetrics
```

### 部署ingress-nginx

- [工作原理官方介绍](https://kubernetes.github.io/ingress-nginx/developer-guide/code-overview/)
- [工作原理](https://kubernetes.github.io/ingress-nginx/how-it-works/)，这里提到的 “同步循环模式”(synchronization loop pattern)，对应文档为 [replication-controller](https://github.com/coreos/docs/blob/master/kubernetes/replication-controller.md)
- [Nginx-Ingress-Controller启动流程](https://zou.cool/2021/11/24/ingress-nginx-read/)
- [如何编写正确且高效的 OpenResty 应用](https://segmentfault.com/a/1190000017563487)，配合代码理解nginx.conf的内容执行逻辑
- [OpenResty最佳实践](https://moonbingbing.gitbooks.io/openresty-best-practices/content/lua/FFI.html)


#### 1.x VS. 0.x Branch 

This is a breaking change between previous v0.X branch

This release only supports Kubernetes versions >= v1.19. The support for Ingress Object in networking.k8s.io/v1beta is being dropped and manifests should now use networking.k8s.io/v1.



```

# online:
NGINX Ingress controller
  Release:       v0.42.0
  Build:         e98e48d99abd6e65b761a66ed3a6a093f1ed16ec
  Repository:    https://github.com/kubernetes/ingress-nginx
  nginx version: nginx/1.19.6

Prepare for release v0.42.0 Manuel Alejandro de Brito Fontes 2020/12/24 22:47
e98e48d99abd6e65b761a66ed3a6a093f1ed16ec

=======================

# dev 
NGINX Ingress controller
  Release:       v0.49.3
  Build:         7ee28f431ccddc4997ab088eff2698b94a8918a0
  Repository:    https://github.com/kubernetes/ingress-nginx
  nginx version: nginx/1.19.9

Prepare for v0.49.3 release (#7742) Ricardo Katz* 2021/10/4 9:37
7ee28f431ccddc4997ab088eff2698b94a8918a0

```

### informers 

- [client-go](https://github.com/kubernetes/client-go)源码informer文件夹下对应的实现
- [Kubernetes Informer 详解](https://www.kubernetes.org.cn/2693.html)，[informer详解](https://zhuanlan.zhihu.com/p/59660536)——这两篇类似，可以看做入门。说明了整体的实现逻辑。做法使用方式的了解，完全足够。
- [如何高效掌控K8s资源变化？K8s Informer实现机制浅析](https://www.cnblogs.com/tencent-cloud-native/p/15268959.html)最新代码解析
- [informer源码分析](https://jimmysong.io/kubernetes-handbook/develop/client-go-informer-sourcecode-analyse.html)，未读，可参考




## tasks

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
- 执行`sudo kubeadm init --image-repository registry.aliyuncs.com/google_containers --service-cidr=10.96.0.0/12 --pod-network-cidr=192.168.0.0/16 --v=5`开始安装
- 安装成功后，master节点会显示为NotReady状态，查看后因为没有CNI插件，可以安装[Deploying flannel manually](https://github.com/flannel-io/flannel#deploying-flannel-manually)，稍后恢复为ready状态。或者使用[Calico插件](https://projectcalico.docs.tigera.io/getting-started/kubernetes/self-managed-onprem/onpremises)
- 确保宿主机上服务正常启动，如下操作
- 确保swap关闭`sudo swapoff -a`
- join后，部署kube-proxy时提示找不到`/run/systemd/resolve/resolv.conf`文件，从master节点复制一份
- 更多相关信息，参考[官方手册](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

```
# work on ubuntu, debian, and centos 7
# create file /etc/docker/daemon.json with 
{
    "exec-opts": ["native.cgroupdriver=systemd"]
}
# do the following commands
 sudo systemctl daemon-reload
 sudo systemctl restart docker
 sudo systemctl restart kubelet
```

基本上就是踩完一遍坑就可以安装成功了。

#### systemd vs cgroupfs

kubernetes为什么要修改使用systemd？

Kubernetes 现在推荐使用 systemd 来代替 cgroupfs
因为systemd是Kubernetes自带的cgroup管理器，负责为每个进程分配cgroups
但docker的cgroup driver默认是cgroupfs，这样就同时运行有两个cgroup控制管理器
当资源有压力的情况时，有可能出现不稳定的情况。

如果不修改配置，会在kubeadm init时有提示:
```
[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. 
The recommended driver is "systemd". 
Please follow the guide at https://kubernetes.io/docs/setup/cri/
```

但[使用 systemd 作为 cgroup 驱动程序可能导致日志泛滥](https://www.ibm.com/docs/zh/cloud-private/3.2.0?topic=iu-log-flooding-when-using-systemd-as-cgroup-driver)

不影响容器指标，只需配置 rsyslog 来忽略此类泛滥的日志， 在 `/etc/rsyslog.conf` 或 `/etc/rsyslog.d/*.conf`中的rule部分下配置——

```
rawmsg, contains, "libcontainer" ~

# Log all kernel messages to the console.
# Logging much else clutters up the screen.
#kern.*                                                 /dev/console

# Log anything (except mail) of level info or higher.
# Don't log private authentication messages!
*.info;mail.none;authpriv.none;cron.none                /var/log/messages

```

- 每个规则行由两部分组成，selector部分和action部分，这两部分由一个或多个空格或tab分隔，selector部分指定源和日志等级，action部分指定对应的操作。
- selector也由两部分组成，设施和优先级，由点号.分隔。第一部分为消息源或称为日志设施，第二部分为日志级别。

`rawmsg, contains, "libcontainer" ~` 波浪线表示discard丢弃。[参考](https://superuser.com/questions/1269643/why-does-mean-discard-the-messages-that-were-matched-in-the-previous-line)，[官方说明](https://www.rsyslog.com/doc/master/configuration/actions.html#discard-stop)

> 1. Note that in legacy configuration the tilde character “~” can also be used instead of the word “stop”.  
> 2. `rawmsg` is the message as it is received from the network. It has the syslog header, e.g. timestamp and tag. `msg` is just the syslog payload (the MSG field from RFC3164 and RFC5424). If your senders are RFC-compliant, both should differ.

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


下载clash，`gunzip clashxx.gz`解压；下载配置文件；[下载mmdb](https://github.com/Dreamacro/clash/issues/854)；启动` ./clash -d conf -f clash.yml`走代理下载镜像；关闭swap `sudo swapoff -a` 还是提示超时。[参考](https://zhuanlan.zhihu.com/p/46341911)

执行`kubeadm config images list`获取需要安装的镜像列表，然后从国内站点拉取。更方便的方式是init时指定repository的地址`sudo kubeadm init --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers`，更多[参数参考](https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/kubeadm-init/) 

- --pod-network-cidr string， 指明 pod 网络可以使用的 IP 地址段。如果设置了这个参数，控制平面将会为每一个节点自动分配 CIDRs。
- --service-cidr string     默认值："10.96.0.0/12"。为服务的虚拟 IP 地址另外指定 IP 地址段

[关于CIDR](https://www.jianshu.com/p/71220cbcf86e)——Classless Inter-Domain Routing, 无类域间路由。基于变长子网掩码([VLSM](https://www.techtarget.com/searchnetworking/definition/variable-length-subnet-mask))，举例，IP4由32位组成，则`192.255.255.255/12`前12位是地址的网络部分，而最后20位是主机地址。[计算器](https://www.ipaddressguide.com/cidr)可计算合适的ip范围

可避免使用固定的私有地址——RFC1918规定的三类私有地址如下：

- A类：10.0.0.0 - 10.255.255.255（10.0.0.0/8）
- B类：172.16.0.0 - 172.31.255.255（172.16.0.0/12）
- C类：192.168.0.0 - 192.168.255.255（192.168.0.0/16）

```
I0116 12:34:31.850670   95794 checks.go:851] image exists: k8s.gcr.io/kube-apiserver:v1.23.1
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

```

[kubeadm init shows kubelet isn't running or healthy](https://stackoverflow.com/questions/52119985/kubeadm-init-shows-kubelet-isnt-running-or-healthy)

提示Port 6443 is in use。需要先执行`sudo kubeadm reset`，重新执行init命令。终于OK了

```
[certs] Using certificateDir folder "/etc/kubernetes/pki"
...
..
.
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

```shell
kubectl label nodes <your_node> kubernetes.io/role=<your_label>
kubectl label --overwrite nodes <your_node> kubernetes.io/role=<your_new_label>
```

[worker节点加入集群](https://stackoverflow.com/questions/51126164/how-do-i-find-the-join-command-for-kubeadm-on-the-master)

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

官方[sample-controller](https://github.com/kubernetes/sample-controller)有针对各个时期的版本说明。

[2021-05-04版本](http://bingerambo.com/posts/2021/05/k8s%E4%B8%AD%E7%9A%84crd%E5%BC%80%E5%8F%91)流程。依赖[code-generator](https://github.com/kubernetes/code-generator)，也可以参考[K8s Client代码自动生成](http://ljchen.net/2019/06/14/K8s-client%E4%BB%A3%E7%A0%81%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90/)

使用代码生成器生成的api兼容标准的k8s api，操作动作相同。

流程概述——自定义资源文件之后，利用`code-generator`生成客户端代码（处理成类似client-go封装标准资源的逻辑，例如可以get、list、watch等）  

第一、 生成框架代码结构。 

- 最顶层的register.go定义了groupName；
- 在版本号目录下又有三个文件：
  - doc.go定义了版本的包名；
  - types.go定义资源的数据结构；
  - register.go真正用于注册该版本下的资源。

第二、 使用`code-generator`提供的脚本生成客户端代码，后续自己的controller逻辑就可以直接调用这个客户端代码。

```shell
~/gopath/src/k8s.io/project> tree
.
└── pkg
    └── apis
        └── controller
            ├── register.go
            └── v1
                ├── doc.go
                ├── register.go
                └── types.go
# 生成代码   
~$ pwd
~$ /gopath/src/k8s.io
~$ $GOPATH/src/k8s.io/code-generator/generate-groups.sh all \ 
   k8s.io/project/pkg/generated \  # 注意路径
   k8s.io/project/pkg/apis \  
   controller:v1

Generating deepcopy funcs
Generating clientset for controller:v1 at k8s.io/project/pkg/generated/clientset
Generating listers for controller:v1 at k8s.io/project/pkg/generated/listers
Generating informers for controller:v1 at k8s.io/project/pkg/generated/informers             
```


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

自动补全，自动完成Enable kubectl autocompletion

```shell
source <(kubectl completion bash)
alias k=kubectl
complete -F __start_kubectl k

#shell生效
echo 'source <(kubectl completion bash)' >>~/.bashrc
echo 'alias kc=kubectl' >>~/.bashrc
echo 'complete -F __start_kubectl kc' >>~/.bashrc
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


