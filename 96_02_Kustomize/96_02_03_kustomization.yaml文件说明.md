
https://izsk.me/2020/07/01/Kubernetes-kustomize/

# 1 kustomization.yaml 

可以发现在base目录或者是overlay目录下都有一个kustomization.yaml文件，该文件是kustomize的核心， 包含了需要部署的资源.

当然它本身是有一定语法的, 详细的使用可参考[这里](https://kubernetes-sigs.github.io/kustomize/api-reference/glossary/#kustomization)，下面会挑选一些常用的配置来说明如何使用

`cat kustomization.yaml`

```
apiVersion: kustomize.config.k8s.io/v1beta1  
kind: Kustomization  
  
commonLabels:  
  k8s-app: p-expoter # 会在所有的资源对象上都会加上该label  
  namespace: stage # 会在所有的资源对象上都指定该ns  
  
# configMapGenerator:   # 也可以写为 secretGenerator  
# - name: special-config  
#   files:  
#     - configmap.yaml  

resources:  
  - config.yaml  
  - service.yaml  
  - deployment.yaml
```


# 2 kustomize功能特性表_字段总览

apiVsersion 和 kind 为两个固定字段

kustomize 提供了比较丰富的字段选择，除此之外还可以自定义插件，下面会大概列举一下每个字段的含义，当我们需要用到的时候知道有这么个能力，然后再去 [Kustomize 官方文档](https://kubectl.docs.kubernetes.io/zh/guides/) 查找对应的 API 文档就行了

https://kubernetes.io/zh-cn/docs/tasks/manage-kubernetes-objects/kustomization/


| 字段                    | 类型                                                                                                            | 解释                                                                                                                            |
| --------------------- | ------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| namespace             | string                                                                                                        | 为所有资源添加名字空间                                                                                                                   |
| namePrefix            | string                                                                                                        | 此字段的值将被添加到所有资源名称前面                                                                                                            |
| nameSuffix            | string                                                                                                        | 此字段的值将被添加到所有资源名称后面                                                                                                            |
| commonLabels          | map[string]string                                                                                             | 要添加到所有资源和选择算符的标签                                                                                                              |
| commonAnnotations     | map[string]string                                                                                             | 要添加到所有资源的注解                                                                                                                   |
| resources             | []string                                                                                                      | 列表中的每个条目都必须能够解析为现有的资源配置文件                                                                                                     |
| configMapGenerator    | [][ConfigMapArgs](https://github.com/kubernetes-sigs/kustomize/blob/master/api/types/configmapargs.go#L7)     | 列表中的每个条目都会生成一个 ConfigMap                                                                                                      |
| secretGenerator       | [][SecretArgs](https://github.com/kubernetes-sigs/kustomize/blob/master/api/types/secretargs.go#L7)           | 列表中的每个条目都会生成一个 Secret                                                                                                         |
| generatorOptions      | [GeneratorOptions](https://github.com/kubernetes-sigs/kustomize/blob/master/api/types/generatoroptions.go#L7) | 更改所有 ConfigMap 和 Secret 生成器的行为                                                                                                |
| bases                 | []string                                                                                                      | 列表中每个条目都应能解析为一个包含 kustomization.yaml 文件的目录                                                                                    |
| patchesStrategicMerge | []string                                                                                                      | 列表中每个条目都能解析为某 Kubernetes 对象的策略性合并补丁                                                                                           |
| patchesJson6902       | [][Patch](https://github.com/kubernetes-sigs/kustomize/blob/master/api/types/patch.go#L10)                    | 列表中每个条目都能解析为一个 Kubernetes 对象和一个 JSON 补丁                                                                                       |
| vars                  | [][Var](https://github.com/kubernetes-sigs/kustomize/blob/master/api/types/var.go#L19)                        | 每个条目用来从某资源的字段来析取文字                                                                                                            |
| images                | [][Image](https://github.com/kubernetes-sigs/kustomize/blob/master/api/types/image.go#L8)                     | 每个条目都用来更改镜像的名称、标记与/或摘要，不必生成补丁                                                                                                 |
| configurations        | []string                                                                                                      | 列表中每个条目都应能解析为一个包含 [Kustomize 转换器配置](https://github.com/kubernetes-sigs/kustomize/tree/master/examples/transformerconfigs) 的文件 |
| crds                  | []string                                                                                                      | 列表中每个条目都应能够解析为 Kubernetes 类别的 OpenAPI 定义文件                                                                                    |




- `resources` 表示 k8s 资源的位置，这个可以是一个文件，也可以指向一个文件夹，读取的时候会按照顺序读取，路径可以是相对路径也可以是绝对路径，如果是相对路径那么就是相对于 `kustomization.yml`的路径
- `crds` 和 `resources` 类似，只是 `crds` 是我们自定义的资源
- `namespace` 为所有资源添加 namespace
- `images` 修改镜像的名称、tag 或 image digest ，而无需使用 patches
- `replicas` 修改资源副本数
- `namePrefix` 为所有资源和引用的名称添加前缀
- `nameSuffix` 为所有资源和引用的名称添加后缀
- `patches` 在资源上添加或覆盖字段，Kustomization 使用 `patches` 字段来提供该功能。
- `patchesJson6902` 列表中的每个条目都应可以解析为 kubernetes 对象和将应用于该对象的 [JSON patch](https://tools.ietf.org/html/rfc6902)。
    
- `patchesStrategicMerge` 使用 strategic merge patch 标准 Patch resources.
- `vars` 类似指定变量
- `commonAnnotations` 为所有资源加上 `annotations` 如果对应的 key 已经存在值，这个值将会被覆盖
```
commonAnnotations:
  app.lailin.xyz/inject: agent

resources:
- deploy.yaml
```

- `commonLabels` 为所有资源的加上 label 和 label selector **注意：这个操作会比较危险**
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

commonLabels:
  app: bingo
```

- `configMapGenerator` 可以生成 config map，列表中的每一条都会生成一个 configmap
- `secretGenerator` 用于生成 secret 资源
- `generatorOptions` 用于控制 `configMapGenerator` 和 `secretGenerator` 的行为



The following configuration options are available for Kustomize:
- `namePrefix` is a prefix appended to resources for Kustomize apps
- `nameSuffix` is a suffix appended to resources for Kustomize apps
- `images` is a list of Kustomize image overrides
- `replicas` is a list of Kustomize replica overrides
- `commonLabels` is a string map of additional labels
- `labelWithoutSelector` is a boolean value which defines if the common label(s) should be applied to resource selectors and templates.
- `forceCommonLabels` is a boolean value which defines if it's allowed to override existing labels
- `commonAnnotations` is a string map of additional annotations
- `namespace` is a Kubernetes resources namespace
- `forceCommonAnnotations` is a boolean value which defines if it's allowed to override existing annotations
- `commonAnnotationsEnvsubst` is a boolean value which enables env variables substition in annotation values
- `patches` is a list of Kustomize patches that supports inline updates
- `components` is a list of Kustomize components


# 3 kustomization.yaml中可以生成资源的字段


## 3.1 configMapGenerator
https://kubernetes.io/zh-cn/docs/tasks/manage-kubernetes-objects/kustomization/

这两者具有相同的功能, 可以从文件中直接生成configmap或者是secret
格式如上代码所示，这部分也会渲染到最终的yaml文件中， 因此，**除了resources指定的资源外, 还能通过这种方式生成部署对象**


```
# configMapGenerator:   # secretGenerator  
# - name: special-config  
#   files:  
#     - configmap.yaml 
```
`configMapGenerator` 可以生成 config map，为 files 中 每一个文件 中的每一条都会生成一个 configmap 类型的资源 
configmap.yaml  中的内容 就作为  configmap 类型的资源  的manifest 中的  data 这个 entry 的具体内容 

1 要基于File来生成 ConfigMap
`cat kustomization.yaml`

```
apiVersion: kustomize.config.k8s.io/v1beta1  
kind: Kustomization  
  
commonLabels:  
  k8s-app: p-expoter # 会在所有的资源对象上都会加上该label  
namespace: stage # 会在所有的资源对象上都指定该ns  
  
# configMapGenerator:   # secretGenerator  
# - name: special-config  
#   files:  
#     - configmap.yaml  

resources:  
  - config.yaml  
  - service.yaml  
  - deployment.yaml
```


要基于文件来生成 ConfigMap，可以在 `configMapGenerator` 的 `files` 列表中添加表项。 下面是一个根据 `.properties` 文件中的数据条目来生成 ConfigMap 的示例：

```shell
# 生成一个  application.properties 文件
cat <<EOF >application.properties
FOO=Bar
EOF

cat <<EOF >./kustomization.yaml
configMapGenerator:
- name: example-configmap-1
  files:
  - application.properties
EOF
```

所生成的 ConfigMap 可以使用下面的命令来检查：

```shell
kubectl kustomize ./
```

所生成的  类型为  ConfigMap 的文件中的内特容为 ：

```yaml
apiVersion: v1
data:
  application.properties: |
    FOO=Bar    
kind: ConfigMap
metadata:
  name: example-configmap-1-8mbdf7882g
```


----

2 要从 env 文件生成 ConfigMap

要从 env 文件生成 ConfigMap，请在 `configMapGenerator` 中的 `envs` 列表中添加一个条目。 下面是一个用来自 `.env` 文件的数据生成 ConfigMap 的例子：

```shell
# 创建一个 .env 文件
cat <<EOF >.env
FOO=Bar
EOF

cat <<EOF >./kustomization.yaml
configMapGenerator:
- name: example-configmap-1
  envs:
  - .env
EOF
```

可以使用以下命令检查生成的 ConfigMap：

```shell
kubectl kustomize ./
```

生成的 ConfigMap 为：

```yaml
apiVersion: v1
data:
  FOO: Bar
kind: ConfigMap
metadata:
  name: example-configmap-1-42cfbf598f
```

---

3 ConfigMap 也可基于字面的键值偶对来生成

ConfigMap 也可基于字面的键值偶对来生成。要基于键值偶对来生成 ConfigMap， 在 `configMapGenerator` 的 `literals` 列表中添加表项。下面是一个例子， 展示如何使用键值偶对中的数据条目来生成 ConfigMap 对象：

```
cat <<EOF >./kustomization.yaml
configMapGenerator:
- name: example-configmap-2
  literals:
  - FOO=Bar
EOF
```

可以用下面的命令检查所生成的 ConfigMap：

```shell
kubectl kustomize ./
```

所生成的 ConfigMap 为：

```yaml
apiVersion: v1
data:
  FOO: Bar
kind: ConfigMap
metadata:
  name: example-configmap-2-g2hdhfc6tk
```

----
4 要在 Deployment 中使用生成的 ConfigMap，

要在 Deployment 中使用生成的 ConfigMap，使用 configMapGenerator 的名称对其进行引用。 Kustomize 将自动使用生成的名称替换该名称。

这是使用生成的 ConfigMap 的 deployment 示例：

```yaml
# 创建一个 application.properties 文件
cat <<EOF >application.properties
FOO=Bar
EOF

cat <<EOF >deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: my-app
        volumeMounts:
        - name: config
          mountPath: /config
      volumes:
      - name: config
        configMap:
          name: example-configmap-1
EOF

cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
configMapGenerator:
- name: example-configmap-1
  files:
  - application.properties
EOF
```

生成 ConfigMap 和 Deployment：

```shell
kubectl kustomize ./
```

生成的 Deployment 将通过名称引用生成的 ConfigMap：

```yaml
apiVersion: v1
data:
  application.properties: |
    FOO=Bar    
kind: ConfigMap
metadata:
  name: example-configmap-1-g4hk9g2ff8
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: my-app
  name: my-app
spec:
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - image: my-app
        name: app
        volumeMounts:
        - mountPath: /config
          name: config
      volumes:
      - configMap:
          name: example-configmap-1-g4hk9g2ff8
        name: config
```


## 3.2 secretGenerator


你可以基于文件或者键值偶对来生成 Secret。要使用文件内容来生成 Secret， 在 `secretGenerator` 下面的 `files` 列表中添加表项。 


1 下面是一个根据文件中数据来生成 Secret 对象的示例：

```shell
# 创建一个 password.txt 文件
cat <<EOF >./password.txt
username=admin
password=secret
EOF

cat <<EOF >./kustomization.yaml
secretGenerator:
- name: example-secret-1
  files:
  - password.txt
EOF
```

所生成的 Secret 如下：

```yaml
apiVersion: v1
data:
  password.txt: dXNlcm5hbWU9YWRtaW4KcGFzc3dvcmQ9c2VjcmV0Cg==
kind: Secret
metadata:
  name: example-secret-1-t2kt65hgtb
type: Opaque
```


---

2 要基于键值偶对字面值生成 Secret，
先要在 `secretGenerator` 的 `literals` 列表中添加表项。下面是基于键值偶对中数据条目来生成 Secret 的示例：

```shell
cat <<EOF >./kustomization.yaml
secretGenerator:
- name: example-secret-2
  literals:
  - username=admin
  - password=secret
EOF
```

所生成的 Secret 如下：

```yaml
apiVersion: v1
data:
  password: c2VjcmV0
  username: YWRtaW4=
kind: Secret
metadata:
  name: example-secret-2-t52t6g96d8
type: Opaque
```


---


3  通过引用 secretGenerator 的名称在 Deployment 中使用
与 ConfigMap 一样，生成的 Secret 可以通过引用 secretGenerator 的名称在 Deployment 中使用：

```shell
# 创建一个 password.txt 文件
cat <<EOF >./password.txt
username=admin
password=secret
EOF

cat <<EOF >deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: my-app
        volumeMounts:
        - name: password
          mountPath: /secrets
      volumes:
      - name: password
        secret:
          secretName: example-secret-1  # 注意这里
EOF

cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml

secretGenerator:
- name: example-secret-1
  files:
  - password.txt
EOF
```


## 3.3 generatorOptions

所生成的 ConfigMap 和 Secret 都会包含内容哈希值后缀。 这是为了确保内容发生变化时，所生成的是新的 ConfigMap 或 Secret。 要禁止自动添加后缀的行为，用户可以使用 generatorOptions。 除此以外，为生成的 ConfigMap 和 Secret 指定贯穿性选项也是可以的。

```shell
cat <<EOF >./kustomization.yaml
configMapGenerator:
- name: example-configmap-3
  literals:
  - FOO=Bar
generatorOptions:
  disableNameSuffixHash: true
  labels:   # 加入这个鞋 label 到 生成额 configmap 的manifest 的 metadata 下面去 为一行行的文本 
    type: generated
  annotations:
    note: generated
EOF
```



运行 `kubectl kustomize ./` 来查看所生成的 ConfigMap：

```yaml
apiVersion: v1
data:
  FOO: Bar
kind: ConfigMap
metadata:
  annotations:
    note: generated
  labels:
    type: generated
  name: example-configmap-3
```




# 4 kustomization.yaml中可以组织和定制资源的字段

## 4.1 resources组织
这个字段包含需要部署的资源文件， 这个应该很容易理解

一种常见的做法是在项目中构造资源集合并将其放到同一个文件或目录中管理。 Kustomize 提供基于不同文件来组织资源并向其应用补丁或者其他定制的能力。


Kustomize 支持组合不同的资源。`kustomization.yaml` 文件的 `resources` 字段定义配置中要包含的资源列表。 你可以将 `resources` 列表中的路径设置为资源配置文件的路径。 下面是由 Deployment 和 Service 构成的 NGINX 应用的示例：


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

# 创建 service.yaml 文件
cat <<EOF > service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
EOF

# 创建 kustomization.yaml 来组织以上两个资源
cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
- service.yaml
EOF
```

`kubectl kustomize ./` 所得到的资源中既包含 Deployment 也包含 Service 对象。




# 5 Demo: Inline Patch 很好的例子 
https://github.com/kubernetes-sigs/kustomize/blob/master/examples/inlinePatch.md

A kustomization file supports patching in three ways:
- patchesStrategicMerge: A list of patch files where each file is parsed as a [Strategic Merge Patch].
- patchesJSON6902: A list of patches and associated targets, where each file is parsed as a [JSON Patch] and can only be applied to one target resource.
- patches: A list of patches and their associated targets. The patch can be applied to multiple objects. It auto detects whether the patch is a [Strategic Merge Patch] or [JSON Patch].

Since 3.2.0, all three support inline patch, where the patch content is put inside the kustomization file as a single string. With this feature, no separate patch files need to be created.

Make a base kustomization containing a Deployment resource.
<!-- @createKustomization @test -->
```
DEMO_HOME=$(mktemp -d)

BASE=$DEMO_HOME/base
mkdir $BASE

cat <<EOF >$BASE/kustomization.yaml
resources:
- deployments.yaml
EOF

cat <<EOF >$BASE/deployments.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy
spec:
  template:
    metadata:
      labels:
        foo: bar
    spec:
      containers:
        - name: nginx
          image: nginx
          args:
          - one
          - two
EOF
```


## 5.1 Inline Patch

Create an overlay and add an inline patch in the `patches` field to the kustomization file
to change the image from `nginx` to `nginx:latest`.

<!-- @addSMPatch @test -->
```
SMP_OVERLAY=$DEMO_HOME/smp
mkdir $SMP_OVERLAY
cat <<EOF >$SMP_OVERLAY/kustomization.yaml
resources:
- ../base

patches:
- patch: |-
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: deploy
    spec:
      template:
        spec:
          containers:
          - name: nginx
            image: nginx:latest
          
EOF
```

Running `kustomize build $SMP_OVERLAY`, in the output confirm that image is updated successfully.

<!-- @confirmSMPatch @test -->
```
test 1 == \
  $(kustomize build $SMP_OVERLAY | grep "image: nginx:latest" | wc -l); \
  echo $?
```

The output is
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy
spec:
  template:
    metadata:
      labels:
        foo: bar
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          args:
          - one
          - two
```

`$patch: delete` and `$patch: replace` also work in the inline patch. Change the inline patch to delete the container `nginx`.

<!-- @addDeleteSMPatch @test -->
```
cat <<EOF >$SMP_OVERLAY/kustomization.yaml
resources:
- ../base

patches:
- patch: |-
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: deploy
    spec:
      template:
        spec:
          containers:
          - name: nginx
            $patch: delete
          
EOF
```
Running `kustomize build $SMP_OVERLAY`, in the output confirm that the `nginx` container has been deleted.

<!-- @confirmDeleteSMPatch @test -->
```
test 0 == \
  $(kustomize build $SMP_OVERLAY | grep "image: nginx" | wc -l); \
  echo $?
```

The output is
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy
spec:
  template:
    metadata:
      labels:
        foo: bar
    spec:
      containers: []
```

## 5.2 Inline Patch for PatchesJson6902

Create an overlay and add an inline patch in `patchesJSON6902` field to the kustomization file
to change the image from `nginx` to `nginx:latest`.

<!-- @addJSONPatch @test -->
```
JSON_OVERLAY=$DEMO_HOME/json
mkdir $JSON_OVERLAY
cat <<EOF >$JSON_OVERLAY/kustomization.yaml
resources:
- ../base

patchesJSON6902:
- target:
    group: apps
    version: v1
    kind: Deployment
    name: deploy
  patch: |-
    - op: replace
      path: /spec/template/spec/containers/0/image
      value: nginx:latest
EOF
```

Running `kustomize build $JSON_OVERLAY`, in the output confirm that image is updated successfully.

<!-- @confirmJSONPatch @test -->
```
test 1 == \
  $(kustomize build $JSON_OVERLAY | grep "image: nginx:latest" | wc -l); \
  echo $?
```

The output is
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy
spec:
  template:
    metadata:
      labels:
        foo: bar
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          args:
          - one
          - two
```

## 5.3 Inline Patch for Patches

Create an overlay and add an inline patch in `patches` field to the kustomization file
to change the image from `nginx` to `nginx:latest`.

<!-- @addPatch @test -->
```
PATCH_OVERLAY=$DEMO_HOME/patch
mkdir $PATCH_OVERLAY
cat <<EOF > $PATCH_OVERLAY/kustomization.yaml
resources:
- ../base

patches:
- target:
    kind: Deployment
    name: deploy   # 哪个文件会被打补丁 
  patch: |-
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: deploy
    spec:
      template:
        spec:
          containers:
          - name: nginx
            image: nginx:latest
EOF
```

Running `kustomize build $PATCH_OVERLAY`, in the output confirm that image is updated successfully.

<!-- @confirmPatch @test -->
```
test 1 == \
  $(kustomize build $PATCH_OVERLAY | grep "image: nginx:latest" | wc -l); \
  echo $?
```

The output is
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy
spec:
  template:
    metadata:
      labels:
        foo: bar
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          args:
          - one
          - two
```


## 5.4 external patch.yaml in patches 

https://github.com/kubernetes-sigs/kustomize/blob/master/examples/patchMultipleObjects.md

kustomize supports patching via either a strategic merge patch (wherein you partially re-specify the thing you want to modify, with in-place changes) or a JSON patch (wherein you specify specific operation/target/value tuples in a particular syntax).

A kustomize file lets one specify many patches. Each patch must be associated with a target selector:

```yaml
patches:
  - path: <relative path to file containing patch>
    target:
      group: <optional group>
      version: <optional version>
      kind: <optional kind>
      name: <optional name or regex pattern>
      namespace: <optional namespace>
      labelSelector: <optional label selector>
      annotationSelector: <optional annotation selector>
```


E.g. select resources with name matching the regular expression foo.*:

    target:
      name: foo.*

Select all resources of kind Deployment:

    target:
      kind: Deployment

Using multiple fields just makes the target more specific. The following selects only Deployments that also have the label app=hello (full label/annotation selector rules):

    target:
      kind: Deployment
      labelSelector: app=hello


### 5.4.1 Demo

[](https://github.com/kubernetes-sigs/kustomize/blob/master/examples/patchMultipleObjects.md#demo)

The example below shows how to inject a sidecar container for multiple Deployment resources.

Make a place to work:

```
DEMO_HOME=$(mktemp -d)
```

Make a file describing two Deployments:

```
cat <<EOF >$DEMO_HOME/deployments.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    old-label: old-value
  name: deploy1
spec:
  template:
    metadata:
      labels:
        old-label: old-value
    spec:
      containers:
        - name: nginx
          image: nginx
          args:
          - one
          - two
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    key: value
  name: deploy2
spec:
  template:
    metadata:
      labels:
        key: value
    spec:
      containers:
        - name: busybox
          image: busybox
EOF
```

Declare a [strategic merge patch](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/strategic-merge-patch.md) file to inject a sidecar container:

```
cat <<EOF >$DEMO_HOME/patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: not-important
spec:
  template:
    spec:
      containers:
        - name: istio-proxy
          image: docker.io/istio/proxyv2
          args:
          - proxy
          - sidecar
EOF
```

Finally, define a kustomization file that specifies both a `patches` and `resources` entry:

```
cat <<EOF >$DEMO_HOME/kustomization.yaml
resources:
- deployments.yaml

patches:
- path: patch.yaml
  target:
    kind: Deployment
EOF
```

Two deployment will be patched, the expected result is:

```
cat <<EOF >$DEMO_HOME/out_expected.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    old-label: old-value
  name: deploy1
spec:
  template:
    metadata:
      labels:
        old-label: old-value
    spec:
      containers:
      - args:
        - proxy
        - sidecar
        image: docker.io/istio/proxyv2
        name: istio-proxy
      - args:
        - one
        - two
        image: nginx
        name: nginx
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    key: value
  name: deploy2
spec:
  template:
    metadata:
      labels:
        key: value
    spec:
      containers:
      - args:
        - proxy
        - sidecar
        image: docker.io/istio/proxyv2
        name: istio-proxy
      - image: busybox
        name: busybox
EOF
```

Run the build:

```
kustomize build $DEMO_HOME >$DEMO_HOME/out_actual.yaml
```

Confirm expectations:

```
diff $DEMO_HOME/out_actual.yaml $DEMO_HOME/out_expected.yaml
```

Let us do one more try. Redefine a kustomization file. This time only patch one deployment whose label is "key: value".

```
cat <<EOF >$DEMO_HOME/kustomization.yaml
resources:
- deployments.yaml

patches:
- path: patch.yaml
  target:
    kind: Deployment
    labelSelector: key=value
EOF
```

Run the build:

```
kustomize build $DEMO_HOME 
```

Confirm expectations:

```
Only deploy2 is patched since its label matches "labelSelector: key=value". No change for deploy1.
```


### 5.4.2 Demo2 

https://medium.com/@tharukam/generate-kubernetes-manifests-with-helm-charts-using-kustomize-2f82ab5c5f11

Now, let’s dive into the process of using Kustomize to augment your Helm Charts. You can provide a `values.yaml` file to Helm, which contains configuration settings tailored to your environment. This allows you to override default values defined in the Helm Chart templates. First let's see how we can build a kustomization file to generate Kubernetes manifests for our Helm chart

```
# kustomization.yaml  

apiVersion: kustomize.config.k8s.io/v1beta1  
kind: Kustomization  
  
helmCharts:  
- name: mergestat  
  repo: https://helm.mergestat.com/  
  releaseName: mergestat  
  namespace: admin  
  version: 0.1.0  
  valuesFile: values.yaml
```

```

# Example values.yaml file  

worker:  
  replicaCount: 1  
  image:  
    repository: mergestat/worker  
    pullPolicy: Always  
  env:  
  - name: POSTGRES_CONNECTION  
    valueFrom:  
      secretKeyRef:  
        name: mergestat-postgresql  
        key: postgresql-uri  
  - name: CONCURRENCY  
    value: "3"  
  - name: ENCRYPTION_SECRET  
    valueFrom:  
      secretKeyRef:  
        name: mergestat-secrets  
        key: encryption-secret  
  
graphql:  
  replicaCount: 1  
  image:  
    repository: mergestat/graphql  
    pullPolicy: Always  
  env:  
  - name: POSTGRES_CONNECTION  
    valueFrom:  
      secretKeyRef:  
        name: mergestat-postgresql  
        key: postgresql-uri  
  - name: JWT_SECRET  
    valueFrom:  
      secretKeyRef:  
        name: mergestat-secrets  
        key: jwt-secret  
  - name: ENCRYPTION_SECRET  
    valueFrom:  
      secretKeyRef:  
        name: mergestat-secrets  
        key: encryption-secret  
  
ui:  
  replicaCount: 1  
  image:  
    repository: mergestat/console  
    pullPolicy: Always  
  env:  
  - name: POSTGRAPHILE_API  
    value: http://mergestat-graphql:5433/graphql  
  - name: POSTGRES_CONNECTION  
    valueFrom:  
      secretKeyRef:  
        name: mergestat-postgresql  
        key: postgresql-uri  
  - name: JWT_SECRET  
    valueFrom:  
      secretKeyRef:  
        name: mergestat-secrets  
        key: jwt-secret  
  
postgresql:  
# 8 https://github.com/bitnami/charts/tree/master/bitnami/postgresql  
  enabled: false
```

You can use the following link as a reference to see what are the supported fields in the helmCharts field: [Link](https://kubectl.docs.kubernetes.io/references/kustomize/builtins/#_helmchartinflationgenerator_)

Incorporating Kustomize patches further enhances your customization capabilities. You can apply Kustomize patches to modify Helm Chart templates selectively. These patches can target specific resources within the Helm Chart and make adjustments without altering the original Chart files.

Let's add a nodeAffinity patch to the kustomization file as well.



```
#  Patch File for nodeAffinity  

apiVersion: apps/v1  
kind: Deployment  
metadata:  
  name: node_affinity_deployment  
spec:  
  template:  
    spec:  
      affinity:  
        nodeAffinity:  
          requiredDuringSchedulingIgnoredDuringExecution:  
            nodeSelectorTerms:  
            - matchExpressions:  
              - key: <key from node labels>  
                operator: In  
                values:  
                - <value to match>
```

```
# 10 kustomization.yaml  
apiVersion: kustomize.config.k8s.io/v1beta1  
kind: Kustomization  
  
helmCharts:  
- name: mergestat  
  repo: https://helm.mergestat.com/  
  releaseName: mergestat  
  namespace: admin  
  version: 0.1.0  
  valuesFile: values.yaml  
  
patches:  
- path: patches/node_affinity_deployment.yaml  
  target:  
    kind: Deployment
```

After configuring your needs you can use the below commands to inflate your helm charts to Kubernetes manifest files

```
mkdir build  
kustomize build -o build --enable-helm
```

In the above commands, you can see the `--enable-helm` flag which tells kustomize to use the helm client. This command will download the Helm chart to your local machine before generating Kubernetes manifests.

![](https://miro.medium.com/v2/resize:fit:499/1*L2FMlTHhH13_Ul5kvl034Q.png)

# 6 Patch功能 另外的解释


[Strategic Merge Patch]: https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/strategic-merge-patch.md
[JSON Patch]: https://tools.ietf.org/html/rfc6902
## 6.1 Patches

patches` 在资源上添加或覆盖字段，Kustomization 使用 `patches` 字段来提供该功能。

Patches are a way to kustomize resources using inline configurations in Argo CD applications. patches follow the same logic as the corresponding Kustomization. Any patches that target existing Kustomization file will be merged.

This Kustomize example sources manifests from the `/kustomize-guestbook` folder of the `argoproj/argocd-example-apps` repository, and patches the `Deployment` to use port 443 on the container. 


1 Kustomization 的 manifest 
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
metadata:
  name: kustomize-inline-example
namespace: test1
resources:
  - https://github.com/argoproj/argocd-example-apps//kustomize-guestbook/
patches:
  - target:
      kind: Deployment
      name: guestbook-ui
    patch: |-
      - op: replace
        path: /spec/template/spec/containers/0/ports/0/containerPort
        value: 443
```

2 直接写到 argo 的 application 的 spec 
和上面的 用到kusomitzation 的 manifest 起到同等效果 

This Application does the equivalent using the inline kustomize.patches configuration. 
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: kustomize-inline-guestbook
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  destination:
    namespace: test1
    server: https://kubernetes.default.svc
  project: default
  source:
    path: kustomize-guestbook
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: master
    kustomize:
      patches:
        - target:
            kind: Deployment
            name: guestbook-ui
          patch: |-
            - op: replace
              path: /spec/template/spec/containers/0/ports/0/containerPort
              value: 443
```



---

The inline kustomize patches work well with `ApplicationSets`, too. Instead of maintaining a patch or overlay for each cluster, patches can now be done in the `Application` template and utilize attributes from the generators. 

For example, with [`external-dns`](https://github.com/kubernetes-sigs/external-dns/) to set the [`txt-owner-id`](https://github.com/kubernetes-sigs/external-dns/blob/e1adc9079b12774cccac051966b2c6a3f18f7872/docs/registry/registry.md?plain=1#L6) to the cluster name.
```
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: external-dns
spec:
  goTemplate: true
  goTemplateOptions: ["missingkey=error"]
  generators:
  - clusters: {}
  template:
    metadata:
      name: 'external-dns'
    spec:
      project: default
      source:
        repoURL: https://github.com/kubernetes-sigs/external-dns/
        targetRevision: v0.14.0
        path: kustomize
        kustomize:
          patches:
          - target:
              kind: Deployment
              name: external-dns
            patch: |-
              - op: add
                path: /spec/template/spec/containers/0/args/3
                value: --txt-owner-id={{.name}}   # patch using attribute from generator
      destination:
        name: 'in-cluster'
        namespace: default

```



## 6.2 patchesStrategicMerge 

==现在已经不怎么使用patchesStrategicMerge 这个关键词了 改用 patches 这个关键词 相关是一样的 ， 见章节 external patch.yaml in patches ==

补丁文件（Patches）可以用来对资源执行不同的定制。 Kustomize 通过 `patchesStrategicMerge` 和 `patchesJson6902` 支持不同的打补丁机制。 `patchesStrategicMerge` 的内容是一个文件路径的列表，其中每个文件都应可解析为 [策略性合并补丁（Strategic Merge Patch）](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-api-machinery/strategic-merge-patch.md)。 补丁文件中的名称必须与已经加载的资源的名称匹配。 建议构造规模较小的、仅做一件事情的补丁。 例如，构造一个补丁来增加 Deployment 的副本个数；构造另外一个补丁来设置内存限制。


patchesStrategicMerge则主要是用于**overlay下的kustomization.yaml中**, 用于使用打patch的方式将该字段指定的文件合并到base目录下对应的内容上.


### 6.2.1 例子1

比如, 现在需要1box环境中，在base声明的deployment.yaml的基础上，给`p-expoter`容器增加一个环境变量
那么就新增overlay/1box/1box-custom-env.yaml

overlay/1box/1box-custom-env.yaml
```
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  name: p-expoter-kustom  
spec:  
  template:  
    spec:  
      containers:  
        - args:  
          - "aaabbb"  
          name: p-expoter # 由于env是在spec.template.containers下的, 因此需要保留这种树型的结构，同时容器的名字很重要，需要与base/deployment.yaml对应上，要不然，会出现env找不到容器，自然无法为对应容器添加成功.  
          env:  
            - name: CUSTOM_ENV_VARIABLE  
              value: "Value defined by Kustomize"
```


> 这里需要注意的是，patch的字段都是通过Yaml的层及进行查询的, 所以要添加或者修改的东西一定要对应上层级，==不然在base里是查找不到相应的字段的, 因此也无法修改, 但是不会报错==

`cat overlay/1box/kustomization.yaml`
```
apiVersion: kustomize.config.k8s.io/v1beta1  
kind: Kustomization  
  
# namePrefix: 1box # 所有的资源名称都会加上1box-  
  
# replicas:  # 直接修改资源副本数  
# - name: p-expoter-kustom  
#   count: 2  
  
# images:   # 直接修改镜像  
# - name: localhost:5055/p-expoter:master-36cd406f  # name 为base/deployment中的镜像的名字  
#   newName: p-expoter   
#   newTag: alpine  # 最终会替换成 images: localhost:5055/p-expoter:alpine  
  
bases:  
- ../../base  
  
patchesStrategicMerge: 
- 1box-custom-env.yaml
```


以上的代码实现修改容器的启动参数, 同时为容器添加env的功能

使用`kustomize build deploy/overlay/1box`会发现，p-expoter容器已经被加上了env环境变量了,**而其它的部分保持不变**.
通过patchesStrategicMerge可以实现，**如果指定的字段不存在,则进行添加， 如果存在，则进行覆盖的功能**

**注意**: 
>如果通过在`kustomization.yaml`修改了同时又使用`patchesStrategicMerge`进行了同一字段的修改, 结果以`kustomization.yaml`中的为准 
比如修改replicas, 如果在`kustomization.yaml中修改成了2， 在`patchesStrategicMerge`又修改成了10` , 结果以`kustomization.yaml`中的为准，也就是最终replicas = 2


### 6.2.2 例子2

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

# 生成一个补丁 increase_replicas.yaml
cat <<EOF > increase_replicas.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
EOF

# 生成另一个补丁 set_memory.yaml
cat <<EOF > set_memory.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  template:
    spec:
      containers:
      - name: my-nginx
        resources:
          limits:
            memory: 512Mi
EOF

cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
patchesStrategicMerge:
- increase_replicas.yaml
- set_memory.yaml
EOF
```

执行 `kubectl kustomize ./` 来查看 Deployment：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      run: my-nginx
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - image: nginx
        name: my-nginx
        ports:
        - containerPort: 80
        resources:
          limits:
            memory: 512Mi
```

## 6.3 patchesJson6902

并非所有资源或者字段都支持策略性合并补丁。为了支持对任何资源的任何字段进行修改， Kustomize 提供通过 `patchesJson6902` 来应用 [JSON 补丁](https://tools.ietf.org/html/rfc6902)的能力。 为了给 JSON 补丁找到正确的资源，需要在 `kustomization.yaml` 文件中指定资源的组（group）、 版本（version）、类别（kind）和名称（name）。 例如，为某 Deployment 对象增加副本个数的操作也可以通过 `patchesJson6902` 来完成：

```shell
# 创建一个 deployment.yaml 文件
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

# 创建一个 JSON 补丁文件
cat <<EOF > patch.yaml
- op: replace
  path: /spec/replicas
  value: 3
EOF

# 创建一个 kustomization.yaml
cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml

patchesJson6902:
- target:
    group: apps
    version: v1
    kind: Deployment
    name: my-nginx
  path: patch.yaml
EOF
```

执行 `kubectl kustomize ./` 以查看 `replicas` 字段被更新：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      run: my-nginx
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - image: nginx
        name: my-nginx
        ports:
        - containerPort: 80
```


# 7 image

除了补丁之外，Kustomize 还提供定制容器镜像或者将其他对象的字段值注入到容器中的能力，并且不需要创建补丁。 例如，你可以通过在 `kustomization.yaml` 文件的 `images` 字段设置新的镜像来更改容器中使用的镜像。


```shell
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

cat <<EOF >./kustomization.yaml
resources:
- deployment.yaml
images:
- name: nginx
  newName: my.image.registry/nginx
  newTag: 1.4.0
EOF
```

执行 `kubectl kustomize ./` 以查看所使用的镜像已被更新：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      run: my-nginx
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - image: my.image.registry/nginx:1.4.0
        name: my-nginx
        ports:
        - containerPort: 80
```


# 8 components

Kustomize [components](https://github.com/kubernetes-sigs/kustomize/blob/master/examples/components.md) encapsulate both resources and patches together. They provide a powerful way to modularize and reuse configuration in Kubernetes applications.

Outside of Argo CD, to utilize components, you must add the following to the `kustomization.yaml` that the Application references. For example:
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
...
components:
- ../component
```



With support added for components in `v2.10.0`, you can now reference a component directly in the Application:
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: application-kustomize-components
spec:
  ...
  source:
    path: examples/application-kustomize-components/base
    repoURL: https://github.com/my-user/my-repo
    targetRevision: main

    # This!
    kustomize:
      components:
        - ../component  # relative to the kustomization.yaml (`source.path`).
```



# 9 kustomization.yaml中贯穿性字段


在项目中为所有 Kubernetes 对象设置贯穿性字段是一种常见操作。 贯穿性字段的一些使用场景如下：
- 为所有资源设置相同的名字空间
- 为所有对象添加相同的前缀或后缀
- 为对象添加相同的标签集合
- 为对象添加相同的注解集合

下面是一个例子：
```shell
# 创建一个 deployment.yaml
cat <<EOF >./deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
EOF

cat <<EOF >./kustomization.yaml
namespace: my-namespace
namePrefix: dev-
nameSuffix: "-001"
commonLabels:
  app: bingo
commonAnnotations:
  oncallPager: 800-555-1212
resources:
- deployment.yaml
EOF
```

执行 `kubectl kustomize ./` 查看这些字段都被设置到 Deployment 资源上：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    oncallPager: 800-555-1212
  labels:
    app: bingo
  name: dev-nginx-deployment-001
  namespace: my-namespace
spec:
  selector:
    matchLabels:
      app: bingo
  template:
    metadata:
      annotations:
        oncallPager: 800-555-1212
      labels:
        app: bingo
    spec:
      containers:
      - image: nginx
        name: nginx
```



## 9.1 commonLables/commonAnnotations
指定了commonLables的话， 则会在所有的资源resources包含的对象上都会加上该label， 支持指定多个，这样就省去了对k8s的yaml中手工地指定label
commonAnnotations同理.


## 9.2 namespace
同上

## 9.3 namePrefix/nameSuffix

这两个实现给所有的资源的名字加上前缀或者后缀，以`-`进行隔离， 非常容易理解，主要用于overlay下
常用的也就这些了，还有一些其它的配置就不一一介绍了，官网上都非常的详细.

有些时候，Pod 中运行的应用可能需要使用来自其他对象的配置值。 例如，某 Deployment 对象的 Pod 需要从环境变量或命令行参数中读取读取 Service 的名称。 由于在 kustomization.yaml 文件中添加 namePrefix 或 nameSuffix 时 Service 名称可能发生变化，建议不要在命令参数中硬编码 Service 名称。 对于这种使用场景，Kustomize 可以通过 vars 将 Service 名称注入到容器中。


```shell
# 创建一个 deployment.yaml 文件（引用此处的文档分隔符）
cat <<'EOF' > deployment.yaml
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
        command: ["start", "--host", "$(MY_SERVICE_NAME)"]
EOF

# 创建一个 service.yaml 文件
cat <<EOF > service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
EOF

cat <<EOF >./kustomization.yaml
namePrefix: dev-
nameSuffix: "-001"

resources:
- deployment.yaml
- service.yaml

vars:
- name: MY_SERVICE_NAME
  objref:
    kind: Service
    name: my-nginx
    apiVersion: v1
EOF
```

执行 `kubectl kustomize ./` 以查看注入到容器中的 Service 名称是 `dev-my-nginx-001`：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dev-my-nginx-001
spec:
  replicas: 2
  selector:
    matchLabels:
      run: my-nginx
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - command:
        - start
        - --host
        - dev-my-nginx-001
        image: nginx
        name: my-nginx
```





# 10 基准（Bases）与覆盖（Overlays） 

Kustomize 中有 基准（bases） 和 覆盖（overlays） 的概念区分。 基准 是包含 kustomization.yaml 文件的一个目录，其中包含一组资源及其相关的定制。 基准可以是本地目录或者来自远程仓库的目录，只要其中存在 kustomization.yaml 文件即可。 覆盖 也是一个目录，其中包含将其他 kustomization 目录当做 bases 来引用的 kustomization.yaml 文件。 基准不了解覆盖的存在，且可被多个覆盖所使用。 覆盖则可以有多个基准，且可针对所有基准中的资源执行组织操作，还可以在其上执行定制。

> 基准不了解覆盖的存在，且可被多个覆盖所使用。 覆盖则可以有多个基准，且可针对所有基准中的资源执行组织操作


```shell
# 创建一个包含基准的目录
mkdir base
# 创建 base/deployment.yaml
cat <<EOF > base/deployment.yaml
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
EOF

# 创建 base/service.yaml 文件
cat <<EOF > base/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
EOF

# 创建 base/kustomization.yaml
cat <<EOF > base/kustomization.yaml
resources:
- deployment.yaml
- service.yaml
EOF
```

此基准可在多个覆盖中使用。你可以在不同的覆盖中添加不同的 `namePrefix` 或其他贯穿性字段。 下面是两个使用同一基准的覆盖：

```shell
mkdir dev
cat <<EOF > dev/kustomization.yaml
resources:
- ../base
namePrefix: dev-
EOF

mkdir prod
cat <<EOF > prod/kustomization.yaml
resources:
- ../base
namePrefix: prod-
EOF
```



# 11 Transformer

向所有资源添加注释（非标识元数据）。和标签一样，它们也是键值对。

```bash
commonAnnotations:
  oncallPager: 800-555-1212
```

## 11.1 通过 `transformers` 字段使用

在 Kustomize 中，`transformers` 字段允许你指定一系列转换器，这些转换器可以对原始的资源清单进行修改和调整。

要在 Kustomize 中使用 `transformers`，你需要在 `kustomization.yaml` 文件中指定它，并列出你要使用的转换器配置文件的路径。

例如：

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml

transformers:
- transformers/add-labels.yaml
- transformers/change-image-tag.yaml
```

在上面的示例中，`add-labels.yaml` 和 `change-image-tag.yaml` 将会作为转换器应用，依次修改 `deployment.yaml` 中的资源。

```yaml
apiVersion: builtin
kind: ImageTagTransformer
metadata:
  name: not-important-to-example
imageTag:
  name: nginx
  newTag: v2
```

### 11.1.1 LabelTransformer

为所有资源和选择器添加标签

```yaml
commonLabels:
  someName: someValue
  owner: alice
  app: bingo
```

### 11.1.2 NamespaceTransformer

将命名空间添加到所有资源

```yaml
namespace: my-namespace
```
