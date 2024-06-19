
# 1 概述

平时，我们使用 Docker 的镜像非常之简单，类似于下面的命令：
docker pull nginx:latest

其实，完整的写法应该是这样：
docker pull docker.io/library/nginx:latest

如果我们使用自己搭建过 Docker 的私有镜像仓库，可以发现会出现如下的命令
docker push 192.168.65.100:5000/xudaxian/ubuntu:1.0


- 说明：
    - `192.168.65.100`：镜像仓库的地址。
    - `5000`：镜像仓库的端口。
    - `xudaxian`：镜像仓库的名称。
    - `ubuntu`：镜像的名称。
    - `1.0`：镜像的版本，如果不写，默认就是 latest 。
- 当然，如果拉取的是 hub.docker.com 中的镜像，那么镜像仓库地址以及端口都可以省略。

## 1.1 Kubernetes 中的镜像

在 Kubernetes 的 Pod 定义容器的时候，必须指定容器所使用的镜像，容器中的 image 字段支持的语法和 docker 命令是一样的，包括私有镜像仓库和标签，如：

```
# 192.168.65.100:5000/xudaxian/ubuntu:1.0
my-registry.example.com:5000/example/web-example:v1.0
```

注意：在生产环境中，建议锁定镜像的版本。


示例

```yml

apiVersion: v1
kind: Namespace
metadata:
  name:  demo 
spec: {} 
status: {} 

# 以上是 namespace 
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: demo 
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.20.2 # Docker 的镜像名称，和 Docker 命令一样，my-registry.example.com:5000/example/web-example:v1.0，实际开发中，建议锁定镜像的版本。
    ports:
    - containerPort:  80

  # 以上的 Pod

```


## 1.2 Kubernetes 中的镜像拉取策略 imagePullPolicy

