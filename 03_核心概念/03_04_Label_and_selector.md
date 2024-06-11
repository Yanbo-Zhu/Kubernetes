
# 1 概述

- 标签（Label）是附件在 Kubernetes 对象上的一组键值对，其意图是按照对用户有意义的方式来标识 Kubernetes 镀锡，同时，又不对 Kubernetes 的核心逻辑产生影响。标签可以用来组织和选择一组 Kubernetes 对象。我们可以在创建 Kubernetes 对象的时候为其添加标签，也可以在创建以后为其添加标签。每个 Kubernetes 对象可以有多个标签，同一个对象的标签的 key 必须是唯一的，如：

**标签（Labels）** 是附加到对象（比如 Pod）上的键值对，用于补充对象的描述信息。
标签使用户能够以松散的方式管理对象映射，而无需客户端存储这些映射。
由于一个集群中可能管理成千上万个容器，我们可以使用标签高效的进行选择和操作容器集合。


```
metadata: 
  labels:   
	key1: value1   
	key2: value2
```

- 使用标签（Label）可以高效的查询和监听 Kubernetes 镀锡，在 Kubernetes 界面工具（如：Kubernetes DashBoard）和 kubectl 中，标签使用的非常普遍。而那些非标识性的信息应该记录在 `注解（Annotation）` 中。


Label的特点：
- 一个Label会以key/value键值对的形式附加到各种对象上，如Node、Pod、Service等等
- 一个资源对象可以定义任意数量的Label ，同一个Label也可以被添加到任意数量的资源对象上去
- Label通常在资源对象定义时确定，当然也可以在对象创建后动态添加或者删除

可以通过Label实现资源的多维度分组，以便灵活、方便地进行资源分配、调度、配置、部署等管理工作。

一些常用的Label 示例如下：
 - 版本标签："version":"release", "version":"stable"......
 - 环境标签："environment":"dev"，"environment":"test"，"environment":"pro"
 - 架构标签："tier":"frontend"，"tier":"backend"


# 2 为什么要使用标签？

- 使用标签，用户可以按照自己期望的形式组织 Kubernetes 对象之间的结构，而无需对 Kubernetes 有任何修改。

- 应用程序的部署或者批处理程序的部署通常是多维度的（如：多个高可用分区、多个程序版本、多个微服务分层）。管理这些对象的时候，很多时候要针对某一个维护的条件做整体操作，如：将某个版本的程序整体删除。这种情况下，如果用户能够事先规划好标签的使用，再通过标签进行选择，就非常的便捷。

- 标签的例子有：
    - `release: stable`、`release: canary`。
    - `environment: dev`、`environment: qa`、`environment: production`。
    - `tier: frontend`、`tier: backend`、`tier: cache`。
    - `partition: customerA`、`partition: customerB`。
    - `track: daily`、`track: weekly`。

- 上面只是一些使用比较普遍的标签，我们也可以根据自己的情况建立合适的标签。

# 3 标签的语法


- **键的格式：**
    - **前缀**(可选)**/名称**(必须)。
- **有效名称和值：**
    - 必须为 63 个字符或更少（可以为空）
    - 如果不为空，必须以字母数字字符（[a-z0-9A-Z]）开头和结尾
    - 包含破折号`**-**`、下划线`**_**`、点`**.**`和字母或数字

- 标签是一组键值对（key/value），标签的 key 有两个部分：可选的前缀和标签名，通过 `/` 分隔。
- 标签前缀：
    - 标签前缀部分是可选的。
    - 如果指定，必须是一个 DNS 的子域名，如：k8s.eip.work 。
    - 不能多余 253 个字符。
    - 使用 `/` 和标签名分隔。

- 标签名：
    - 标签名部分是必须的。
    - 不能多余 63 个字符。
    - 必须由字母、数字开始和结尾。
    - 可以包含字母、数字、减号（`-`）、下划线（`_`）、小数点（`.`）。

如果省略标签前缀，则标签的 key 就被认为是专属于用户的。Kubernetes 的系统组件（如：kube-scheduler、kube-controller-manager、kube-apiserver、kubectl 或其他第三方组件）向可以的 Kubernetes 对象添加标签的时候，必须指定一个前缀。`kubernetes.io/` 和 `k8s.io/` 这两个前缀是 Kubernetes 核心组件预留的。

- 标签的 value ：
    - 不能多于 63 个字符。
    - 可以为空字符串。
    - 如果不为空，则必须由字母、数字开始和结尾。
    - 如果不为空，可以包含字母、数字、减号（`-`）、下划线（`_`）、小数点（`.`）

# 4 标签的例子 

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels: # 标签
    app: nginx
    environment: prod 
spec:
  containers:
  - name: nginx
    image: nginx
```



[label配置模版](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/#syntax-and-character-set)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: label-demo
  labels: #定义Pod标签
    environment: test
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.22
    ports:
    - containerPort: 80
```

```sh
kubectl get pod --show-labels
kubectl get pod -l environment=test,app=nginx
```


# 5 标签选择器 label selector 

- 通常来讲，会有多个 Kubernetes 对象包含相同的标签。通过使用标签选择器（label selector），用户/客户端可以选择一组对象。标签选择器是 Kubernetes 中最主要的分类和筛选手段。
- **标签选择器** 可以识别一组对象。标签不支持唯一性。 
- 标签选择器最常见的用法是为Service选择一组Pod作为后端。
- Kubernetes 的 api-server 支持两种形式的标签选择器，`equality-based 基于等式的` 和 `set-based 基于集合的` 。
- 标签选择器可以包含多个条件，并使用逗号进行分隔，此时只要满足所有条件的 Kubernetes对象才会被选中。
- 标签的选择条件可以使用多个，此时将多个Label Selector进行组合，使用逗号","进行分隔即可。例如：
    - name=slave，env!=production
    - name not in (frontend)，env!=production
