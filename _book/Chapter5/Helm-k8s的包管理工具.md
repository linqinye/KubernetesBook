# Helm-k8s的包管理工具

KS8能够很好的组织和编排容器，但它缺少一个更高层次得应用打包工具，Helm就是干这件事

Mysql服务

1.Service,让外界能够访问到MYSQL

```
apiversion: v1
kind: Service
metadata:
 name: my-mysql
 labels:
  app: my-mysql
spec:
 ports:
 - name: mysql
   port: 3306
   targetPort: mysql
 selector:
   app: my-mysql
```

2.secret,定义mysql密码

```
apiversion: v1
kind: Secret
metadata:
 name: my-mysql
 labels:
  app: my-mysql
type: Opaque
data: 
 mysql-root-password: ""
 mysql-password: ""
```

3.PersistentVolumeClaim为mysql申请持久化存储空间

```
kind: PersistentVolumeClaim
apiversion: v1
metadata:
 name: my-mysql
 labels:
  app: my-mysql
spec:
 accessModes:
  - "ReadWriteOnce"
 resources:
  request:
   storage: "8Gi"
```

4.Deployment,部署mysql Pod

```
apiversion: extensions/v1beta1
kind: Deployment
metadata:
 name: my-mysql
 labels:
  app: my-mysql
spec:
 template:
  metadata:
   labels:
    app: my-mysql
  spec:
   containers:
   - name: my-mysql
     image: "mysql:5.7.14"
     env:
     - name: MYSQL_ROOT_PASSWORD
       volueFrom:
        secretKeyRef:
         name: my-mysql
         key: mysql-root-password
     - name: MYSQL_PASSWORD
       volueFrom:
        secretKeyRef:
         name: my-mysql
         key: mysql-password
     - name: MYSQL_USER
       value: ""
     - name: MYSQL_DATABASE
       value: “”
     ports:
     - name: mysql
       containerPort: 3306
     volumeMounts:
     - name: data
       mountPath: /var/lib/mysql
   volumes:
   - name: data
     persistentVolumeClaim:
       claimName: my-mysql
```

## Helm架构

### chart

创建一个应用得信息集合，包括各种K8S对象得配置模板、参数定义、依赖关系、文档说明等。chart是应用部署得自包含逻辑单元。可以想象成apt、yum中得软件安装包

### release

chart的运行实例，代表一个正在运行的应用。当chart被安装到k8s集群，就生成一个release。chart能够多次安装到同一个集群，每次安装都是一个release



### helm能够做什么

1.从零创建chart

2.与存储chart的仓库交互，拉取、保存和更新chart

3.在k8s集群中安装和卸载release

4.更新、回滚和测试release



## Helm包含组件：

### Helm客户端（管理chart）

终端用户使用的命令行工具

1.在本地开发chart

2.管理chart仓库

3.与Tiller服务器交互

4.在远程K8S集群安装chart

5.查看release信息

6.升级或卸载已有release

### Tiller服务器（管理release）

运行在k8s集群上，会处理Helm客户端请求，与K8S APIserver交互

1.监听来自Helm客户端请求

2.通过chart构建release

3.在K8S中安装chart,并跟踪release状态

4.通过APIserver升级或卸载已有release

## 安装Helm

将Helm客户端安装在能执行kubectl命令的节点上。

```
[docker@k8s-master ~]$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  7028  100  7028    0     0   6316      0  0:00:01  0:00:01 --:--:--  6320
Helm v2.14.0 is already latest
Run 'helm init' to configure helm.
[docker@k8s-master ~]$ helm version
Client: &version.Version{SemVer:"v2.14.0", GitCommit:"05811b84a3f93603dd6c2fcfe57944dfa7ab7fd0", GitTreeState:"clean"}
Error: could not find tiller

wget https://storage.googleapis.com/kubernetes-helm/helm-v2.14.0-linux-amd64.tar.gz
tar xf helm-v2.9.1-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin
rm -rf linux-amd64# 查看版本，不显示出server版本，因为还没有安装serverhelm version
```

安装helm的bash命令补全脚本

