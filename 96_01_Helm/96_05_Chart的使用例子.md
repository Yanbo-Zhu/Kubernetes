

# 1 helm repo 设置 

Get a list of all of the repos added.
helm repo list

Update your repos
helm repo update

 you can list all charts by doing:
helm search repo
helm search repo -r ".*"

Or, you can do a case insensitive match on any part of chart name using the following:
`helm search repo [your_search_string]`

Search for 'nginx' in all of the repos that you have
helm search repo nginx


Finally you can use grep to filter out in a given repo
helm search repo bitnami | grep nginx

Lists all versions of all charts
helm search repo -l 

 Lists all versions of all chart names that contain search string
helm search repo -l [your_search_string]

# 2 使用 chart 部署一个应用


1 查找 chart
```
# 在Artifact Hub https://artifacthub.io/ 或自己的hub实例中搜索chart
helm search hub xxx

# 在本地 helm 客户端中的仓库中搜索chart
helm search repo xxx


helm search repo mysql
```


2 查看chart的信息 

```

# 检查chart(目录、文件或URL)并显示所有的内容（values.yaml, Chart.yaml, README）
helm show all [CHART]

# 检查chart(目录、文件或URL)并显示Chart.yaml文件的内容
helm show chart [CHART]

# 检查chart(目录、文件或URL)并显示values.yaml文件的内容
helm show values [CHART]

# 检查chart(目录、文件或URL)并显示README文件内容
helm show readme [CHART] [flags]


helm show chart stable/mysql
```

3 安装 chart ，形成 release

```
helm install [NAME] [CHART]
helm install db1 stable/mysql
```

注意：
● 每执行一次 install 命令，就会形成一个 release 。
● mysql 是有状态的应用，如果要想执行成功，必须设置持久化存储，并设置默认的动态供应。




4 查看 charts installed by helm ， 也就是查看 已经安装的release 列表
```
helm list

#show all the charts installed by helm on a K8s?
helm list --all-namespaces

状态可能是  unknown, deployed, uninstalled, superseded, failed, uninstalling, pending-install, pending-upgrade 或 pending-rollback 。
```

![](image/Pasted%20image%2020240613173355.png)

```
NAME                            NAMESPACE               REVISION        UPDATED                                   STATUS          CHART                           APP VERSION
csi-driver-smb                  kube-system             2               2024-07-25 13:31:41.479439887 +0200 CEST  deployed        csi-driver-smb-v1.14.0          v1.14.0    
haproxy-ingress-controller      ingress-controller      2               2024-07-25 13:31:39.740794487 +0200 CEST  deployed        haproxy-ingress-0.14.5          v0.14.5    
loki                            loki                    2               2024-07-25 13:31:55.279288948 +0200 CEST  deployed        loki-6.5.2                      3.0.0      
prometheus                      prometheus              2               2024-07-25 13:33:33.672774513 +0200 CEST  deployed        kube-prometheus-stack-58.5.3    v0.73.2    
promtail                        promtail                2               2024-07-25 13:31:50.847799126 +0200 CEST  deployed        promtail-6.15.5                 2.9.3      
[root@e20-d3042b-a01 ~]# helm list --all-namespaces
NAME                            NAMESPACE               REVISION        UPDATED                                         STATUS          CHART                          APP VERSION
csi-driver-smb                  kube-system             2               2024-07-25 13:31:41.479439887 +0200 CEST        deployed        csi-driver-smb-v1.14.0         v1.14.0    
haproxy-ingress-controller      ingress-controller      2               2024-07-25 13:31:39.740794487 +0200 CEST        deployed        haproxy-ingress-0.14.5         v0.14.5    
loki                            loki                    2               2024-07-25 13:31:55.279288948 +0200 CEST        deployed        loki-6.5.2                     3.0.0      
prometheus                      prometheus              2               2024-07-25 13:33:33.672774513 +0200 CEST        deployed        kube-prometheus-stack-58.5.3   v0.73.2    
promtail                        promtail                2               2024-07-25 13:31:50.847799126 +0200 CEST        deployed        promtail-6.15.5                2.9.3 
```



5 查看 release  状态

helm status RELEASE_NAME

状态包括：
● 最后部署时间
● 发布版本所在的k8s命名空间
● 发布状态(可以是： unknown, deployed, uninstalled, superseded, failed, uninstalling, pending-install, pending-upgrade 或 pending-rollback)
● 发布版本修订
● 发布版本描述(可以是完成信息或错误信息，需要用--show-desc启用)
● 列举版本包含的资源，按类型排序
● 最后一次测试套件运行的详细信息（如果使用）
● chart提供的额外的注释

