

在一些包含YAML资源文件（部署、服务、映射等）的目录中，创建kustomization文件。

当然，Kustomize 和 Helm 可以一起使用，下面是一些使用它们的方法和功能:
1. **HelmChartInflationGenerator**: Kustomize 中内建了一个非常有用的功能叫做 “HelmChartInflationGenerator”，它可以让你在 Kustomize 清单中使用 Helm 图表。当运行 Kustomize 命令时，它会[扩展 Helm 图表以包括 Helm 生成的所有文件](https://medium.com)。
2. **helmCharts 插件**: 你可以直接在 Kustomize 中使用 HelmCharts 插件。例如，你可以将 `values-prod.yaml` 文件放在与 `kustomization.yaml` 文件相同的目录中，然后通过 Kustomize 覆盖 Helm 图表中的默认值。
3. **helm template 和 kubectl kustomize**: 你可以首先使用 `helm template` 命令生成清单，并将其导出到一个文件中，然后运行 `kubectl kustomize` 命令来应用 Kustomize 修改。另一种方式是使用 `helm install` (或 `helm upgrade --install`) 命令，并指定一个自定义的后渲染器来运行 `kubectl kustomize`。
4. **覆盖 Helm 图表**: Kustomize 可以覆盖现有的 Helm 图表，并使用 `HelmChartInflationGenerator` 覆盖一组自定义值。例如，可以使用 Kustomize 部署 Bitnami 的 NGINX Helm 图表，并覆盖默认值以提供自定义的 `nginx.conf` 和自定义的首页。



# 1 ChartInflator插件

>用写好kustomization 文件, 渲染某个已经存在的Helm Charts, 使得这个charts中添加一些内容

Kustomize 提供了一个很好的插件生态系统，允许扩展 Kustomize 的功能。其中就有一个名为 **ChartInflator** 的非内置插件，它允许 Kustomize 来渲染 Helm Charts，并执行任何需要的变更。

**首先先安装 `ChartInflator` 插件：**

```yaml
$ chartinflator_dir="./kustomize/plugin/kustomize.config.k8s.io/v1/chartinflator"

# 创建插件目录
$ mkdir -p ${chartinflator_dir}

# 下载插件
$ curl -L https://raw.githubusercontent.com/kubernetes-sigs/kustomize/kustomize/v3.8.2/plugin/someteam.example.com/v1/chartinflator/ChartInflator > ${chartinflator_dir}/ChartInflator

# 设置插件执行权限
$ chmod u+x ${chartinflator_dir}/ChartInflator
```

比如我们要定制 **Vault Helm Chart** 包，接下来创建 ChartInflator 资源清单和 Helm 的 `values.yaml` 值文件：

```yaml
# ChartInflator 资源清单
$ cat << EOF >> chartinflator-vault.yaml
apiVersion: kustomize.config.k8s.io/v1
kind: ChartInflator
metadata:
  name: vault-official-helm-chart
chartRepo: https://helm.releases.hashicorp.com  
chartName: vault
chartRelease: hashicorp
chartVersion: 0.7.0
releaseName: vault
values: values.yaml
EOF

# 创建 values 值文件
$ helm repo add hashicorp https://helm.releases.hashicorp.com 
$ helm show values --version 0.7.0 hashicorp/vault > values.yaml

# 创建 Kustomize 文件
$ kustomize init
$ cat << EOF >> kustomization.yaml
generators:
- chartinflator-vault.yaml
EOF

# 为所有资源添加一个 label 标签
$ kustomize edit add label env:dev

# 最后生成的 kustomize 文件如下所示：
$ cat kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
generators:
- chartinflator-vault.yaml
commonLabels:
  env: dev

# 整个资源清单目录结构
$ tree .
.
├── chartinflator-vault.yaml
├── kustomization.yaml
├── kustomize
│   └── plugin
│       └── kustomize.config.k8s.io
│           └── v1
│               └── chartinflator
│                   └── ChartInflator
└── values.yaml

5 directories, 4 files
```

现在就可以来渲染 Chart 模板了，执行如下所示的命令即可：

```javascript
$ kustomize build --enable_alpha_plugins .
```

正常渲染完成后我们可以看到所有的资源上都被添加了一个 `env: dev` 的标签，这是实时完成的，不需要维护任何额外的文件的。


# 2 用单个清单文件定制

另一种使用 Kustomize 定制 Chart 的方法是使用 `helm template` 命令来生成一个单一的资源清单，这种方式可以对 Chart 进行更多的控制，但它需要更多的工作来出来处理更新该生成文件的版本控制。

通常我们可以使用 Make 来进行辅助处理，如下示例所示：

```javascript
# Makefile
CHART_REPO_NAME   := hashicorp
CHART_REPO_URL    := https://helm.releases.hashicorp.com
CHART_NAME        := vault
CHART_VERSION     := 0.7.0
CHART_VALUES_FILE := values.yaml

add-chart-repo:
    helm repo add ${CHART_REPO_NAME} ${CHART_REPO_URL}
    helm repo update

generate-chart-manifest:
    helm template ${CHART_NAME} ${CHART_REPO_NAME}/${CHART_NAME} \
        --version ${CHART_VERSION} \
        --values ${CHART_VALUES_FILE} > ${CHART_NAME}.yaml

get-chart-values:
    @helm show values --version ${CHART_VERSION} \
    ${CHART_REPO_NAME}/${CHART_NAME}

generate-chart-values:
    @echo "Create values file: ${CHART_VALUES_FILE}"
    @$(MAKE) -s get-chart-values > ${CHART_VALUES_FILE}

diff-chart-values:
    @echo "Diff: Local <==> Remote"
    @$(MAKE) -s get-chart-values | \
    diff --suppress-common-lines --side-by-side ${CHART_VALUES_FILE} - || \
    exit 0
```

要定制上游的 Vault Helm Chart，我们可以做如下操作：

```javascript
# 初始化 chart 文件
$ make generate-chart-values generate-chart-manifest 

# 创建 Kustomize 文件并添加一个 label 标签
$ kustomize init
$ kustomize edit add resource vault.yaml
$ kustomize edit add label env:dev

# 最后生成的文件结构如下所示
$ tree .
.
├── kustomization.yaml
├── makefile
├── values.yaml
└── vault.yaml

0 directories, 4 files

# kustomize 文件内容如下所示
$ cat kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- vault.yaml
commonLabels:
  env: dev
```

最后同样用 `kustomize build` 命令来渲染：

```javascript
$ kustomize build .
```

在渲染的结果中同样可以看到所有的资源里面都被添加进了一个 `env: dev` 的标签。

这种方法，需要以某种方式运行 make 命令来生成更新的一体化资源清单文件，另外，要将更新过程与你的 GitOps 工作流整合起来可能有点麻烦。

# 3 使用 Post Rendering 定制

**Post Rendering** 是 Helm 3 带来的一个新功能，在前面的2种方法中，Kustomize 是用来处理生成图表清单的主要工具，但在这里，Kustomize 是作为 Helm 的辅助工具而存在的。

下面我们来看下如何使用这种方法来进行定制：

```javascript
# 创建 Kustomize 文件并添加一个 label 标签
$ kustomize init
$ kustomize edit add label env:dev

# 创建一个包装 Kustomize 的脚本文件，后面在 Helm 中会使用到
$ cat << EOF > kustomize-wrapper.sh
#!/bin/bash
cat <&0 > chart.yaml
kustomize edit add resource chart.yaml
kustomize build . && rm chart.yaml
EOF
$ chmod +x kustomize-wrapper.sh
```

然后我们可以直接使用 Helm 渲染或者安装 Chart：

```javascript
$ helm repo add hashicorp https://helm.releases.hashicorp.com 
$ helm template vault hashicorp/vault --post-renderer ./kustomize-wrapper.sh
```

正常情况下我们也可以看到最后渲染出来的每一个资源文件中都被添加进了一个 `env:dev` 的标签。

这种方法就是需要管理一个额外的脚本，其余的和第一种方式基本上差不多，只是不使用 Kustomize 的插件，而是直接使用 Helm 本身的功能来渲染上游的 Chart 包。



