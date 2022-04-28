+++
title = "Prometheus And Grafana"
description = "the way to monitor k8s"
tags = [
    "prometheus",
    "grafana"
]
date = "2022-04-25"
topics = [
    "prometheus",
    "grafana"
]
draft = false
toc = true
+++


## 安装

- [下载prometheus二进制包](https://prometheus.io/download/)
- [下载开源版本grafana二进制包](https://grafana.com/grafana/download?edition=oss)

## 监控

[Prometheus 监控外部 Kubernetes 集群](https://www.qikqiak.com/post/monitor-external-k8s-on-prometheus/)关键步骤，生成对应的证书文件

- 创建 Prometheus 访问 Kubernetes 资源对象的RBAC，官方示例[rbac-setup](https://github.com/prometheus/prometheus/blob/main/documentation/examples/rbac-setup.yml)
- 创建上述资源
- 获取对应的资源： 先查到对应的sa，再根据对应的secrets名称获取到具体的证书和token信息
  - describe出来的信息会只显示字节长度，可使用`kubectl get secret prometheus-token-wj7fb -n kube-mon -o yaml`或` -o jsonpath='{.data}'`方式获取
  - 显示的信息是base64编码的值，解码可使用 `echo encoded-string | base64 --decode`看到原始信息，参考[使用 kubectl 管理 Secret](https://kubernetes.io/zh/docs/tasks/configmap-secret/managing-secret-using-kubectl/) 
- 增加对应的job信息

```yaml
# prom.rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
  namespace: kube-mon
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups:
  - ""
  resources:
  - nodes
  - services
  - endpoints
  - pods
  - nodes/proxy
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - "extensions"
  resources:
    - ingresses
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - configmaps
  - nodes/metrics
  verbs:
  - get
- nonResourceURLs:
  - /metrics
  verbs:
  - get
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: kube-mon
```

操作流程——`nohup ./prometheus --config.file=prometheus.yml &> run.log &` 

```shell
# 部署资源
kubectl apply -f prom.rbac.yaml
#查看资源
kubectl get sa prometheus -n kube-mon -o yaml

#获取证书， 上一步会展示出 secrets的name信息
kubectl describe secret prometheus-token-wj7fb -n kube-mon
kubectl get secret prometheus-token-wj7fb -n kube-mon -o yaml
```

启动prometheus服务：

``

### 证书问题

提示证书无法验证`x509: cannot validate certificate for 192.168.50.200 because it doesn't contain any IP SANs`

- 定位openssl的配置文件目录地址：`openssl version -d`
- 编辑[v3_ca]下面添加：`subjectAltName = IP:IP`地址，例如`subjectAltName = IP:192.168.50.200`
- 生成证书相关文件
- 将上一步对应的域名添加到本地`/etc/hosts`中

这种情况下，在执行完上述操作的机器上使用域名访问对应的服务时，可以使用ssl证书。

```shell
# create cert
openssl genrsa -out ca.key 2048 
openssl req -x509 -new -nodes -key ca.key -subj "/CN=api.k8s.com" -days 5000 -out ca.crt

openssl genrsa -out server.key 2048 
openssl req -new -key server.key -subj "/CN=api.k8s.com" -out server.csr
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 5000

# add to hosts
192.168.50.200 api.k8s.com
```

### kube-state-metrics

[Prometheus 结合 StateMetrics+cAdvisor 监控 Kubernetes 集群服务](http://www.mydlq.club/article/113/)

参考[kube-state-metrics 详解](https://mp.weixin.qq.com/s/176eyFBknzdA5wpiJrxDSg)说明。

参考官方[使用 RBAC 鉴权](https://kubernetes.io/zh/docs/reference/access-authn-authz/rbac/)

- 定义对应的ServiceAccount资源
 - ServiceAccount，定义账户信息
 - ClusterRole，定义对应的角色
 - ClusterRoleBinding，将定义的角色ClusterRole赋予对应的ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kube-state-metrics
  namespace: medusa-system

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: kube-state-metrics
rules:
- apiGroups:
  - ""
  resources:
  - configmaps
  - secrets
  - nodes
  - pods
  - services
  - resourcequotas
  - replicationcontrollers
  - limitranges
  - persistentvolumeclaims
  - persistentvolumes
  - namespaces
  - endpoints
  verbs:
  - list
  - watch
- apiGroups:
  - extensions
  resources:
  - daemonsets
  - deployments
  - replicasets
  - ingresses
  verbs:
  - list
  - watch
- apiGroups:
  - apps
  resources:
  - daemonsets
  - deployments
  - replicasets
  - statefulsets
  verbs:
  - list
  - watch
- apiGroups:
  - batch
  resources:
  - cronjobs
  - jobs
  verbs:
  - list
  - watch
- apiGroups:
  - autoscaling
  resources:
  - horizontalpodautoscalers
  verbs:
  - list
  - watch
- apiGroups:
  - policy
  resources:
  - poddisruptionbudgets
  verbs:
  - list
  - watch
- apiGroups:
  - certificates.k8s.io
  resources:
  - certificatesigningrequests
  verbs:
  - list
  - watch
- apiGroups:
  - storage.k8s.io
  resources:
  - storageclasses
  verbs:
  - list
  - watch
- apiGroups:
  - autoscaling.k8s.io
  resources:
  - verticalpodautoscalers
  verbs:
  - list
  - watch

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kube-state-metrics
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kube-state-metrics
subjects:
- kind: ServiceAccount
  name: kube-state-metrics
  namespace: medusa-system
```

- 获取部署yaml描述，其中使用到ServiceAccount

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: kubernetes-state-metrics
  name: kubernetes-state-metrics
  namespace: medusa-system
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: kubernetes-state-metrics
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      annotations:
        prometheus.io/scrape: "true"
      creationTimestamp: null
      labels:
        app: kubernetes-state-metrics
    spec:
      containers:
      - image: hub.myrepo.com/kubespray-library/kube-state-metrics:v1.9.7
        imagePullPolicy: IfNotPresent
        name: kubernetes-state-metrics
        ports:
        - containerPort: 8080
          name: http-metrics
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /healthz
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 5
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 5
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      serviceAccount: kube-state-metrics
      serviceAccountName: kube-state-metrics
      terminationGracePeriodSeconds: 30
```

### prometheus增加job

```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - /root/prometheus/rules/*.yml
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]

  - job_name: kubernetes-cadvisor
    scheme: https
    tls_config:
      ca_file: /root/prometheus/pki/ca.crt
    bearer_token_file: /root/prometheus/pki/token
    kubernetes_sd_configs:
    - role: node
      api_server: https://192.168.50.200:6443
      tls_config:
        ca_file: /root/prometheus/pki/ca.crt
      bearer_token_file: /root/prometheus/pki/token
    relabel_configs:
    - source_labels: [__address__]
      modulus: 90
      target_label: __tmp_hash
      action: hashmod
    - source_labels: [__tmp_hash]
      regex: ^([0-9]|1[0-9])$
      action: keep
    - action: labelmap
      regex: __meta_kubernetes_node_label_(.+)
    - target_label: __address__
      replacement: 192.168.50.200:6443
    - source_labels: [__meta_kubernetes_node_name]
      regex: (.+)
      target_label: __metrics_path__
      replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor
    - source_labels: [job]
      action: replace
      target_label: job
      regex: (.+)
      replacement: kubernetes-cadvisor
    metric_relabel_configs:
    - action: labelmap
      regex: pod
      replacement: pod_name
    - action: labelmap
      regex: container
      replacement: container_name


  - job_name: kubernetes-state-metrics
    scheme: http
    tls_config:
      ca_file: /root/prometheus/pki/ca.crt
    bearer_token_file: /root/prometheus/pki/token
    kubernetes_sd_configs:
    - role: pod
      api_server: https://192.168.50.200:6443
      tls_config:
        ca_file: /root/prometheus/pki/ca.crt
      bearer_token_file: /root/prometheus/pki/token
    relabel_configs:
    - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
      action: keep
      regex: true
    - source_labels: [__meta_kubernetes_namespace]
      action: keep
      regex: medusa-system
    - source_labels: [__meta_kubernetes_pod_label_app]
      regex: ^kubernetes-state-metrics
      action: keep
    - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
      action: replace
      target_label: __metrics_path__
      regex: (.+)
    - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
      action: replace
      regex: ([^:]+)(?::\d+)?;(\d+)
      replacement: $1:$2
      target_label: __address__
    - action: labelmap
      regex: __meta_kubernetes_pod_label_(.+)
    - source_labels: [__meta_kubernetes_namespace]
      action: replace
      target_label: kubernetes_namespace
    - source_labels: [__meta_kubernetes_pod_name]
      action: replace
      target_label: kubernetes_pod_name
    - source_labels: [__meta_kubernetes_pod_label_app]
      regex: (.+)
      target_label: job
      replacement: $1
      action: replace
```