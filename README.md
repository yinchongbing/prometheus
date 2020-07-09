# prometheus
kubernetes(版本&lt;1.16)部署prometheus;
gpu监控需要集群安装gpu监控插件;nvidia插件[地址](https://github.com/NVIDIA/k8s-device-plugin/tree/v1.11)
promethues相关配置文件[地址](https://github.com/kubernetes/kubernetes/tree/release-1.14/cluster/addons/prometheus);
DCGM相关配置文件[地址](https://github.com/NVIDIA/gpu-monitoring-tools)

## 1.安装kube-state-metrics和node-exporter

```sh
kubectl apply -f kube-state-metrics-rbac.yaml
kubectl apply -f kube-state-metrics-service.yaml
kubectl apply -f kube-state-metrics-deployment.yaml
kubectl apply -f node-exporter-service.yaml
kubectl apply -f node-exporter-ds.yml
```

## 2.安装alertmanager

### 2.1修改alertmanager-pvc.yaml的storageClass为集群已定义的sc

### 2.2依次安装alertmanager

```sh
kubectl apply -f alertmanager-configmap.yaml
kubectl apply -f alertmanager-pvc.yaml
kubectl apply -f alertmanager-service.yaml
kubectl apply -f alertmanager-deployment.yaml
```

## 3.安装prometheus

```sh
kubectl apply -f prometheus-configmap.yaml
kubectl apply -f prometheus-rbac.yaml
kubectl apply -f prometheus-service.yaml
kubectl apply -f prometheus-statefulset.yaml
```
## 4.安装grafana
！！!注意:grafana5.1版本后 groupid 更改了引起的问题，需要更改目录所属用户
请自行创建好grafana所使用的pvc；grafana-chown-job.yaml对应的deployment会去更改pvc目录权限（不然会导致>5.1版本的grafana启动不成功；<5.1版本的请忽略grafana-chown-job.yaml）。

```
kubectl apply -f grafana-chown-job.yaml
kubectl apply -f grafana-service.yaml
kubectl apply -f grafana-deployment2.yaml
```

## 5.gpu监控插件dcgm-exporter安装部署

### 5.1安装dcgm-exporter

```sh
kubectl apply -f dcgm-exporter.yaml
```

### 5.2测试dcgm-exporter暴露的metrics

```sh
curl dcgm-exporter-serviceIp:9400/metrics
# HELP DCGM_FI_DEV_SM_CLOCK SM clock frequency (in MHz).
# TYPE DCGM_FI_DEV_SM_CLOCK gauge
# HELP DCGM_FI_DEV_MEM_CLOCK Memory clock frequency (in MHz).
....
```

### 5.3更改prometheus配置文件，加入job_name: dcgm-exporter

```sh
vim prometheus-configmap.yaml
---
...
data:
  prometheus.yml: |
    scrape_configs:
    - job_name: prometheus
      static_configs:
      - targets:
        - localhost:9090

    - job_name: dcgm-exporter
      metrics_path: /metrics
      static_configs:
      - targets:
        - dcgm-exporter:9400

    - job_name: kubernetes-apiservers
      kubernetes_sd_configs:
...
```

### 5.4重启prometheus server

## 6.访问grafana,导入gpu监控模板dcgm-exporter-dashboard.json即可