- IfNotPresent（默认） ：只有当镜像在本地不存在时才会拉取。
- Always ：每当 kubelet 启动一个容器时，kubelet 会查询容器的镜像仓库， 将名称解析为一个镜像[摘要](https://docs.docker.com/engine/reference/commandline/pull/#pull-an-image-by-digest-immutable-identifier)。 如果 kubelet 有一个容器镜像，并且对应的摘要已在本地缓存，kubelet 就会使用其缓存的镜像； 否则，kubelet 就会使用解析后的摘要拉取镜像，并使用该镜像来启动容器。
- Never ：Kubelet 不会尝试获取镜像。如果镜像已经以某种方式存在本地， kubelet 会尝试启动容器；否则，会启动失败。


```

apiVersion: v1
kind: Namespace
metadata:
  name:  demo 
spec: {} 
status: {} 

# 以上是 namespace 
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: demo 
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.20.2 # Docker 的镜像名称，和 Docker 命令一样，my-registry.example.com:5000/example/web-example:v1.0，实际开发中，建议锁定镜像的版本。
    imagePullPolicy: Always # 镜像拉取策略：IfNotPresent（默认）、Always、Never
    ports:
    - containerPort:  80
  # 以上的 Pod

```


## 1.3 下载私有仓库的镜像

● 我们在使用阿里云容器镜像的私有仓库的使用，阿里云要求我们进行登录，如果是 docker 拉取镜像，那么只需要 docker login 之类的就可以了；但是，如果使用 Kubernetes 该怎么办？


1 创建 secret ：

```yml 

# -n demo ：表示该密钥将只在指定的名称空间 demo 中生效
# docker-registry aliyun ：指定 Docker 镜像仓库的名称
# --docker-server：Docker 镜像仓库的地址
# --docker-username：Docker 镜像仓库的用户名
# --docker-password：Docker 镜像仓库的密码
kubectl create secret -n demo docker-registry aliyun \
       --docker-server=registry.cn-shanghai.aliyuncs.com \
       --docker-username=xudaxian \
       --docker-password=123456
```

2 在 yaml 中拉取镜像的时候设置镜像拉取的密钥（secret）

```yaml

apiVersion: v1
kind: Namespace
metadata:
  name:  demo 
spec: {} 
status: {} 

# 以上是 namespace 
---
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
  namespace: demo 
  labels:
    app: nginx
spec:
  containers:
  imagePullSecrets: # Pull镜像时使用的 secret 名称，以 key：secretkey 格式指定
    - name:  aliyun 
  - name: nginx
    image: nginx:1.20.2 # Docker 的镜像名称，和 Docker 命令一样，my-registry.example.com:5000/example/web-example:v1.0，实际开发中，建议锁定镜像的版本。
    imagePullPolicy: Always # 镜像拉取策略：IfNotPresent（默认）、Always、Never
  - name: arcgis
    image: registry.cn-shanghai.aliyuncs.com/xudaxian/arcgis/v1.0 
    imagePullPolicy: Always

```

> 注意：这个肯定运行不了，需要将 secret 的用户名和密码设置为自己的，而且在拉取阿里云私有镜像的时候设置为自己的镜像。




# 2 环境变量

使用 env 用来给 Pod 中的容器设置环境变量，相当于 docker run -e xxx=xxx 中的 -e  参数。 


vim k8s-mysql.yaml


```
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
  namespace: default
  labels:
    app: mysql-pod
spec:
  containers:
  - name: mysql-pod
    image: mysql:5.7
    env: # 环境变量 相当于 docker run -e  xxx = xxx
    - name: MYSQL_ROOT_PASSWORD # root 的密码
      value: "123456"
    - name:  MYSQL_DATABASE # mysql 的 数据库
      value: ssm
  restartPolicy: Always
```


kubectl apply -f k8s-mysql.yaml


# 3 启动命令

- Docker 的镜像拥有存储镜像信息的相关元数据，如果不设置生命周期命令和参数，容器运行时会运行镜像制作时提供的默认的命令和参数，Docker 原生定义这两个字段为 `ENTRYPOINT` 和 `CMD` 。

- 如果在创建工作负载时填写了容器的运行命令和参数，将会覆盖镜像构建时的默认命令 `Entrypoint` 、`CMD`，规则如下：

![](image/Pasted%20image%2020240611153001.png)

换言之，如果在 Kubernetes 的 yaml 中定义了 comand 和 args ，那么就会覆盖 Dockerfile 中的 ENTRPOINT 和 CMD 。


## 3.1 例子

示例：启动 MySQL 

vim k8s-mysql.yaml

```yml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
  namespace: default
  labels:
    app: mysql-pod
spec:
  containers:
  - name: mysql-pod
    image: mysql:5.7
    env:
    - name: MYSQL_ROOT_PASSWORD
      value: "123456"
    - name: MYSQL_DATABASE
      value: ssm
    args:
      - "--lower_case_table_names=1"
      - "--character-set-server=utf8mb4"
      - "--collation-server=utf8mb4_general_ci"
      - "--default-authentication-plugin=mysql_native_password"    
    ports:
    - containerPort:  3306
  restartPolicy: Always
```

kubectl apply -f k8s-mysql.yaml

  
# 4 资源限额

- 容器中的程序要运行，肯定会占用一定的资源，比如 CPU 和内存等，如果不对某个容器的资源做限制，那么它就可能吃掉大量的资源，导致其他的容器无法运行。
- 针对上面的情况，Kubernetes 提供了对内存和 CPU 的资源进行配额的机制，这种机制主要通过 resources 选项实现，它有两个子选项：
    - limits：用于限制运行的容器的最大占用资源，当容器占用资源超过 limits 时会被终止，并进行重启。
    - requests：用于设置容器需要的最小资源，如果环境资源不够，容器将无法启动。
- cpu：core 数，可以为整数或小数，1 == 1000m。
- memory：内存大小，可以使用 Gi、 Mi、G、M 等形式。



```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.20.2
    resources: # 资源限制
      limits:
        cpu: 2
        memory: 1500M
      requests:
        cpu: 1
        memory: 1024M
    ports:
    - containerPort:  80
  restartPolicy: Always


```
