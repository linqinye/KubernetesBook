# 通过Service访问Pod

Pod是脆弱的，应用是健壮的

每个Pod都有自己的IP地址，当Controller用新Pod替换故障Pod，IP会随时变化，客户端找到并访问服务。（K8S通过Service）

## 创建Service

多个资源可以在一个yaml定义，用---分割

```
apiVersion: V1
kind: Service
metadate：
#Service名字
 name: httpd-svc
 #指定所属namespace
 namespace: kube-public
spec:
#添加Service类型
 type: NodePort
 selector:
  run: httpd
 ports:
 #将Service的8080映射到Pod的80，使用TCP协议
  protocol: TCP
  #节点上监听的端口
  nodePort: 30000
  #ClusterIP上监听的端口
  port: 8080
  #Pod监听的端口
  targetPort: 80
```

```
kubectl get service
kubectl describe service httpd-svc
```

## Cluster Ip底层实现

Cluster ip是虚拟Ip,由K8S节点上的iptables规则管理

```
iptables-save #打印当前节点iptables规则
```

iptables将访问Service的流量转发到后端Pod,而且使用类似轮询的负载均衡策略

补充：Cluster每个节点都配置相同的iptables规则

## DNS访问Service

```
kubectl get deployment --namespace=kube-system
#Pod与httpd-svc同属default namespace，可忽略default
wget httpd-svc.default:8080
#nslookup查看httpd-svc的DNS信息
nslookup httpd-svc
```

要访问其他namespace中的Service，必须带上namespace

wget httpd-svc.kube-public:8080

```
kubectl get namespace
```

## 外网访问Service

默认ClusterIP

**ClusterIP**

Service通过Cluster内部IP对外提供服务，只有Cluster内的节点和Pod能访问

**NodePort**

Service通过Cluster节点的静态端口对外提供服务。内部通过<NodeIP>:<NodePort>访问Service

**LoadBalancer**