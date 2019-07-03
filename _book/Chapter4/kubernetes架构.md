# Master节点

cluster的大脑，运行着Daemon服务

## API Server(kube-apiserver)

API Server提供HTTP/HTTPS RESTful API。是kubernetes Cluster的前端接口，各种客户端工具通过它管理Cluster各种资源

## Scheduler（kube-scheduler）

负责决定将Pod放在哪个Node上运行。

## Controller Manager(kube-controller-manager)

负责管理Cluster各种资源，保证资源处于预期状态

不同controller管理不同的资源

## etcd

负责保存Kubernetes Cluster的配置信息和各种资源的状态信息，当数据发生变化，etcd会快速通知相关组件。

## Pod网络

Pod要能够相互通信，Kubernetes Cluster必须部署Pod网络

# Node节点

Node是Pod运行的地方，运行着Kubernetes组件有kubelet、kube-proxy和Pod网络

## kubelet

Node的agent，当Scheduler确定Pod在哪个Node上运行，会将Pod具体配置信息发送给该节点的kubelet，kubelet根据信息创建和运行容器，并向Master报告运行状态

## kube-proxy

service在逻辑上代表了后端的多个Pod,外界通过service访问Pod。

service接收到的请求如何转发Pod是kube proxy负责

每个Node都会运行kube-proxy服务，负责将访问service的TCP/UPD数据流转发到后端的容器，如果存在多副本，kube-proxy实现负载均衡。

## Pod网络



<font color="red">Master上也可以运行应用，即Master也是一个Node，因此Master上也有kubelet和kebu-proxy<br/>几乎所有的Kubernetes组件本身也运行在Pod里。</font>

```
kubectl get pod --all-namespaces -o wide
```

Kubernetes的系统组件都被放到kube-system namespace里。

kube-dns组件为Cluster提供DNS服务，在执行kubeadm init时作为附加组件安装

kubelet是唯一没有以容器形式运行的Kubernetes组件，通过Systemd服务运行。