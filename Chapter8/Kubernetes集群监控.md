# Kubernetes集群监控

## Weave Scope

## Heapster

K8S原生得集群监控方案

### 部署

```
git clone https://github.com/kubernetes-retired/heapster.git
kubectl apply -f heapster/deploy/kube-config/influxdb/
kubectl apply -f heapster/deploy/kube-config/rbac/heapster-rbac.yaml

kubectl get --namespace=kube-system deployment | grep -e heapster -e monitor
kubectl get --namespace=kube-system service | grep -e heapster -e monitor
kubectl get --namespace=kube-system pod -o wide | grep -e heapster -e monitor
#修改monitoring-grafana为NodePort
kubectl edit --namespace=kube-system service monitoring-grafana
```

访问http://192.168.129.132:30209/

![10](.\images\K8S-10.png)

## Prometheus Operator