![](image/Pasted%20image%2020240613173439.png)



# 3 安装前自定义chart配置选项


## 3.1 概述

- 自定义选项是因为并不是所有的 chart 都能按照默认配置运行成功，可能会需要一些环境依赖，例如 PV 。
- 所以我们需要自定义 chart 配置选项，安装过程中有两种方法可以传递配置数据：
    - ①`--values`（或`-f`）：指定带有覆盖的 YAML 文件。这里可以多次指定，最右边的文件优先。
    - ②`--set`：在命令行上指定替代。如果两种都用，那么`--set`的优先级高。


## 3.2 --values 的使用

1 先将修改的变量写到一个文件中。
helm show values stable/mysql > config.yaml


2 修改文件

vi config.yaml

```
-- 修改部分
persistence:
  enabled: true
  accessMode: ReadWriteOnce
  size: 8Gi

mysqlUser: "k8s"
mysqlPassword: "123456"
mysqlDatabase: "k8s"
```

3 使用 --values 来替换默认的配置

helm install db stable/mysql -f config.yaml

可以根据提示，去 Pod 中查看。


## 3.3 --set 的使用

helm install db --set persistence.storageClass="nfs-client" stable/mysql


## 3.4 其他技巧

1 
其实，也可以将 chart 包下载下来，查看并修改：

```
# --untar 表示下载并解压
helm pull stable/mysql --untar
```

2 
 
● helm install 命令可以从多个来源安装： 
  ○ ① chart 存储库。
  ○ ② 本地 chart 压缩包。
  ○ ③ chart 目录。
  ○ ④ 完整的 URL。

 示例：从 chart 存储库安装 chart 
helm install db stable/mysql

示例：从本地 chart 压缩包安装 chart
helm pull stable/mysql
helm install db mysql-1.6.9.tgz


示例：从 chart 目录安装 chart
helm pull stable/mysql --untar

示例：从远程 URL 安装 chart
helm install db http://mirror.azure.cn/kubernetes/charts/mysql-1.6.9.tgz



# 4 构建一个 chart 


## 4.1 Chart 的文件结构


- chart 是一个组织在文件目录中的集合。目录名称就是 chart 名称（没有版本信息），因此描述 wordpress 的 chart 可以存储在  `wordpress/` 目录中。

```
wordpress/
├── charts # 包含chart依赖的其他chart
├── Chart.yaml # 用于描述这个 Chart 的基本信息，包括名字、描述信息以及版本等。
├── templates # 模板目录， 当和 values 结合时，可生成有效的Kubernetes manifest文件
│   ├── deployment.yaml
│   ├── _helpers.tpl # 放置可以通过 chart 复用的模板辅助对象
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt # 用于介绍 Chart 帮助信息， helm install 部署后展示给用户。例如：如何使用这个 Chart、列出缺省的设置等。
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml # chart 默认的配置值
```


## 4.2 其他 

1 创建自定义 chart
```
helm create chart的名称
注意：这个命令就如同前端的 vue 、react 生成一个脚手架项目一样，后续我们自定义 chart 就是在这个基础上进行修改。

helm create nginx
```


2 安装自定义 chart

```
helm install [NAME] [CHART]
helm install nginxdemo nginx
```


3 对自定义 chart 进行打包

```
helm package [CHART_PATH]
之所以会对自定义的 chart 进行打包，主要的目的是：方便传输（节省带宽）等。

helm package nginx
```


4 查看实际模板被渲染后的文件
- Kubernetes 目前是不支持在 yaml 中定义变量的，这也是 Helm 出现的意义，Helm 可以有效的管理变量，并使用模板渲染技术（目前，各种语言都有提供，如：Java 的 JSP 本质上就是一种模板渲染技术）将变量渲染到文件中。
- 有的时候，我们想知道模板（template 目录中的 `*.yaml` 文件）渲染后的效果是什么？那么，就需要使用下面的命令：

```
helm get manifest RELEASE_NAME
注意：
● 看到 RELEASE_NAME ，就应该知道该命令需要先执行 helm install 命令。
● 该命令其实是将 value.yaml 中的变量的值填充到 template 目录中的 *.yaml 文件对应的变量中。

helm get manifest nginx-demo
```