```
helm completion bash > .helmrc echo "source .helmrc" >> .bashrc
```

重新登录 可以Tab补全了



## Tiller服务器

```
#删除Tiller
kubectl get all --all-namespaces | grep tiller
kubectl get all -n kube-system -l app=helm -o name|xargs kubectl delete -n kube-system
#提前拉去镜像gcr.io/kubernetes-helm/tiller:v2.14.0
docker pull sapcc/tiller:v2.14.0
docker tag sapcc/tiller:v2.14.0 gcr.io/kubernetes-helm/tiller:v2.14.0
#安装tiller
helm init

kubectl get --namespace=kube-system pod
kubectl describe pod tiller-deploy-f8c677c4-68v78 -n kube-system
#如果仍然因为网络拉取不下来
kubectl edit deploy tiller-deploy -n kube-system
将 image gcr.io/kubernetes-helm/tiller:v2.14.0  替换成   image: sapcc/tiller:v2.14.0
#保存后，kubernetes会自动生效，再次查看pod，已经处于running状态了
kubectl get pod -n kube-system
```

成功标志

```
[docker@k8s-master ~]$ helm version
Client: &version.Version{SemVer:"v2.14.0", GitCommit:"05811b84a3f93603dd6c2fcfe57944dfa7ab7fd0", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.14.0", GitCommit:"05811b84a3f93603dd6c2fcfe57944dfa7ab7fd0", GitTreeState:"clean"}
```



Tiller本身作为容器化应用运行在k8s cluster中

```
kubectl get --namespace=kube-system deployment
kubectl get --namespace=kube-system svc（service）
kubectl get --namespace=kube-system pod

#查看pod错误
kubectl describe pod tiller-deploy-765dcb8745-76vqm -n kube-system

gcr.io/kubernetes-helm/tiller:v2.14.0
```

## 使用Helm

```
#查看当前可安装的chart
helm search
helm repo list
#添加仓库
helm repo add
#安装chart
helm install 软件name（stable/mysql）
```

安装如果权限不足，添加权限

```
[docker@k8s-master ~]$ helm install stable/mysql
Error: no available release name found
[docker@k8s-master ~]$ 

```

添加权限

```
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p'{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```

***解析***

```
[docker@k8s-master ~]$ helm install stable/mysql
#chart本次部署的描述信息
#release的名字，没用-n，因此Helm随机生成一个
NAME:   agile-sheep
LAST DEPLOYED: Tue May 28 21:27:22 2019
#release部署的namespace，默认default，也可以通过--namespace指定
NAMESPACE: default
#DEPLOYED，表示已经将chart部署到集群
STATUS: DEPLOYED

#当前release包含的资源，命名格式为ReleaseName-ChartName
RESOURCES:
==> v1/PersistentVolumeClaim
NAME               STATUS   VOLUME  CAPACITY  ACCESS MODES  STORAGECLASS  AGE
agile-sheep-mysql  Pending  0s

==> v1/Pod(related)
NAME                               READY  STATUS   RESTARTS  AGE
agile-sheep-mysql-d9787fcd4-89ktn  0/1    Pending  0         0s

==> v1/Secret
NAME               TYPE    DATA  AGE
agile-sheep-mysql  Opaque  2     0s

==> v1/Service
NAME               TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)   AGE
agile-sheep-mysql  ClusterIP  10.104.230.48  <none>       3306/TCP  0s

==> v1beta1/Deployment
NAME               READY  UP-TO-DATE  AVAILABLE  AGE
agile-sheep-mysql  0/1    1           0          0s

#release的使用方法
NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
agile-sheep-mysql.default.svc.cluster.local

To get your root password run:

    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default agile-sheep-mysql -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)

To connect to your database:

1. Run an Ubuntu pod that you can use as a client:

    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il

2. Install the mysql client:

    $ apt-get update && apt-get install mysql-client -y

3. Connect using the mysql cli, then provide your password:
    $ mysql -h agile-sheep-mysql -p

To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306

    # Execute the following commands to route the connection:
    export POD_NAME=$(kubectl get pods --namespace default -l "app=agile-sheep-mysql" -o jsonpath="{.items[0].metadata.name}")
    kubectl port-forward $POD_NAME 3306:3306

    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}
```

