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


## start over 

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


### setup 

[download binary](https://www.downloadkubernetes.com/)

[Install kubectl on Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

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


