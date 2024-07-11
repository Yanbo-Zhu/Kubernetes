
# 1 Depolyment 

在kubernetes中，Pod是最小的控制单元，但是kubernetes很少直接控制Pod，一般都是通过Pod控制器来完成的。Pod控制器用于pod的管理，确保pod资源符合预期的状态，当pod的资源出现故障时，会尝试进行重启或重建pod。
sda

在kubernetes中Pod控制器的种类有很多，本章节只介绍一种：Deployment。

> Deployment能够确保Prometheus的Pod能够按照预期的状态在集群中运行，而Pod实例可能随机运行在任意节点上


![](image/Pasted%20image%2020240611160938.png)

## 1.1 命令式操作


```yaml
# 1 命令格式: kubectl create deployment 名称  [参数] 
# 2 --image  指定pod的镜像
# 3 --port   指定端口
# 4 --replicas  指定创建pod数量
# 5 --namespace  指定namespace
[root@master ~]# kubectl run nginx --image=nginx:latest --port=80 --replicas=3 -n dev
deployment.apps/nginx created


# 6 查看创建的Pod
[root@master ~]# kubectl get pods -n dev
NAME                     READY   STATUS    RESTARTS   AGE
nginx-5ff7956ff6-6k8cb   1/1     Running   0          19s
nginx-5ff7956ff6-jxfjt   1/1     Running   0          19s
nginx-5ff7956ff6-v6jqw   1/1     Running   0          19s

# 7 查看deployment的信息
[root@master ~]# kubectl get deploy -n dev
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           2m42s

# 8 UP-TO-DATE：成功升级的副本数量
# 9 AVAILABLE：可用副本的数量
[root@master ~]# kubectl get deploy -n dev -o wide
NAME    READY UP-TO-DATE  AVAILABLE   AGE     CONTAINERS   IMAGES              SELECTOR
nginx   3/3     3         3           2m51s   nginx        nginx:latest        run=nginx

# 10 查看deployment的详细信息
[root@master ~]# kubectl describe deploy nginx -n dev
Name:                   nginx
Namespace:              dev
CreationTimestamp:      Wed, 08 May 2021 11:14:14 +0800
Labels:                 run=nginx
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               run=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max 违规词汇
Pod Template:
  Labels:  run=nginx
  Containers:
   nginx:
    Image:        nginx:latest
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-5ff7956ff6 (3/3 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  5m43s  deployment-controller  Scaled up replicaset nginx-5ff7956ff6 to 3
  
# 11 删除 
[root@master ~]# kubectl delete deploy nginx -n dev
deployment.apps "nginx" deleted
```


## 1.2 配置式操作

创建一个deploy-nginx.yaml，内容如下: 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
```


然后就可以执行对应的创建和删除命令了

创建：kubectl create -f deploy-nginx.yaml
删除：kubectl delete -f deploy-nginx.yaml



# 2 Depolyment部署 和 ReplucaSet副本集


Deployment是对ReplicaSet和Pod更高级的抽象。
它使Pod拥有多副本，自愈，扩缩容、滚动升级等能力。

ReplicaSet(副本集)是一个Pod的集合。
它可以设置运行Pod的数量，确保任何时间都有指定数量的 Pod 副本在运行。 当某个 pod  被删除了,  Kubernetive 会自动生成 pod, 以达到 ReplucaSet 所设置的 pod 数量 


通常我们不直接使用ReplicaSet，而是在Deployment中声明。

```shell
#创建deployment,部署3个运行nginx的Pod
kubectl create deployment nginx-deployment --image=nginx:1.22 --replicas=3

#查看deployment
kubectl get deploy

#查看replicaSet
kubectl get rs 

#删除deployment
kubectl delete deploy nginx-deployment

```

![](image/Pasted%20image%2020230915141737.png)


## 2.1 缩放

- 手动缩放

```sh
#将副本数量调整为5
kubectl scale deployment/nginx-deployment --replicas=5
kubectl get deploy
```

- 自动缩放

自动缩放通过增加和减少副本的数量，以保持所有 Pod 的平均 CPU 利用率不超过 75%。
自动伸缩需要声明Pod的资源限制，同时使用 [Metrics Server](https://github.com/kubernetes-sigs/metrics-server#readme) 服务（K3s默认已安装）。
本例仅用来说明`kubectl autoscale`命令的使用，完整示例参考：[HPA演示](https://kubernetes.io/zh-cn/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)

```sh
#自动缩放
kubectl autoscale deployment/nginx-auto --min=3 --max=10 --cpu-percent=75  // 自动缩放通过增加和减少副本的数量，以保持所有 Pod 的平均 CPU 利用率不超过 75%。

#查看自动缩放
kubectl get hpa

#删除自动缩放
kubectl delete hpa nginx-deployment
```

## 2.2 滚动更新

```sh
#查看版本和Pod
kubectl get deployment/nginx-deployment -owide
kubectl get pods

#更新容器镜像
# 更新某个 depliymen 中 容器 的版本. 
# 比如 更新 deoplyment name 为 nginx-depoly 的 depolyment  . 内涵三个 容器 , 容器名为 nginx, 用的是 nignx 版本名为 1.22 的 image. 我们想要把他 update 成 1.23 
kubectl set image deployment/nginx-deployment nginx=nginx:1.23

#滚动更新
kubectl rollout status deployment/nginx-deployment

#查看过程, 同步官产副本集的状态 
kubectl get rs --watch  // rs 就是 repulicaSet 
```

其实本质上就是新生成一个 副本集,  <mark> 但是旧的副本集并没有被自动删除  </mark>
新的副本集 中container的版本  为1.23

![](image/Pasted%20image%2020230915143349.png)

上面命令下 kubectl set image deployment/nginx-deployment nginx=nginx:1.23, 用 kubectl get rs --watch   时时观察 副本集的状态变换
根据 name 可以观察 右新旧两个副本集, 就副本集中的 容器数量 在不断变化, 新副本集 中 也不断莲花. 就副本集 中 容器数量最后变为 0 0 0 . 最后新的副本集 完全代替的就得副本集 
![](image/Pasted%20image%2020230915143403.png)

## 2.3 版本回滚

nginx-deployment 为 某个 deployment 的name, 前面要加上 deployment/ 才能表明 这个nginx-deployment 为某个 deployment 

```sh
#查看历史版本
kubectl rollout history deployment/nginx-deployment

#查看指定版本的信息
kubectl rollout history deployment/nginx-deployment --revision=2

#回滚到历史版本
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

下面例子中 由 1.23 回滚到了 1.22 版本, 因为之前就是 1.22 版本
![](image/Pasted%20image%2020230915144453.png)

看到 revision=1 , 就是 第一次部署的时候, 是用的 1.22 版本 
![](image/Pasted%20image%2020230915144548.png)

可以看到 有两个副本集, 
第一个副本集 为 版本2 的. 
第二个副本集为 版本1 的, 第二个副本集, 以为已经被第一个副本集取代了 , 所以 内含有的 container 数量变为0 了. 

![](image/Pasted%20image%2020230915144922.png)

回滚, --to-revision=1 就是版本号 
![](image/Pasted%20image%2020230915145106.png)

之后 可以看到  旧的副本集又被启用了,  他的内含有的container 数量变为了 3 . 
![](image/Pasted%20image%2020230915145125.png)

![](image/Pasted%20image%2020230915145658.png)


