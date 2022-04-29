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


kubernetes-cadvisor
Kube State Metrics


## 安装

- [下载prometheus二进制包](https://prometheus.io/download/)
- [下载开源版本grafana二进制包](https://grafana.com/grafana/download?edition=oss)

二进制安装包，默认配置即可启动。或者修改对应的配置文件 `prometheus.yml`和 `conf\defaults.ini`

## 监控

[Prometheus 结合 StateMetrics+cAdvisor 监控 Kubernetes 集群服务](http://www.mydlq.club/article/113/)

- cAdvisor提供node级别的监控，获取对应服务的cpu、内存等信息
- state-metric可以做进一步的聚合，配置自定义的rule计算规则

### cadvisor

[Prometheus 监控外部 Kubernetes 集群](https://www.qikqiak.com/post/monitor-external-k8s-on-prometheus/)关键步骤，生成对应的证书文件

[Kubernetes 系统组件指标](https://kubernetes.io/zh/docs/concepts/cluster-administration/system-metrics/)，kubelet 会在 /metrics/cadvisor， /metrics/resource 和 /metrics/probes 端点中公开度量值 。对应的监控job：kubernetes-cadvisor 

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

操作流程——

```shell
# 部署资源
kubectl apply -f prom.rbac.yaml
#查看资源
kubectl get sa prometheus -n kube-mon -o yaml

#获取证书， 上一步会展示出 secrets的name信息
kubectl describe secret prometheus-token-wj7fb -n kube-mon
kubectl get secret prometheus-token-wj7fb -n kube-mon -o yaml

#查看secret内容
echo 'encoded-string-of-ca.crt-content' | base64 --decode
 
#保存secret内容到本地
echo 'encoded-string-of-ca.crt-content' | base64 --decode  > ca.crt
```

启动prometheus服务：`nohup ./prometheus --config.file=prometheus.yml &> run.log &` 


### kube-state-metrics

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

### prometheus增加对应的job

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

计算资源还包括自定义的rule，类似——

```yaml
groups:
  - name: apps
    rules:
      # 应用内存使用率 精确到container  guage
      - record: app:memory:total:usage
        expr: |
          sum by (namespace, pod_name, container_name, cluster) (container_memory_rss) /
            sum by (namespace, label_app, label_reference,pod_name,container_name, cluster) (container_spec_memory_limit_bytes > 0) *100  *
              on (pod_name) group_left(label_app,label_reference) label_replace(kube_pod_labels,"pod_name","$1", "pod",  "(.*)" )
      # 应用cpu利用率 精确到container guage
      - record: app:cpu:total:usage
        expr: |
          sum by (namespace, pod_name,container_name, cluster) (rate(container_cpu_usage_seconds_total[1m])) /
            sum by (namespace, label_app,label_reference,pod_name,container_name, cluster) (container_spec_cpu_quota/100000 >0 ) * 100 *
              on (pod_name) group_left(label_app,label_reference) label_replace(kube_pod_labels,"pod_name","$1","pod","(.*)")
      # 2m cpu使用变化率 精确到container guage
      - record: app:cpu_increase_sharply
        expr: |
          increase(container_cpu_usage_seconds_total{container_name!="POD"}[2m]) /
            (container_cpu_usage_seconds_total{container_name!="POD"} offset 2m > 1) * 100 *
              on (pod_name) group_left(label_app,label_reference) label_replace(kube_pod_labels,"pod_name","$1","pod","(.*)")
      # 2m 内存激增, 精确到container guage
      - record: app:memory_increase_sharply
        expr: |
          container_memory_usage_bytes{container_name!="POD"} /
            (quantile_over_time(0.1,container_memory_usage_bytes{container_name!="POD"}[2m])>0)  *  100 *
              on (pod_name) group_left(label_app,label_reference)
                label_replace(kube_pod_labels, "pod_name", "$1", "pod", "(.*)") -100
      # 应用重启 guage
      - record: app:restart_frequently
        expr: |
          increase(kube_pod_container_status_restarts_total[3m])*
            on (pod) group_left(label_app,label_reference) kube_pod_labels
      # 网络流量激增 guage, 由于插件原因 container_name默认都为"POD"
      - record: app:network_increase_sharply
        expr: |
          increase(container_network_receive_bytes_total[2m]) /
            (container_network_receive_bytes_total offset 2m > 1) * 100 *
              on (pod_name) group_left(label_app,label_reference) label_replace(kube_pod_labels,"pod_name","$1","pod","(.*)")
          # 应用cpu使用量 精确到container guage
      - record: app:cpu:total:usage:quantity
        expr: |
          sum by (namespace, pod_name,container_name, cluster) (rate(container_cpu_usage_seconds_total[1m])) *
              on (pod_name) group_left(label_app,label_reference) label_replace(kube_pod_labels,"pod_name","$1","pod","(.*)")
      # 应用内存使用量 精确到container guage
      - record: app:memory:total:usage:quantity
        expr: |
          sum by (namespace, pod_name, container_name, cluster) (container_memory_rss) *
              on (pod_name) group_left(label_app,label_reference) label_replace(kube_pod_labels,"pod_name","$1", "pod",  "(.*)" )

  - name: k8s
    rules:
      # 集群节点接收流量
      - record: k8s:node:container_networt_receive_bytes:rate
        expr: |
          sum by (kubernetes_io_hostname) (rate(container_network_receive_bytes_total{kubernetes_io_hostname!=""}[1m]))
      # 集群节点发送流量
      - record: k8s:node:container_networt_transmit_bytes:rate
        expr: |
          sum by (kubernetes_io_hostname) (rate(container_network_transmit_bytes_total{kubernetes_io_hostname!=""}[1m]))
      # 集群节点内存使用率
      - record: k8s:node:memory_usage
        expr: |
          sum(label_join(container_memory_working_set_bytes{id="/",kubernetes_io_hostname!=""}, "node", "", "kubernetes_io_hostname")) by (node) /
            sum(kube_node_status_allocatable_memory_bytes) by (node) * 100
      # 集群节点cpu使用率
      - record: k8s:node:cpu_usage
        expr: |
          sum (label_join(rate(container_cpu_usage_seconds_total{id="/",kubernetes_io_hostname!=""}[1m]),"node","","kubernetes_io_hostname")) by (node) /
            sum (kube_node_status_allocatable_cpu_cores) by (node) * 100
      # 集群节点file system usage
      - record: k8s:node:fs:usage
        expr: |
          sum by(kubernetes_io_hostname) (container_fs_usage_bytes{device=~"^/dev/[sv]d[a-z][1-9]$",id="/",kubernetes_io_hostname!=""}) /
            sum by(kubernetes_io_hostname) (container_fs_limit_bytes{device=~"^/dev/[sv]d[a-z][1-9]$",id="/",kubernetes_io_hostname!=""}) * 100
      # 集群cpu 使用核数 core
      - record: k8s:cluster:cpu:use:total
        expr: |
          sum  (rate (container_cpu_usage_seconds_total{id="/",kubernetes_io_hostname!=""}[1m]))
      # 集群总cpu 总量 core
      - record: k8s:cluster:cpu:total
        expr: |
          sum (kube_node_status_allocatable_cpu_cores)
      # 集群memory 使用量 bytes
      - record: k8s:cluster:memory_bytes:use:total
        expr: |
          sum(container_memory_working_set_bytes{id="/",kubernetes_io_hostname!=""})
      # 集群memory 总量 bytes
      - record: k8s:cluster:memory_bytes:total
        expr: |
          sum (kube_node_status_allocatable_memory_bytes)
      # 集群file system 使用量
      - record: k8s:cluster:fs:use:total
        expr: |
          sum (container_fs_usage_bytes{device=~"^/dev/[sv]d[a-z][1-9]$",id="/",kubernetes_io_hostname!=""})
      # 集群file system 总量
      - record: k8s:cluster:fs:total
        expr: |
          sum (container_fs_limit_bytes{device=~"^/dev/[sv]d[a-z][1-9]$",id="/",kubernetes_io_hostname!=""})
      # 集群节点 内存使用top 7
      - record: k8s:node:memory:usage:top7
        expr: |
          topk(7, k8s:node:memory_usage)
      # apiserver 访问量qps
      - record: k8s:apiserver:qps:rate5m
        expr: |
          sum(rate(apiserver_request_count[5m])) by (instance)
      # 集群apiserver 5xx
      - record: k8s:apiserver:5xx:sum5m
        expr: |
          sum(increase(apiserver_request_count{code=~"5.."}[5m])) by (instance)
      # 集群 apiserver 资源请求延迟
      - record: k8s:apiserver_request_latencies:quantile:sum5m
        expr: |
          sum by (verb, scope,resource, instance) (rate(apiserver_request_latencies_summary{quantile="0.9"}[5m]))
      # apiserver 资源延迟 0.9 分位
      - record: k8s:apiserver_request_latencies:top10
        expr: |
          topk(10,k8s:apiserver_request_latencies:quantile:sum5m)
      # 集群 pod request memory 总量
      - record: k8s:cluster:pod:request:memory:total
        expr: |
          sum(kube_pod_container_resource_requests_memory_bytes)
      # 集群 pod request cpu 总量
      - record: k8s:cluster:pod:request:cpu:total
        expr: |
          sum(kube_pod_container_resource_requests_cpu_cores)
      # 集群 pod 当前数量
      - record: k8s:cluster:pod:count
        expr: |
                sum(kube_pod_info)
          # 集群 pod 数量使用百分比
                k8s:cluster:pod:count / sum(kube_node_status_allocatable_pods) * 100
      # envoy upstrean 发生连接重置
      - record: k8s:envoy:upstream:rqrx:reset:rate5m:gt0
        expr: |
          sum by (cluster_name) (rate(envoy_cluster_upstream_rq_rx_reset[5m])) > 0
      # envoy 出现连接超时
      - record: k8s:envoy:cx:connect:timeout:rate5m:gt0
        expr: |
          sum by (cluster_name) (rate(envoy_cluster_upstream_cx_connect_timeout[5m])) > 0
      # envoy upstream 连接失败
      - record: k8s:envoy:cx:connect:fail:rate5m:gt0
        expr: |
          sum by (cluster_name) (rate(envoy_cluster_upstream_cx_connect_fail[5m])) > 0
      # envoy 连接重试
      - record: k8s:envoy:cx:connect:attempts:rate5m:gt0
        expr: |
          sum by (cluster_name) (rate(envoy_cluster_upstream_cx_connect_attempts_exceeded[5m]))  > 0

  - name: oap
            - record: oap:http_qps:sum
            expr: |
              sum by (cluster, kubernetes_namespace, app, handler) (irate(med_http_request_total{oap="true"}[2m]))
    # http 新增请求数
            - record: oap:http_incr_1m:sum
              expr: |
                sum by (cluster, kubernetes_namespace, app, handler) (increase(med_http_request_total{oap="true"}[1m]))
    # http 新增异常请求数
            - record: oap:http_err_incr_1m:sum
              expr: |
                sum by (cluster, kubernetes_namespace, app, handler) (increase(med_http_request_total{oap="true", status=~"5.*"}[1m]))
    # http p99延迟
            - record: oap:http_latency_p99:sum
              expr: |
                histogram_quantile(0.99, sum by (cluster, kubernetes_namespace, app, handler, le) (rate(med_http_request_duration_milliseconds_bucket{oap="true"}[2m])))
    # http 平均延迟
            - record: oap:http_latency_avg:sum
              expr: |
```

## 证书问题

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

## grafana 配置

启动后，从前端进行配置。创建dashboard，可将现有的应用监控 导出为json文件，然后导入到新的grafana中。重新编辑对应的DataSource即可。

```json
{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "datasource": "-- Grafana --",
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "type": "dashboard"
      }
    ]
  },
  "editable": true,
  "gnetId": null,
  "graphTooltip": 0,
  "id": 895,
  "iteration": 1651041868586,
  "links": [],
  "panels": [
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "$datasource",
      "decimals": null,
      "description": "(使用核数/申请核数 *100 ) %",
      "fill": 0,
      "gridPos": {
        "h": 9,
        "w": 24,
        "x": 0,
        "y": 0
      },
      "id": 9,
      "legend": {
        "alignAsTable": true,
        "avg": true,
        "current": true,
        "hideZero": false,
        "max": true,
        "min": false,
        "rightSide": true,
        "show": true,
        "sideWidth": null,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "paceLength": 10,
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "sum(app:cpu:total:usage{namespace=\"$namespace\", label_app=\"$app\", pod_name=~\"$deploy.+\", container_name=\"\"}) by (pod_name)",
          "format": "time_series",
          "hide": false,
          "instant": false,
          "intervalFactor": 1,
          "legendFormat": "{{pod_name}}",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "CPU 总使用率",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "percent",
          "label": "CPU(%)",
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": "",
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "$datasource",
      "description": "使用核数",
      "fill": 0,
      "gridPos": {
        "h": 9,
        "w": 24,
        "x": 0,
        "y": 9
      },
      "id": 4,
      "legend": {
        "alignAsTable": true,
        "avg": true,
        "current": true,
        "max": true,
        "min": false,
        "rightSide": true,
        "show": true,
        "sort": null,
        "sortDesc": null,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "paceLength": 10,
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "sum by (namespace, pod_name, cluster) (rate(container_cpu_usage_seconds_total{namespace=\"$namespace\", container_name!=\"\",pod_name=~\"$deploy.*\"}[1m]))  ",
          "format": "time_series",
          "interval": "",
          "intervalFactor": 1,
          "legendFormat": "{{pod_name}}",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "CPU 使用量",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "short",
          "label": "CPU(core)",
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": false
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "$datasource",
      "description": "(使用内存/申请内存 * 100) %",
      "fill": 0,
      "gridPos": {
        "h": 7,
        "w": 24,
        "x": 0,
        "y": 18
      },
      "id": 11,
      "legend": {
        "alignAsTable": true,
        "avg": true,
        "current": true,
        "max": true,
        "min": false,
        "rightSide": true,
        "show": true,
        "sort": "max",
        "sortDesc": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "paceLength": 10,
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "(sum by (pod_name, container) (container_memory_rss{image!=\"\", namespace=\"$namespace\", pod_name=~\"$deploy.+\", container!=\"POD\"}))/(sum by (pod_name, container) (container_spec_memory_limit_bytes{image!=\"\", namespace=\"$namespace\",pod_name=~\"$deploy.+\"})) *100",
          "format": "time_series",
          "hide": false,
          "intervalFactor": 1,
          "legendFormat": "pod: {{ pod_name }} ",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "内存使用率",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "percent",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "$datasource",
      "fill": 0,
      "gridPos": {
        "h": 9,
        "w": 24,
        "x": 0,
        "y": 25
      },
      "id": 5,
      "legend": {
        "alignAsTable": true,
        "avg": true,
        "current": true,
        "max": true,
        "min": false,
        "rightSide": true,
        "show": true,
        "sort": "max",
        "sortDesc": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "paceLength": 10,
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "sum by (pod_name) (container_memory_rss{image!=\"\", namespace=\"$namespace\", pod_name=~\"$deploy.+\"}) ",
          "format": "time_series",
          "hide": false,
          "interval": "",
          "intervalFactor": 1,
          "legendFormat": "{{pod_name}}",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "内存占用",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "bytes",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "$datasource",
      "description": "",
      "fill": 0,
      "gridPos": {
        "h": 7,
        "w": 24,
        "x": 0,
        "y": 34
      },
      "id": 15,
      "legend": {
        "alignAsTable": true,
        "avg": false,
        "current": true,
        "max": true,
        "min": false,
        "rightSide": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "paceLength": 10,
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "sum by (pod_name) (container_threads{image!=\"\", namespace=\"$namespace\", pod_name=~\"$deploy.+\"})",
          "format": "time_series",
          "hide": false,
          "intervalFactor": 1,
          "legendFormat": "{{ pod_name }}",
          "refId": "A"
        }
      ],
      "thresholds": [
        {
          "colorMode": "critical",
          "fill": true,
          "line": true,
          "op": "gt",
          "value": 3000,
          "yaxis": "left"
        }
      ],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "线程（PID）数量",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": "0",
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "$datasource",
      "description": "超出 CPU 限制后被限速的确狂",
      "fill": 0,
      "gridPos": {
        "h": 9,
        "w": 24,
        "x": 0,
        "y": 41
      },
      "id": 14,
      "legend": {
        "alignAsTable": true,
        "avg": false,
        "current": true,
        "max": true,
        "min": false,
        "rightSide": true,
        "show": true,
        "sort": "max",
        "sortDesc": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "paceLength": 10,
      "percentage": false,
      "pointradius": 1,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [
        {
          "alias": "/seconds .+/",
          "color": "#B877D9"
        },
        {
          "alias": "/periods .+/",
          "color": "#FADE2A"
        }
      ],
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "sum by (namespace, pod_name, cluster) (irate(container_cpu_cfs_throttled_seconds_total{namespace=\"$namespace\",pod_name=~\"$deploy.*\"}[30s]))  * on (pod_name) group_left(label_app) label_replace(kube_pod_labels{label_app=\"$app\",namespace=\"$namespace\"},\"pod_name\",\"$1\",\"pod\",\"(.*)\") > 0",
          "format": "time_series",
          "interval": "",
          "intervalFactor": 1,
          "legendFormat": "seconds {{pod_name}}",
          "refId": "A"
        },
        {
          "expr": "sum by (namespace, pod_name, cluster) (irate(container_cpu_cfs_throttled_periods_total{namespace=\"$namespace\",pod_name=~\"$deploy.*\"}[30s]))  * on (pod_name) group_left(label_app) label_replace(kube_pod_labels{label_app=\"$app\",namespace=\"$namespace\"},\"pod_name\",\"$1\",\"pod\",\"(.*)\") > 0",
          "format": "time_series",
          "interval": "",
          "intervalFactor": 1,
          "legendFormat": "periods {{pod_name}}",
          "refId": "B"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "CPU 超限量",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "short",
          "label": "periods",
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": "seconds",
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "$datasource",
      "fill": 0,
      "gridPos": {
        "h": 9,
        "w": 24,
        "x": 0,
        "y": 50
      },
      "id": 6,
      "legend": {
        "alignAsTable": true,
        "avg": false,
        "current": true,
        "max": false,
        "min": false,
        "rightSide": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "paceLength": 10,
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [
        {
          "alias": "/.*trans.*/",
          "transform": "negative-Y"
        }
      ],
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "sum by (pod_name) (rate(container_network_receive_bytes_total{image!=\"\", namespace=\"$namespace\", pod_name=~\"$deploy.+\"}[1m]))",
          "format": "time_series",
          "interval": "",
          "intervalFactor": 1,
          "legendFormat": "recv {{ pod_name }}",
          "refId": "A"
        },
        {
          "expr": "sum by (pod_name) (rate(container_network_transmit_bytes_total{image!=\"\", namespace=\"$namespace\", pod_name=~\"$deploy.+\"}[1m]))",
          "format": "time_series",
          "interval": "",
          "intervalFactor": 1,
          "legendFormat": "trans {{pod_name}}",
          "refId": "B"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "网络流量",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "bytes",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "$datasource",
      "description": "",
      "fill": 0,
      "gridPos": {
        "h": 8,
        "w": 24,
        "x": 0,
        "y": 59
      },
      "id": 13,
      "legend": {
        "alignAsTable": true,
        "avg": false,
        "current": true,
        "max": false,
        "min": false,
        "rightSide": true,
        "show": true,
        "sort": "current",
        "sortDesc": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "paceLength": 10,
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "sum by (hostname,app_name,pod_name)(cloud_app_logs_file_size_kilobytes{namespace=\"$namespace\", app_name=\"$app\"}) / 1024^2",
          "format": "time_series",
          "interval": "",
          "intervalFactor": 1,
          "legendFormat": "{{pod_name}}",
          "refId": "A"
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "日志文件大小",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "transparent": true,
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "decimals": null,
          "format": "decgbytes",
          "label": "size",
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "$datasource",
      "fill": 0,
      "gridPos": {
        "h": 8,
        "w": 24,
        "x": 0,
        "y": 67
      },
      "id": 7,
      "legend": {
        "alignAsTable": true,
        "avg": false,
        "current": true,
        "max": false,
        "min": false,
        "rightSide": true,
        "show": true,
        "sort": "current",
        "sortDesc": false,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "paceLength": 10,
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [
        {
          "alias": "/.*read.*/",
          "transform": "negative-Y"
        }
      ],
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "expr": "sum by (pod_name) (rate(container_fs_writes_bytes_total{image!=\"\", namespace=\"$namespace\", pod_name=~\"$deploy.+\"}[1m]))",
          "format": "time_series",
          "interval": "",
          "intervalFactor": 1,
          "legendFormat": "write {{ pod_name }}",
          "refId": "A"
        },
        {
          "expr": "sum by (pod_name) (rate(container_fs_reads_bytes_total{image!=\"\", namespace=\"$namespace\", pod_name=~\"$deploy.+\"}[1m]))",
          "format": "time_series",
          "interval": "",
          "intervalFactor": 1,
          "legendFormat": "read {{pod_name}}",
          "refId": "B"
        }
      ],
      "thresholds": [
        {
          "colorMode": "critical",
          "fill": true,
          "line": true,
          "op": "gt",
          "yaxis": "left"
        }
      ],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "磁盘 IO",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "format": "bytes",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "cacheTimeout": null,
      "gridPos": {
        "h": 9,
        "w": 12,
        "x": 0,
        "y": 75
      },
      "id": 2,
      "links": [],
      "timeFrom": null,
      "timeShift": null,
      "title": "Panel Title",
      "type": "prometheus-kubernetes-podnav-panel"
    }
  ],
  "refresh": false,
  "schemaVersion": 18,
  "style": "dark",
  "tags": [],
  "templating": {
    "list": [
      {
        "current": {
          "text": "p8s-online",
          "value": "p8s-online"
        },
        "hide": 0,
        "label": "datasource",
        "name": "datasource",
        "options": [],
        "query": "prometheus",
        "refresh": 1,
        "regex": "p8s-(dev|online.*)",
        "skipUrlSync": false,
        "type": "datasource"
      },
      {
        "allValue": null,
        "current": {
          "text": "sale-java",
          "value": "sale-java"
        },
        "datasource": "$datasource",
        "definition": "label_values(kube_deployment_labels, namespace)",
        "hide": 0,
        "includeAll": false,
        "label": "namespace:",
        "multi": false,
        "name": "namespace",
        "options": [],
        "query": "label_values(kube_deployment_labels, namespace)",
        "refresh": 2,
        "regex": "",
        "skipUrlSync": false,
        "sort": 1,
        "tagValuesQuery": "",
        "tags": [],
        "tagsQuery": "",
        "type": "query",
        "useTags": false
      },
      {
        "allValue": null,
        "current": {
          "text": "workphone-app",
          "value": "workphone-app"
        },
        "datasource": "$datasource",
        "definition": "label_values(kube_deployment_labels{namespace=~\"$namespace\"}, label_app)",
        "hide": 0,
        "includeAll": false,
        "label": "app:",
        "multi": false,
        "name": "app",
        "options": [],
        "query": "label_values(kube_deployment_labels{namespace=~\"$namespace\"}, label_app)",
        "refresh": 2,
        "regex": "",
        "skipUrlSync": false,
        "sort": 1,
        "tagValuesQuery": "",
        "tags": [],
        "tagsQuery": "",
        "type": "query",
        "useTags": false
      },
      {
        "allValue": null,
        "current": {
          "text": "workphone-app-workphone-app-deploy-online",
          "value": "workphone-app-workphone-app-deploy-online"
        },
        "datasource": "$datasource",
        "definition": "label_values(kube_deployment_labels{namespace=\"$namespace\", label_app=\"$app\"}, label_reference)",
        "hide": 0,
        "includeAll": false,
        "label": "deploy:",
        "multi": false,
        "name": "deploy",
        "options": [],
        "query": "label_values(kube_deployment_labels{namespace=\"$namespace\", label_app=\"$app\"}, label_reference)",
        "refresh": 2,
        "regex": "",
        "skipUrlSync": false,
        "sort": 0,
        "tagValuesQuery": "",
        "tags": [],
        "tagsQuery": "",
        "type": "query",
        "useTags": false
      }
    ]
  },
  "time": {
    "from": "now-6h",
    "to": "now"
  },
  "timepicker": {
    "refresh_intervals": [
      "1m"
    ],
    "time_options": [
      "5m",
      "15m",
      "1h",
      "6h",
      "12h",
      "24h",
      "2d",
      "7d",
      "30d"
    ]
  },
  "timezone": "",
  "title": "应用监控",
  "uid": "qyp1uiqmz",
  "version": 105
}
```