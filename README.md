# prometheus
kubernetes(版本&lt;1.16)部署prometheus;

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
