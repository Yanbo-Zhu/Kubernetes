

命名空间(Namespace)是一种资源隔离机制，将同一集群中的资源划分为相互隔离的组。

命名空间可以在多个用户之间划分集群资源（通过[资源配额](https://kubernetes.io/zh-cn/docs/concepts/policy/resource-quotas/)）。例如我们可以设置开发、测试、生产等多个命名空间。同一命名空间内的资源名称要唯一，但跨命名空间时没有这个要求。 命名空间作用域仅针对带有名字空间的对象，例如 Deployment、Service 等。这种作用域对集群访问的对象不适用，例如 StorageClass、Node、PersistentVolume 等。


Namespace是kubernetes系统中的一种非常重要资源，它的主要作用是用来实现**多套环境的资源隔离**或者**多租户的资源隔离**。

默认情况下，kubernetes集群中的所有的Pod都是可以相互访问的。但是在实际中，可能不想让两个Pod之间进行互相的访问，那此时就可以将两个Pod划分到不同的namespace下。kubernetes通过将集群内部的资源分配到不同的Namespace中，可以形成逻辑上的"组"，以方便不同的组的资源进行隔离使用和管理。

可以通过kubernetes的授权机制，将不同的namespace交给不同租户进行管理，这样就实现了多租户的资源隔离。此时还能结合kubernetes的资源配额机制，限定不同租户能占用的资源，例如CPU使用量、内存使用量等等，来实现租户可用资源的管理。

![](image/Pasted%20image%2020240611153704.png)



# 1 Kubernetes 会创建四个初始命名空间

- `**default**` 默认的命名空间，不可删除，未指定命名空间的对象都会被分配到default中。
- `**kube-system**` Kubernetes 系统对象(控制平面和Node组件)所使用的命名空间。
- `**kube-public**` 自动创建的公共命名空间，所有用户（包括未经过身份验证的用户）都可以读取它。通常我们约定，将整个集群中公用的可见和可读的资源放在这个空间中。
- `**kube-node-lease**` [租约（Lease）](https://kubernetes.io/docs/reference/kubernetes-api/cluster-resources/lease-v1/)对象使用的命名空间。每个节点都有一个关联的 lease 对象，lease 是一种轻量级资源。lease对象通过发送[心跳](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/#heartbeats)，检测集群中的每个节点是否发生故障。

```shell
[root@master ~]# kubectl  get namespace
NAME              STATUS   AGE
default           Active   45h     #  所有未指定Namespace的对象都会被分配在default命名空间
kube-node-lease   Active   45h     #  集群节点之间的心跳维护，v1.13开始引入
kube-public       Active   45h     #  此命名空间下的资源可以被所有人访问（包括未认证用户）
kube-system       Active   45h     #  所有由Kubernetes系统创建的资源都处于这个命名空间
```



# 2 使用多个命名空间

- 命名空间是在多个用户之间划分集群资源的一种方法（通过[资源配额](https://kubernetes.io/zh-cn/docs/concepts/policy/resource-quotas/)）。
    - 例如我们可以设置**开发、测试、生产**等多个命名空间。
- 不必使用多个命名空间来分隔轻微不同的资源。
    - 例如同一软件的不同版本： 应该使用[标签](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/) 来区分同一命名空间中的不同资源。
- 命名空间适用于跨多个团队或项目的场景。
    - 对于只有几到几十个用户的集群，可以不用创建命名空间。
- 命名空间不能相互嵌套，每个 Kubernetes 资源只能在一个命名空间中。


# 3 名称空间在实际开发中如何划分？

- ① 基于环境隔离，如：dev（开发）、test（测试）、prod（生产）等。
- ② 基于产品线隔离，如：前端、后端、中间件、大数据、Android、iOS、小程序等。
- ③ 基于团队隔离，如：企业发展事业部、技术工程事业部、云平台事业部等。

# 4 名称空间的特点

- 名称空间资源隔离、网络不隔离，如：配置文件不可以跨名称空间访问，但是网络访问可以跨名称空间访问。
- 默认情况下，安装 Kubernetes 集群的时候，会初始化一个 `default` 名称空间，用来承载那些没有指定名称空间的 Pod 、Service 、Deployment 等对象。

# 5 名称空间的命名规则

- ① 不能带小数点（`.`）。
- ② 不能带下划线（`_`）。
- ③ 使用数字、小写字母或减号（`-`）组成的字符串。

# 6 管理命名空间​


## 6.1 yaml文件的方式 

vim k8s-namespace.yaml

```yml
apiVersion: v1
kind: Namespace
metadata:
  name:  demo # 名称空间的名字
spec: {}  
status: {}
```


```
# 创建名称空间
kubectl apply -f k8s-namespace.yaml
kubectl delete -f k8s-namespace.yaml
```


---


创建pod 的时候 指定 指定自定义的名称空间（yaml）


vim k8s-pod.yaml

```yml

apiVersion: v1
kind: Namespace
metadata:
  name:  demo # 名称空间的名字
spec: {} # 默认为空，其实可以不写
status: {} # 默认为空，其实可以不写

# 以上是 namespace 
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: demo # 指定自定义的名称空间，如果不写，默认为 default
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    resources: # 后面会讲
      limits:
        cpu: 200m
        memory: 500Mi
      requests:
        cpu: 100m
        memory: 200Mi
    ports:
    - containerPort:  80
      name:  http
    volumeMounts:
    - name: localtime
      mountPath: /etc/localtime
  volumes:
    - name: localtime
      hostPath:
        path: /usr/share/zoneinfo/Asia/Shanghai
  restartPolicy: Always 

  # 以上的 Pod

```


kubectl apply -f k8s-pod.yaml

## 6.2 命令行的方式

```sh
#创建命名空间
kubectl create namespace dev
#查看命名空间
kubectl get ns|namespace

#在命名空间内运行Pod
kubectl run nginx --image=nginx --namespace=dev
kubectl run my-nginx --image=nginx -n=dev

#查看命名空间内的Pod
kubectl get pods -n=dev

#查看命名空间内所有对象
kubectl get all
# 删除命名空间会删除命名空间下的所有内容
kubectl delete ns dev
```

 1 **查看**

```shell
# 1 查看所有的ns  命令：kubectl get ns
[root@master ~]# kubectl get ns
NAME              STATUS   AGE
default           Active   45h
kube-node-lease   Active   45h
kube-public       Active   45h     
kube-system       Active   45h     

# 2 查看指定的ns   命令：kubectl get ns ns名称
[root@master ~]# kubectl get ns default
NAME      STATUS   AGE
default   Active   45h

# 3 指定输出格式  命令：kubectl get ns ns名称  -o 格式参数
# kubernetes支持的格式有很多，比较常见的是wide、json、yaml
[root@master ~]# kubectl get ns default -o yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2021-05-08T04:44:16Z"
  name: default
  resourceVersion: "151"
  selfLink: /api/v1/namespaces/default
  uid: 7405f73a-e486-43d4-9db6-145f1409f090
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
  
# 4 查看ns详情  命令：kubectl describe ns ns名称
[root@master ~]# kubectl describe ns default
Name:         default
Labels:       <none>
Annotations:  <none>
Status:       Active  # Active 命名空间正在使用中  Terminating 正在删除命名空间

# ResourceQuota 针对namespace做的资源限制
# LimitRange针对namespace中的每个组件做的资源限制
No resource quota.
No LimitRange resource.
```

2 **创建**

```shell
# 创建namespace
[root@master ~]# kubectl create ns dev
namespace/dev created
```

3  **删除**

```shell
# 删除namespace
[root@master ~]# kubectl delete ns dev
namespace "dev" deleted
```


4 切换当前命名空间

```sh
#查看当前上下文
kubectl config current-context

#将dev设为当前命名空间，后续所有操作都在此命名空间下执行。
kubectl config set-context $(kubectl config current-context) --namespace=dev
```


# 7 重点（后面讲）

● 当我们创建一个 Service 的时候，Kubernetes 会创建一个相应的 DNS 条目。
● 该条目的形式是`<service-name>.<namespace-name>.svc.cluster.local`，这意味着如果容器中只使用`<服务名称>`，它将被解析到本地名称空间的服务器。这对于跨多个名字空间（如开发、测试和生产） 使用相同的配置非常有用。如果你希望跨名字空间访问，则需要使用完全限定域名（FQDN）。

注意事项
大多数的 Kubernetes 资源（如：Pod、Service、副本控制器等）都位于某些名称空间中，但是名称空间本身并不在名称空间中，而且底层资源（如：node 和持久化卷）不属于任何命名空间。

查看在名称空间中的资源：
kubectl api-resources --namespaced=true

查看不在名称空间中的资源：
kubectl api-resources --namespaced=false