通过kubectl get查看release各个对象

```
[docker@k8s-master ~]$ kubectl get service agile-sheep-mysql
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
agile-sheep-mysql   ClusterIP   10.104.230.48   <none>        3306/TCP   5m4s
[docker@k8s-master ~]$ kubectl get deployment agile-sheep-mysql
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
agile-sheep-mysql   0/1     1            0           5m18s
[docker@k8s-master ~]$ kubectl get pvc agile-sheep-mysql
NAME                STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
agile-sheep-mysql   Pending                                                     5m46s
```

未准备PersistentVolume，因此当前release不可用

```
#helm list显示已经部署的release
[docker@k8s-master ~]$ helm list
#helm delete可以删除release
[docker@k8s-master ~]$ helm delete agile-sheep
```

## chart

chart是Helm的应用打包格式，由一系列文件组成。可以很简单只部署一个服务，也可以很负责部署一个应用

chart将文件放置在预定义的目录结构中，通常整个chart被打成tar包，而且标注版本信息，便于Helm部署

```
#一旦安装了chart，即可查看tar包
[docker@k8s-master ~]$ ls ~/.helm/cache/archive/
mysql-0.3.5.tgz
```

目录名就是chart名字（不带版本信息）

1.Chart.yaml:描述chart概要，name和version必填

2.README.md:相当于chart使用文档

3.LICENSE:文本文件，描述chart的许可信息

4.requirements.yaml:通过该文件指定chart依赖的其他chart,安装过程中依赖的chart也被一起安装

5.values.yaml:chart支持在安装时根据参数进行定制化配置，而该文件提供 了配置参数的默认值

6.templates目录：各类K8S资源的配置模板都放置在这里，Helm会将values.yaml中的参数值注入模板中，生成标准的YAML配置文件

模板时chart最重要的部分，也是Helm最强大的部分。模板增加了应用部署灵活性

7.templates/NOTES.txt:chart的简易使用文档，chart安装成功后显示此文档内容

可以在该文件中插入配置参数，Helm会动态注入参数值

### chart模板

Go语言模板编写chart

```
{{template "mysql.fullname"}}
```

定义Secret的name。template 作用引入一个子模版。子模版在templates/_helpers.tp1文件中定义

注意：如果存在一些信息多个模板使用，则可在templates/_helpers.tp1中定义子模版，然后通过templates函数引用

**chart和release是Helm预定义的对象，每个对象有自己的属性**

略

```
#查看chart的values.yaml内容
helm inspect values stable/mysql
```

```
[docker@k8s-master ~]$ helm inspect values stable/mysql
## mysql image version
## ref: https://hub.docker.com/r/library/mysql/tags/
##
image: "mysql"
imageTag: "5.7.14"

## Specify password for root user
##
## Default: random 10 character string
# mysqlRootPassword: testing

## Create a database user
##
# mysqlUser:
# mysqlPassword:

## Allow unauthenticated access, uncomment to enable
##
# mysqlAllowEmptyPassword: true

## Create a database
##
# mysqlDatabase:

## Specify an imagePullPolicy (Required)
## It's recommended to change this to 'Always' if the image tag is 'latest'
## ref: http://kubernetes.io/docs/user-guide/images/#updating-images
##
imagePullPolicy: IfNotPresent

livenessProbe:
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  successThreshold: 1
  failureThreshold: 3

readinessProbe:
  initialDelaySeconds: 5
  periodSeconds: 10
  timeoutSeconds: 1
  successThreshold: 1
  failureThreshold: 3

## Persist data to a persistent volume
persistence:
  enabled: true
  ## database data Persistent Volume Storage Class
  ## If defined, storageClassName: <storageClass>
  ## If set to "-", storageClassName: "", which disables dynamic provisioning
  ## If undefined (the default) or set to null, no storageClassName spec is
  ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
  ##   GKE, AWS & OpenStack)
  ##
  # storageClass: "-"
  accessMode: ReadWriteOnce
  size: 8Gi

## Configure resource requests and limits
## ref: http://kubernetes.io/docs/user-guide/compute-resources/
##
resources:
  requests:
    memory: 256Mi
    cpu: 100m

# Custom mysql configuration files used to override default mysql settings
configurationFiles:
#  mysql.cnf: |-
#    [mysqld]
#    skip-name-resolve


## Configure the service
## ref: http://kubernetes.io/docs/user-guide/services/
service:
  ## Specify a service type
  ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services---service-types
  type: ClusterIP
  port: 3306
  # nodePort: 32000
```

