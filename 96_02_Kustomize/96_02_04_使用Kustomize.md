

在 kubectl 命令中使用 --kustomize 或 -k 参数来识别被 kustomization.yaml 所管理的资源。 注意 -k 要指向一个 kustomization 目录。例如：

kubectl apply -k <kustomization 目录>/


# 1 执行命令

通过上面的介绍，应该对kustomize有个初步的了解， 最后的工作就是根据不同的环境将资源发布到集群中


## 1.1 `kubectl kustomize`

要查看包含 kustomization 文件的目录中的资源，执行下面的命令
```
kubectl kustomize <kustomization_directory>
```

从print 的结果中可以预览 合成出来的yamlfile 的样子, ==但是不会真的生成 那个文件 ==

例子
执行 执行 `kubectl kustomize ./`depolyment.yaml 
就可查看base+overlay 合并后 出来的最终的 depolyment.yaml 的效果 
这些字段都被设置到 Deployment 资源上：




## 1.2 `kustomize build`

```
# 直接使用kustomize命令
使用命令 kustomize build /tmp/hello/base即可生成定制化后的资源清单：
kustomize build ~/ldap/overlays/staging | kubectl apply -f -

可以使用`kustomiz build . | kubectl apply -f -`
```


另外一个情况下, `bases`已经在kustomize v2.1.0+ 版本中给去掉了,高版本的会直接将`bases`转换成`resources`，这个可以从渲染后的kustomization.yaml中看出





## 1.3 `kubectl apply -k `
 在 kubernetes 1.14 版本， kustomize 已经集成到 kubectl 命令中，成为了其一个子命令，
 
kubectl apply -k ~/ldap/overlays/1box


```


#要应用这些资源，使用 --kustomize 或 -k 参数来执行 kubectl apply：
kubectl apply -k <kustomization_directory>

#可使用 kubectl 来进行部署
kubectl apply -k ~/ldap/overlays/staging
```



```yaml
  if $action == 'apply' {
    exec { "kubectl kustomize apply ${kustomize_dir}":
      path    => '/usr/bin:/usr/local/bin',
      command => "k0s kubectl kustomize ${kustomize_dir} --enable-helm | k0s kubectl apply -f -",
      # run this exec only if it hasn't been applied before or if the hash value of kustomize files has changed
      onlyif  => $compare_cmd,
    }
    ~> exec { "save kustomize applied checksum ${name}":
      path    => '/usr/bin:/usr/local/bin',
      command => "sha512sum ${kustomize_dir}/*.yaml | sha512sum | cut -d ' ' -f 1 > ${apply_file}",
      # run this exec only if it hasn't been applied before or if the hash value of kustomize files has changed
      onlyif  => $compare_cmd,
    }
  } elsif $action == 'delete' {
    exec { "kubectl kustomize delete ${kustomize_dir}":
      path    => '/usr/bin:/usr/local/bin',
      command => "k0s kubectl kustomize ${kustomize_dir} --enable-helm | k0s kubectl delete -f -",
      onlyif  => "[ -f ${apply_file} ]",
    }
    ~> exec { "remove kustomize applied checksum ${name}":
      path    => '/usr/bin:/usr/local/bin',
      command => "rm ${apply_file}",
      onlyif  => "[ -f ${apply_file} ]",
    }
  }
```


### 1.3.1 例子

假定使用下面的 `kustomization.yaml`：

```shell
# 创建 deployment.yaml 文件
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

# 创建 kustomization.yaml
cat <<EOF >./kustomization.yaml
namePrefix: dev-
commonLabels:
  app: my-nginx
resources:
- deployment.yaml
EOF
```

执行下面的命令来应用 Deployment 对象 `dev-my-nginx`：

```shell
> kubectl apply -k ./
deployment.apps/dev-my-nginx created
```


### 1.3.2 可能出现的error 

如果 `kubectl apply -k 出现error: rawResources failed to read Resources`的错误
```
error: rawResources failed to read Resources: Load from path ../../base/ failed: '../../base/' must be a file (got d='/Users/zhoushuke/git_uni/gitlab.bj.sensetime.com/gitlabci-golang-demo/deploy/base')
```

出现这种情况是因为`kustomize`与`kubectl apply -k` 版本不匹配(kubectl的版本其实是跟着集群的版本走的, 而kustomize二进制可能不一定会对着kubectl的版本安装，有可能就直接安装了最新版本)

# 2 查看 Deployment 对象 
运行下面的命令之一来查看 Deployment 对象 dev-my-nginx：

kubectl get -k ./

kubectl describe -k ./