5 升级
有的时候，我们修改了 chart 的配置，那么就需要对 chart 进行升级了，可以使用 helm upgrade 命令：
```
helm upgrade --set xxx=xxx [RELEASE] [CHART]
helm upgrade -f values.yaml [RELEASE] [CHART]
注意：这边的 xxx=xxx ，就是就是 chart 目录中的 values.yaml 的值。


helm upgrade --set image.tag=1.17 nginx-demo nginx
helm upgrade -f values.yaml nginx-demo nginx
```


6 查看升级的版本

```
helm history RELEASE_NAME
helm history nginx-demo
```


7 回滚
```
helm rollback <RELEASE> [REVISION]
helm rollback nginx-demo 1
```


8  卸载

```
helm uninstall RELEASE_NAME
helm uninstall nginx-demo
```

# 5 创建一个 Chart模板的流程 

1 
创建一个 Chart 模板
helm create mychart


2 

- 默认 mychart/templates/ 目录下，会存在一些文件：
    - `NOTES.txt`: chart 的 帮助文本。这会在用户执行 `helm install` 时展示给他们。
    - `deployment.yaml`: 创建Kubernetes [工作负载](https://kubernetes.io/docs/user-guide/deployments/)的基本清单。
    - `service.yaml`: 为你的工作负载创建一个 [service终端](https://kubernetes.io/docs/user-guide/services/)基本清单。
    - `_helpers.tpl`: 放置可以通过 char t复用的模板辅助对象。




3 现在我们将 templates 目录下的文件全部删除
rm -rf mychart/templates/*
在实际的生产级别的 chart ，基础版本的 chart 信息还是很有用的，此处只是防止干扰而全部删除。



4 在 template 目录下创建一个 configmap.yaml 文件，用于创建 ConfigMap 对象。

vi mychart/templates/configmap.yaml

```
kind: ConfigMap
apiVersion: v1
metadata:
  name: mychart-configmap
  namespace: default
data:
  myvalue: "Hello World"
```


5 安装 Chart 
helm install demo mychart

这样安装当然没有问题，Helm 读取这个模板的时候会原模原样的传递给 k8s ，毕竟我们之前使用 kubectl apply -f xxx.yaml 也是这么写的。



6 我们可以查看 Helm 实际加载的模板
helm get manifest demo

- 可以注意到，每个文件都是以 `---` 开头（YAML 文件的开头），然后是自定生成的注释行，表示那个模板文件生成了这个 YAML 文档。
- 现在，我们就可以看到 YAML 数据确实是 configmap.yaml 文件中的内容、


7  卸载
helm uninstall demo


8 
Helm 的功能当然不能就这么点，模板文件 configmap.yaml 中的 `metadata.name` 目前是硬编码，不是很好

```
kind: ConfigMap
apiVersion: v1
metadata:
  name: mychart-configmap
  namespace: default
data:
  myvalue: "Hello World"
```


将他修改为 vi mychart/templates/configmap.yaml

```

kind: ConfigMap
apiVersion: v1
metadata:
  name: {{ .Release.Name }}-configmap # {{ .Release.Name }}
  namespace: default
data:
  myvalue: "Hello World"
  
```


注意：由于 DNS 系统的限制，name:字段长度限制为 63 个字符，所以 helm install [NAME] [CHART] 中的 NAME 不要太长。

- `{{ .Release.Name }}-configmap` 中的 `{{ xxx }}` 是模板语法，很像 Vue 中的插值语法。
- 模板命令 `{{ .Release.Name }}` 将发布名称注入了模板，值作为了一个名称空间对象传递给了模板，用点（`.`）分隔每个名称空间的元素。
- `Release`前面的点表示从作用域最顶层的命名空间开始（稍后会谈作用域）。这样`.Release.Name`就可解读为 `通过顶层名称空间开始查找 Release对象，然后在其中找 Name 对象` 。
- `Release`是一个 Helm 的内置对象。稍后会更深入地讨论。但现在足够说明它可以显示从库中赋值的发布名称。


9 
安装 Chart ，查看模板命令的结果
helm install demo1 mychart



10 
查看 Helm 实际加载的模板

helm get manifest demo1

注意：此时 configmap.yaml 文件的 metadata.name 是 demo1-configmap 而不是原来的 mychart-configmap 。



11
k8s 还有 dry-run（干跑），Helm 当然也有这个功能，可能加速模板的构建速度（我们想测试模板渲染的内容但是又不想安装实际应用，只会将模板渲染的内容输出到控制台上）

helm install --dry-run demo3 mychart/

注意：helm 干跑命令仅仅用来查看模板渲染的结果，不能保证 k8s 会正常接收生成的模板。




