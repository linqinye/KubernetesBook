# Health Check

自愈的默认实现方法是自动重启发生故障的容器

## 默认的健康检查

每个容器启动时都会执行一个进程，此进程由Dockerfile的CMD或ENTRYPOINT指定。如果进程退出时返回码非零，则认为容器故障。K8S就会根据restartPolicy重启容器

```
#模拟容器启动10秒后发生故障
- sleep 10; exit 1
```



## Liveness探测

让用户可以自定义判断容器是否健康的条件，探测失败，K8S就会重启容器

## Readiness探测

告诉K8S何时将容器加入Service负载均衡池中，对外提供服务



上面两者配置语法类似，yaml文件如下：Liveness将readiness替换即可

```
apiVersion: v1
kind: Pod
meradata:
    labels:
	  test: readiness
	name:readiness
spec:
  restartPolicy: OnFailure
  containers:
  - name: readiness
   image: busybox
   args:
   - /bin/sh
   - -c
   - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
   readinessProbe:
    exec:
     command:
     - cat
     - /tmp/healthy
    initialDelaySeconds: 10
    periodSeconds: 5
```



## 两者探测区别

1.不特意配置，K8S对两者都采用默认行为，即判断容器返回是否为0来判断是否探测成功

2.3次探测失败后，Liveness重启容器，Readiness将容器设置不可用，不接收Service转发的请求

3.两者独立互不依赖，可同时使用，Liveness探测判断容器是否需要重启以实现自愈，Readiness判断容器是否已经准备好对外提供服务。