# 3 `kubectl diff -k ` 比较 Deployment 对象 dev-my-nginx 与清单被应用之后集群
执行下面的命令来比较 Deployment 对象 dev-my-nginx 与清单被应用之后集群将处于的状态：

```shell
kubectl diff -k ./
```

执行下面的命令删除 Deployment 对象 `dev-my-nginx`：

```shell
> kubectl delete -k ./
deployment.apps "dev-my-nginx" deleted
```

# 4 不同场景应用不同variant

Kustomize 的目标是一样的，但不使用模板。相反，它在一个目录中保留**完整**版本的 YAML 文件。按照惯例，这个文件被称为 `base`，但也可以根据自己的喜好给它命名。然后可以为每个环境/场景/用例创建一个目录(或目录树)，每个目录都需要一个名为 `kustomization.yaml` 的 YAML 文件，该文件的目的是告知 Kustomize 应该考虑哪些 manifest 文件，以及需要对这些文件进行哪些修改。下面通过例子来说明这，看看如何使用 Kustomize 得出与 Helm 相同的结果。



首先创建一个目录结构：
```
myapp/
├── kustomization.yaml
├── base
│   └── deployment.yaml
└── overlay
    └── deployment.yaml
```


myapp/kustomization.yaml 的内容如下：
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - base/deployment.yaml

patchesStrategicMerge:
  - overlay/deployment.yaml
```


`base/deployment.yaml` 看起来像这样：
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: myapp
          image: myapp/image:v1.0.0
          ports:
            - containerPort: 8080
```

请注意，这是一个完全有效的 YAML，如果需要，也可以按原样应用。

  

要更改该部署以适应环境需求，可以使用 `overlay/deployment.yaml` 文件：
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  template:
    spec:
      containers:
        - name: myapp
          args:
            - arg1
            - arg2
            - arg3
```

这样，发送到 Kubernetes API 服务器的文件就变成了
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: myapp
          image: myapp/image:v1.0.0
          ports:
            - containerPort: 8080
          args:
            - arg1
            - arg2
            - arg3
```



---

把同样的机制应用到三个环境中，目录结构可以是这样的：
```
myapp/
├── kustomization.yaml
├── base
│   └── deployment.yaml
├── overlays
│   ├── dev
│   │   └── kustomization.yaml
│   ├── qa
│   │   └── kustomization.yaml
│   └── prod
│       └── kustomization.yaml
└── patches
    └── deployment-patch.yaml
```

如果需要对整个环境进行更改，只需在 `base/deployment` 文件中进行一次更改，就会传播到所有地方。针对特定环境的更改在相应环境的自定义文件中完成。

![](https://static001.geekbang.org/infoq/27/27376c746a9d5c9c51ecae0265d3886e.png)

Kustomize 使用补丁和覆盖层，同时保持源 YAML 不变




# 5 workflows 工作流


kustomize 将对 Kubernetes 应用的管理转换成对 Kubernetes manifests YAML 文件的管理，而对应用的修改也通过 YAML 文件来修改。这种修改变更操作可以通过 Git 版本控制工具进行管理维护, 因此用户可以使用 Git 风格的流程来管理应用。 workflows 是使用并配置应用所使用的一系列 Git 风格流程步骤。官网提供了两种方式，一种是定制配置，另一种是现成配置


## 5.1 定制场景
[workflow](https://kubernetes-sigs.github.io/kustomize/guides/bespoke/)如下:

[![](https://raw.githubusercontent.com/zhoushuke/BlogPhoto/master/githuboss/20200701133434.png)](https://raw.githubusercontent.com/zhoushuke/BlogPhoto/master/githuboss/20200701133434.png)


![](image/20210430172702.jpg)


如上图所示，有一个 `ldap` 的应用，`/base`目录保存的是基本的配置，`/overlays`里放置的不同环境的配置，例如 `/dev`、`/staging`，`/prod`这些就是不同环境的配置，`/base`等文件夹下都有一个 `kustomization .yml` 文件，用于配置。

执行 `kustomize build dir`的方式就可以生成我们最后用于部署的 yaml 文件，也就是进行到了我们上图的第四步，然后通过 `kubectl apply -f`命令进行部署




## 5.2 现成配置
在这个工作流方式中，可从别人的 repo 中 fork kustomize 配置，并根据自己的需求来配置

[workflow](https://kubernetes-sigs.github.io/kustomize/guides/offtheshelf/)如下:

[![](https://raw.githubusercontent.com/zhoushuke/BlogPhoto/master/githuboss/20200701133953.png)](https://raw.githubusercontent.com/zhoushuke/BlogPhoto/master/githuboss/20200701133953.png)

流程非常地简洁，相信不需要过多解释， 个人觉得这两种方式除了源头上有区别外，在使用上没什么差别.









