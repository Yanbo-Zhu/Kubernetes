

# 1 什么是 Kubernetes 的对象

- [官网](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/kubernetes-objects/)。

- Kubernetes 里面操作的资源实体，就是 Kubernetes 的对象，可以使用 yaml 来声明，然后让 Kubernetes 根据 yaml 的声明创建出这个对象。
- 操作 Kubernetes 对象，无论是创建、修改还是删除，都需要使用 [Kubernetes 的 API](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md) 。如：当使用 kubectl 命令行的时候，CLI 会执行必要的 Kubernetes API 调用。
- Kubernetes 对象指的是 Kubernetes 系统的持久化实体，所有的这些 Kubernetes 对象，就代表了当前集群中的实际情况。常规的应用中，我们将应用程序产生的数据存放在数据库中；同理，Kubernetes 将其数据以 Kubernetes 对象的形式通过 api-server 存储在 etcd 中。具体来说，这些数据（Kubernetes 对象）描述了：
    - 集群中运行了那些容器化应用程序，以及在哪个节点上运行。
    - 集群中应用程序可用的资源，如：网络、存储等。
    - 应用程序相关的策略定义，如：重启策略、升级策略和容错策略等。
    - 其他 Kubernetes 管理应用程序时所需要的信息。

- 每一个 Kubernetes 对象都包含了两个重要字段：
    - `spec`：需要由我们来提供，描述了我们对该对象所期望的 `目标状态` 。
    - `status`：只能由 Kubernetes 系统来修改，描述了该对象在 Kubernetes 系统中的实际状态。

- Kubernetes 通过对应的`控制器（如：Deployment 等）`，不断的使得实际状态趋向于我们期望的目标状态。



示例：查看 Deployment 时在系统底层运行的 yaml
kubectl create deployment my-nginx --image=nginx -o yaml
kubectl get deployment my-nginx -o yaml


```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2022-03-11T06:39:04Z"
  generation: 1
  labels:
    app: my-nginx
  name: my-nginx
  namespace: default
  resourceVersion: "21848"
  uid: 0919fb61-2543-49be-a6cc-9f4d41add783
spec: # 期望的状态
  progressDeadlineSeconds: 600
  replicas: 1 # 副本数量
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: my-nginx
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: my-nginx
    spec:
      containers:
      - image: nginx  # 使用这个镜像创建容器
        imagePullPolicy: Always
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status: # 当前状态
  conditions:
  - lastTransitionTime: "2022-03-11T06:39:04Z"
    lastUpdateTime: "2022-03-11T06:39:04Z"
    message: Deployment does not have minimum availability.
    reason: MinimumReplicasUnavailable
    status: "False"
    type: Available
  - lastTransitionTime: "2022-03-11T06:39:04Z"
    lastUpdateTime: "2022-03-11T06:39:04Z"
    message: ReplicaSet "my-nginx-6b74b79f57" is progressing.
    reason: ReplicaSetUpdated
    status: "True"
    type: Progressing
  observedGeneration: 1
  replicas: 1
  unavailableReplicas: 1  # 当前集群不可用
  updatedReplicas: 1
```


- 【问】：Kubernetes 是如何保证最终一致的？
- 【答】：
    - etcd 保存的创建资源期望的状态以及最终这个资源的状态，这两种状态需要最终一致，即：spec 和 status 要最终一致。
    - 当输入 `kubectl create deployment my-nginx --image=nginx` 命令的时候，api-server 保存到 etcd ，controller-manager 会解析数据，知道集群中需要 my-nginx ，保存到 etcd 中。
    - kubelet 在底层不停的死循环使得 spec 状态和 status 最终一致，如果 spec.replicas != status.replicas ，就启动 Pod。

```java
 while(true){   
     if(status.replicas != spec.replicas){     
         kubelet.startPod();   
     }
 }
```


# 2 描述 Kubernetes 对象

在创建的 Kubernetes 对象所对应的 `yaml`文件中，需要配置的字段如下：

- `**apiVersion**` - Kubernetes API 的版本
- `**kind**` - 对象类别，例如`Pod`、`Deployment`、`Service`、`ReplicaSet`等
- `**metadata**` - 描述对象的元数据，包括一个 name 字符串、UID 和可选的 namespace
- `**spec**` - 对象的配置


- 当我们在 Kubernetes 集群中创建一个对象的时候，我们必须提供：
    - 该对象的 `spec` 字段，通过该字段描述我们`期望的目标状态`。
    - 该对象的一些基本信息，如：名字等。
- 可以使用 kubectl 命令行创建对象，也可以使用 yaml 格式的文件进行创建。


**掌握程度：**

- 不要求自己会写
- 找模版
- 能看懂
- 会修改
- 能排错


```yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name:  nginx-deployment
  namespace: default
  labels:
    app:  nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app:  nginx
    spec:
      containers:
      - name:  nginx
        image:  nginx
        ports:
        - containerPort:  80
    - 
```


