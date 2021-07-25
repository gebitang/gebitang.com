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

### 术语列表


{{< fluid_imgs
  "pure-u-1-1|https://static001.geekbang.org/resource/image/8e/67/8ee9f2fa987eccb490cfaa91c6484f67.png|Kubernetes项目架构|https://time.geekbang.org/column/article/23132"
>}}

- OCI: Open Container Initiative
- CNI: Container Network Interface
- CRI: Container Runtime Interface
- CSI: Container Storage Interface

使用环境Windows。

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


