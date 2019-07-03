# Rolling Update

滚动更新是一次只更新一小部分副本，成功后再更新更多副本，最终完成所有副本的更新。

***最大好处***

零停机，整个更新过程始终有副本运行，保证业务连续性

修改镜像版本，即更新，再次执行kubectl apply

之前的三个Pod会被三个新的Pod替换掉

每次替换Pod数量是可以定制，K8S提供maxSurge和maxUnavailable来精细控制Pod的替换数量

## maxSurge

控制滚动更新过程副本总数超过DESIRED的上线。默认25%。roundUp(DESIRED+DESIRED*25%)

## maxUnavailable

控制滚动更新过程中，不可用的副本占DESIRED最大比例。默认25%，DESIRED-roundDown(DESIRED*25%)

<font color="red">maxSurge值越大，初始创建的新副本数量就越多；maxUnavailable值越大，初始销毁的旧副本数量旧越多。</font>

定制maxSurge和maxUnavailable

```
spec:
 strategy:
  rollingUpdate:
   maxSurge: 35%
   maxUnavailable: 35%
```



## 回滚

kubectl apply每次更新应用时，k8s会记录下当前的配置，保存为一个revision（版次），这样就可以回滚到特定revision。

默认配置下，K8S只会保留最近的几个revision，可以在Deployment配置文件中通过revisionHistoryLimit属性增加revision数量。

```
kubactl apply -f httpd.v1.yml --record
```

--record的作用是将当前命令记录到revision记录中，

```
#查看revision历史记录
kubectl rollout history deployment httpd
#回滚到某个版本
kubectl rollout undo deployment httpd --to-revision=1
```