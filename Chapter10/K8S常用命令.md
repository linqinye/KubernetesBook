# K8S常用命令

```
#部署应用
kubectl apply -f httpd.yml
#了解信息
kubectl get deployment httpd -o wide
#了解更详细信息
kubectl describe deployment <imageName>
#查看副本信息
kubectl get replicaset -o wide
kubectl describe replicaset
#查看副本Pod
kubectl get pod
kubectl describe pod
```