使用 kubectl apply 命令进行部署：
kubectl apply -f deployment.yaml

使用 kubectl delete 命令进行删除：
kubectl delete -f deployment.yaml

# 3 Kubernetes 对象的 yaml 格式

![](image/2.webp)

- 在上述的 yaml 文件中，如下的字段是必须填写的：
    - `apiVersion`：用来创建对象时所需要的 Kubernetes API 的版本，可以通过 kubectl api-resources 查询。
    - `kind`：创建对象的类型，可以通过 kubectl api-resources 查询。
    - `metadata`：用来唯一确定该对象的元数据，包括 name 、namespace 等，如果 namespace 为空，则默认值为 default 。
    - `spec`：翻译为 `规格` ，表示我们对该对象的期望状态。

status 不需要编写，那是 Kubernetes 实际运行过程中将变化的记录保存在此字段中。
- 不同类型的 Kubernetes ，其 spec 对象的格式不同（含有不同的内嵌字段），通过 [API 手册](https://kubernetes.io/docs/reference/#api-reference)可以查看 Kubernetes 对象的字段和描述。
- 当然，我们以后也可以通过[此地址](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.21/)来参照。


# 4 实际中如何创建 Kubernetes 对象的 yaml

① 如果 Kubernetes 集群中已经存在了要创建的对象，那么可以使用 kubectl get 直接输出 yaml ，然后去除 status 即可：
kubectl get pod xxx -o yaml > demo.yaml

② 如果 Kubernetes 集群中不存在了要创建的对象，那么可以使用类似 kubectl run xxx --dry-run=client  输出 yaml ：
```
# --dry-run=client 表示客户端干跑
kubectl run nginx-pod --image=nginx --dry-run=client -o yaml > demo.yaml
```



# 5 注解 annotations

注解（annotation） 可以用来向 Kubernetes 对象的 meta.annotations 字段添加任意的信息。Kubernetes 的客户端或者自动化工具可以存取这些信息以实现自定义的逻辑。

```
metadata: 
  annotations:   
	key1: value1   
	key2: value2
```

# 6 使用声明式对象配置模拟上述效果


## 1.1 使用yaml定义一个`Pod`

[Pod配置模版](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/#using-pods)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.22
    ports:
    - containerPort: 80
```

使用yaml文件管理对象
```kubectl
#创建对象
kubectl apply -f my-pod.yaml
#编辑对象
kubectl edit nginx
#删除对象
kubectl delete -f my-pod.yaml

```


## 6.1 部署一个 Deployment


vim deployment.yaml

```yaml

apiVersion: apps/v1	#与k8s集群版本有关，使用 kubectl api-versions 即可查看当前集群支持的版本
kind: Deployment	#该配置的类型，我们使用的是 Deployment
metadata:	        #译名为元数据，即 Deployment 的一些基本属性和信息
  name: nginx-deployment	#Deployment 的名称
  labels:	    #标签，可以灵活定位一个或多个资源，其中key和value均可自定义，可以定义多组，目前不需要理解
    app: nginx	#为该Deployment设置key为app，value为nginx的标签
spec:	        #这是关于该Deployment的描述，可以理解为你期待该Deployment在k8s中如何使用
  replicas: 1	#使用该Deployment创建一个应用程序实例
  selector:	    #标签选择器，与上面的标签共同作用，目前不需要理解
    matchLabels: #选择包含标签app:nginx的资源
      app: nginx
  template:	    #这是选择或创建的Pod的模板
    metadata:	#Pod的元数据
      labels:	#Pod的标签，上面的selector即选择包含标签app:nginx的Pod
        app: nginx
    spec:	    #期望Pod实现的功能（即在pod中部署）
      containers:	#生成container，与docker中的container是同一种
      - name: nginx	#container的名称
        image: nginx:1.17	#使用镜像nginx:1.17创建container，该container默认80端口可访问

```


kubectl apply -f deployment.yaml


## 6.2 暴露应用 

vim service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service	#Service 的名称
  labels:     	#Service 自己的标签
    app: nginx	#为该 Service 设置 key 为 app，value 为 nginx 的标签
spec:	    #这是关于该 Service 的定义，描述了 Service 如何选择 Pod，如何被访问
  selector:	    #标签选择器
    app: nginx	#选择包含标签 app:nginx 的 Pod
  ports:
  - name: nginx-port	#端口的名字
    protocol: TCP	    #协议类型 TCP/UDP
    port: 80	        #集群内的其他容器组可通过 80 端口访问 Service
    nodePort: 32600   #通过任意节点的 32600 端口访问 Service
    targetPort: 80	#将请求转发到匹配 Pod 的 80 端口
  type: NodePort	#Serive的类型，ClusterIP/NodePort/LoaderBalancer
  
```


kubectl apply -f service.yaml