事先创建好相应PV->mysql-pv.yml

```
apiversion: v1
kind: PersistentVolume
metadata:
 name: mysql-pv
spec:
 accessModes:
  - ReadWriteOnce
 capacity:
  storage: 8Gi
 persistentVolumeReclaimPolicy: Retain
 nfs:
  path: /nfsdata/mysql-pv
  server: 192.168.56.105
```

```
kubectl apply -f mysql-pv.yml
kubectl get pv
```

### 定制化安装chart

#### Helm有两种方式传递配置参数

1.指定自己的values文件。通常做法

首先通过helm inspect values mysql > myvalues.yaml生成values文件

设置mysqlRootPassword

最后执行helm install --values=mysqlvalues.yaml mysql

2.通过--set直接传入参数

```
#-n即设置release为my,各类资源名称即为my-mysql
helm install stable/mysql --set mysqlRootPassword=abc123 -n my
#通过helm list和helm status my查看chart最新状态

```

### 升级和回滚release

通过--values或--set应用新的配置。升级mysql到5.7.15

```
helm upgrade --set imageTag=5.7.15 mystable/mysql
kubectl get deployment my-mysql -o wide
```

```
#查看release所有版本
helm history my
#回滚到任何版本
helm rollback my 1
```



### 开发自己的chart

#### 创建chart

```
#helm会帮我们创建目录，并生成各类chart文件
helm create mychart
```

新建的chart默认包含一个nginx应用示例

#### 调试chart

Helm提供了debug工具

helm lint:检测chart语法，报告错误以及给出建议

```
helm lint mychart
```

helm install --dry-run --debug：模拟安装安装chart，并输出每个模板生成的YAML内容

```
helm install --dry-run mychart --debug
```

#### 安装chart

Helm支持四种安装方法

1.安装仓库中的chart

```
helm install stable/nginx
```

2.通过tar包安装

```
helm install ./nginx-1.2.3.tgz
```

3.通过chart本地目录安装

```
helm install ./nginx
```

4.通过URL安装

```
helm install https://example.com/charts/nginx-1.2.3.tgz
```

#### 将chart添加到仓库

任何HTTP Server都可以用作chart仓库

在k8s-node1上搭建仓库

1.在node1上启动httpd容器

```
mkdir /var/www
docker run -d -p 8080:80 -v /var/www/:/usr/local/apache2/htdocs/ httpd
```

2.helm package将mychart打包

```
helm package mychart
```

3.生成仓库的Index文件

```
mkdir myrepo
mv mychart-0.1.0.tgz myrepo/
helm repo index myrepo/ --url http://node1IP:8080/charts
ls myrepo/
```

Helm会扫描myrepo目录中的所有tgz包并生成index.yaml

--url指定的是新仓库的访问路径。

新生成的index.yaml记录了当前仓库下所有chart的信息

4.将mychart-0.1.0.tgz和index.yaml上传到k8s-node1的/var/www/charts目录

5.通过helm repo add将新仓库添加到Helm

```
helm repo add newrepo http://node1IP:8080/charts
helm repo list
```

6.除了newrepo/mychart还有一个local/mychart，在执行2打包时mychart也被同步到local的仓库

```
helm search mychart
```

7.直接从新仓库安装mychart

```
helm install newrepo/mychart
```

8.若以后添加了新的chart,可通过helm repo update更新本地的index.相当于apt-get update

```
helm repo update
```

