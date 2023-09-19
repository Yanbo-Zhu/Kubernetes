
# 1 Pod 容器集

**Pod** 是包含一个或多个容器的容器组，是 Kubernetes 中创建和管理的最小对象。

Pod 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。

Pod（就像在鲸鱼荚或者豌豆荚中）是一组（一个或多个） 容器； 这些容器共享存储、网络、以及怎样运行这些容器的声明。 Pod 中的内容总是并置（colocated）的并且一同调度，在共享的上下文中运行。 Pod 所建模的是特定于应用的 “逻辑主机”，其中包含一个或多个应用容器， 这些容器相对紧密地耦合在一起。 在非云环境中，在相同的物理机或虚拟机上运行的应用类似于在同一逻辑主机上运行的云应用。

除了应用容器，Pod 还可以包含在 Pod 启动期间运行的 Init 容器。 你也可以在集群支持临时性容器的情况下， 为调试的目的注入临时性容器。


Pod 有以下特点：

- Pod是kubernetes中**最小的调度单位****（**原子单元**）**，Kubernetes直接管理Pod而不是容器。
- 同一个Pod中的容器总是会被自动安排到集群中的**同一节点**（物理机或虚拟机）上，并且**一起调度**。
- Pod可以理解为运行特定应用的“逻辑主机”，这些容器共享存储、网络和配置声明(如资源限制)。
- 每个 Pod 有唯一的 IP 地址。 **IP地址分配给Pod**，在同一个 Pod 内，所有容器共享一个 IP 地址和端口空间，Pod 内的容器可以使用`localhost`互相通信。

---

例如，你可能有一个容器，为共享卷中的文件提供 Web 服务器支持 (Web Server)，以及一个单独的 "边车 (sidercar)" 容器负责从远端更新这些文件 (File Puller)，如下图所示：

Web Server 和 File Puller, 共享同一个Volumen 里面的文件 



![](https://cdn.nlark.com/yuque/0/2022/svg/28915315/1663812948001-3b8ae10e-1a30-4f56-b465-502201528156.svg)

## 1.1 创建和管理Pod

```shell
# 创建容器 
kubectl run mynginx --image=nginx:1.22   // 使用 nginx 1.22 版本

# 查看Pod
kubectl get pod

# 描述
kubectl describe pod mynginx

# 查看Pod的运行日志
kubectl logs mynginx

# 显示pod的IP和运行节点信息
kubectl get pod -owide

# 使用Pod的ip+pod里面运行容器的端口
curl 10.42.1.3

#在容器中执行
kubectl exec mynginx -it -- /bin/bash
kubectl get po --watch

# -it 交互模式   -it -- /bin/bash, 把 要在交互模式中使用到的命令 写在 -- 后面  
# --rm 退出后删除容器，多用于执行一次性任务或使用客户端
kubectl run mynginx --image=nginx -it --rm -- /bin/bash 

# 删除某个pod
kubectl delete pod mynginx
# 强制删除
kubectl delete pod mynginx --force
```

![](https://cdn.nlark.com/yuque/0/2022/png/28915315/1663814727899-c31f2117-d540-48fc-93dc-e004dbf1abfb.png)


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