- 目前支持两种类型的选择运算：**基于等值的**和**基于集合的**。




等值选择
```sh
selector:
  matchLabels: # component=redis && version=7.0
    component: redis
    version: 7.0
    
```

**集合选择**
```sh
selector:
  matchExpressions: # tier in (cache, backend) && environment not in (dev, prod)
    - {key: tier, operator: In, values: [cache, backend]}
    - {key: environment, operator: NotIn, values: [dev, prod]}

```


## 5.1 等式的标签选择器

name = slave: 选择所有包含Label中key="name"且value="slave"的对象
env != production: 选择所有包括Label中的key="env"且value不等于"production"的对象
  

- 基于等式的标签选择器，可以使用三种操作符 `=` 、`==` 、`!=`。前两个操作符含义是一样的，都代表相等；后一个操作符代表不相等。

```
# 选择了标签名为 `environment` 且 标签值为 `production` 的Kubernetes对象
kubectl get pods -l environment=production,tier=frontend

# 选择了标签名为 `tier` 且标签值不等于 `frontend` 的对象，以及不包含标签 `tier` 的对象
kubectl get pods -l tier != frontend

# 选择所有包含 `partition` 标签的对象
kubectl get pods -l partition

# 选择所有不包含 `partition` 标签的对象
kubectl get pods -l !partition

```


## 5.2 基于集合标签选择器

name in (master, slave): 选择所有包含Label中的key="name"且value="master"或"slave"的对象
name not in (frontend): 选择所有包含Label中的key="name"且value不等于"frontend"的对象

- 基于集合标签选择器，可以根据标签名的一组值进行筛选。支持的操作符有三种：`in`、`notin`、`exists`。

```
# 选择所有的包含 `environment` 标签且值为 `production` 或 `qa` 的对象
kubectl get pods -l environment in (production, qa)

# 选择所有的 `tier` 标签不为 `frontend` 和 `backend`的对象，或不含 `tier` 标签的对象
kubectl get pods -l tier notin (frontend, backend)

# 选择包含 `partition` 标签（不检查标签值）且 `environment` 不是 `qa` 的对象
kubectl get pods -l partition,environment notin (qa)

```

示例：Job、Deployment、ReplicaSet 和 DaemonSet 同时支持基于等式的选择方式和基于集合的选择方式。

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name:  nginx
  namespace: default
  labels:
    app:  nginx
spec:
  selector:
    matchLabels: # matchLabels 是一个 {key,value} 组成的 map。map 中的一个 {key,value} 条目相当于 matchExpressions 中的一个元素，其 key 为 map 的 key，operator 为 In， values 数组则只包含 value 一个元素。matchExpression 等价于基于集合的选择方式，支持的 operator 有 In、NotIn、Exists 和 DoesNotExist。当 operator 为 In 或 NotIn 时，values 数组不能为空。所有的选择条件都以 AND 的形式合并计算，即所有的条件都满足才可以算是匹配
      app: nginx
    matchExpressions: 
      - {key: tier, operator: In, values: [cache]}
      - {key: environment, operator: NotIn, values: [dev]}
  replicas: 1
  template:
    metadata:
      labels:
        app:  nginx
    spec:
      containers:
      - name:  nginx
        image:  nginx:latest
```



## 5.3 标签选择器的例子 


[Service配置模版](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#type-nodeport)

```sh
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector: #与Pod的标签一致
    environment: test
    app: nginx
  ports:
      # 默认情况下，为了方便起见，`targetPort` 被设置为与 `port` 字段相同的值。
    - port: 80
      targetPort: 80
      # 可选字段
      # 默认情况下，为了方便起见，Kubernetes 控制平面会从某个范围内分配一个端口号（默认：30000-32767）
      nodePort: 30007
```



# 6 标签的操作

## 6.1 命令方式


添加标签
kubectl label pod nginx-pod hello=world

更新标签
kubectl label pod nginx-pod hello=java --overwrite

删除标签 
kubectl label pod nginx-pod hello-


```shell
# 为pod资源打标签
[root@master ~]# kubectl label pod nginx-pod version=1.0 -n dev
pod/nginx-pod labeled

# 为pod资源更新标签
[root@master ~]# kubectl label pod nginx-pod version=2.0 -n dev --overwrite
pod/nginx-pod labeled

# 查看标签
[root@master ~]# kubectl get pod nginx-pod  -n dev --show-labels
NAME        READY   STATUS    RESTARTS   AGE   LABELS
nginx-pod   1/1     Running   0          10m   version=2.0

# 筛选标签
[root@master ~]# kubectl get pod -n dev -l version=2.0  --show-labels
NAME        READY   STATUS    RESTARTS   AGE   LABELS
nginx-pod   1/1     Running   0          17m   version=2.0
[root@master ~]# kubectl get pod -n dev -l version!=2.0 --show-labels
No resources found in dev namespace.

#删除标签
[root@master ~]# kubectl label pod nginx-pod version- -n dev
pod/nginx-pod labeled
```

## 6.2 配置方式

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: dev
  labels:
    version: "3.0" 
    env: "test"
spec:
  containers:
  - image: nginx:latest
    name: pod
    ports:
    - name: nginx-port
      containerPort: 80
      protocol: TCP
```


然后就可以执行对应的更新命令了: kubectl apply -f pod-nginx.yaml

