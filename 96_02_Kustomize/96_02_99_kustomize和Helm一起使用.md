> render Helm charts with Kustomize.

https://github.com/kubernetes-sigs/kustomize/blob/master/examples/chart.md

https://www.baeldung.com/ops/kubernetes-helm-vs-kustomize

# 1 总览 

就是使用kustomize 去补足 由执行helm chart 生成的 Manifest Yaml File

Kustomize is [built](https://kubectl.docs.kubernetes.io/references/kustomize/kustomization) from _generators_ and _transformers_; the former make kubernetes YAML, the latter transform said YAML.
Kustomize, via the `helmCharts` field, has the ability to use the [`helm`](https://helm.sh) command line program in a subprocess to inflate a helm chart, generating YAML as part of (or as the entirety of) a kustomize base.
This YAML can then be modified either in the base directly (transformers always run _after_ generators), or via a kustomize overlay.
Either approach can be viewed as [last mile](https://testingclouds.wordpress.com/2018/07/20/844/) modification of the chart output before applying it to a cluster.
The example below arbitrarily uses the [_minecraft_](https://artifacthub.io/packages/helm/minecraft-server-charts/minecraft) chart pulled from the [artifact hub](https://artifacthub.io) chart repository.



在一些包含YAML资源文件（部署、服务、映射等）的目录中，创建kustomization文件。

当然，Kustomize 和 Helm 可以一起使用，下面是一些使用它们的方法和功能:
1. **HelmChartInflationGenerator**: Kustomize 中内建了一个非常有用的功能叫做 “HelmChartInflationGenerator”，它可以让你在 Kustomize 清单中使用 Helm 图表。当运行 Kustomize 命令时，它会[扩展 Helm 图表以包括 Helm 生成的所有文件](https://medium.com)。
2. **helmCharts 插件**: 你可以直接在 Kustomize 中使用 HelmCharts 插件。例如，你可以将 `values-prod.yaml` 文件放在与 `kustomization.yaml` 文件相同的目录中，然后通过 Kustomize 覆盖 Helm 图表中的默认值。
3. **helm template 和 kubectl kustomize**: 你可以首先使用 `helm template` 命令生成清单，并将其导出到一个文件中，然后运行 `kubectl kustomize` 命令来应用 Kustomize 修改。另一种方式是使用 `helm install` (或 `helm upgrade --install`) 命令，并指定一个自定义的后渲染器来运行 `kubectl kustomize`。
4. **覆盖 Helm 图表**: Kustomize 可以覆盖现有的 Helm 图表，并使用 `HelmChartInflationGenerator` 覆盖一组自定义值。例如，可以使用 Kustomize 部署 Bitnami 的 NGINX Helm 图表，并覆盖默认值以提供自定义的 `nginx.conf` 和自定义的首页。


# 2 这个解释好
https://www.baeldung.com/ops/kubernetes-helm-vs-kustomize

## 2.1 Using Kustomize to Customize Helm Charts[](https://www.baeldung.com/ops/kubernetes-helm-vs-kustomize#1-using-kustomize-to-customize-helm-charts)

**We can use Kustomize to modify and customize Helm chats by applying patches or overlays to the rendered manifests**. This way, we use Helm for packaging and templating while leveraging Kustomize for environment-specific customizations.

Let’s see an example of _kustomization.yaml_ file that references a Helm chart:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- helm-chart.yaml
patchesStrategicMerge:
- patch.yaml
```

In this example, _helm-chart.yaml_ is a rendered Helm chart, and _patch.yaml_ contains the Kustomize patches to modify the chart.

## 2.2 Helm as a Package Manager With Kustomize[](https://www.baeldung.com/ops/kubernetes-helm-vs-kustomize#2-helm-as-a-package-manager-with-kustomize)

**We can also use Helm as a package manager to manage and deploy Helm charts while using Kustomize for further customization**. Thus, we benefit from Helm’s packaging and release management features while still having the flexibility to customize the deployed resources using Kustomize.

Let’s look at an example of a _kustomization.yaml_ file that includes a Helm chart as a resource:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
helmCharts:
- name: myapp
  repo: https://charts.example.com
  version: 1.2.3
  valuesFile: myapp-values.yaml
patchesStrategicMerge:
- patch.yaml
```

In this example, the Helm chart _myapp_ is included as a resource in the _kustomization.yaml_ file specifying the chart repository, version, and values file. Additionally, _patch.yaml_ contains the Kustomize patches to customize the deployed resources.


values的改变 的给可以使用 vulesInline
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

helmCharts:
- name: mlflow-server
  repo: https://strangiato.github.io/helm-charts/
  version: "0.5.7"
  releaseName: mlflow-server
  namespace: my-namespace
  valuesFile: values.yaml
  valuesInline:
    fullnameOverride: helloagain

```

# 3 

https://github.com/kubernetes-sigs/kustomize/blob/master/examples/chart.md#build-the-base-and-the-variants

## 3.1 Preparation

This example defines the `helm` command as

```
helmCommand=${MYGOBIN:-~/go/bin}/helmV3
```

This value is needed for testing this example in CI/CD. A user doesn't need this if their binary is called `helm` and is on their shell's `PATH`.

Make a place to work:

```
DEMO_HOME=$(mktemp -d)
mkdir -p $DEMO_HOME/base $DEMO_HOME/dev $DEMO_HOME/prod
```

## 3.2 Define some variants

1 _development_ variant.

Define a kustomization representing your _development_ variant.

This could involve any number of kustomizations (see other examples), but in this case just add the name prefix '`dev-`' to all resources:

```
cat <<'EOF' >$DEMO_HOME/dev/kustomization.yaml
namePrefix:  dev-
resources:
- ../base
EOF
```



2 production variant.

Likewise define a _production_ variant, with a name prefix '`prod-`':

```
cat <<'EOF' >$DEMO_HOME/prod/kustomization.yaml
namePrefix:  prod-
resources:
- ../base
EOF
```

These two variants refer to a common base.


3  base variant

Define this base the usual way by creating a `kustomization` file:

```
cat <<'EOF' >$DEMO_HOME/base/kustomization.yaml
helmCharts:
- name: minecraft
  includeCRDs: false
  valuesInline:
    minecraftServer:
      eula: true
      difficulty: hard
      rcon:
        enabled: true
  releaseName: moria
  version: 3.1.3
  repo: https://itzg.github.io/minecraft-server-charts
EOF
```

The only thing in this particular file is a `helmCharts` field, specifying a single chart.

The `valuesInline` field overrides some native chart values.

The `includeCRDs` field instructs Helm to generate `CustomResourceDefinitions`. See [the Helm documentation](https://helm.sh/docs/chart_best_practices/custom_resource_definitions/) for details.

Check the directory layout:

```
tree $DEMO_HOME
```

Expect something like:

```
 /tmp/whatever
 ├── base
 │  └── kustomization.yaml
 ├── dev
 │  └── kustomization.yaml
 └── prod
    └── kustomization.yaml
```



### 3.2.1 Helm related flags


Attempt to build the `base`:
```
cmd="kustomize build --helm-command $helmCommand $DEMO_HOME/base"
if ($cmd); then
   echo "Build should fail!" && false  # Force test to fail.
else
   echo "Build failed because no --enable-helm flag (desired outcome)."
fi
```


1 `--enable-helm`
This `build` fails and complains about a missing `--enable-helm` flag.

The flag `--enable-helm` exists to have the user acknowledge that kustomize is running an external program as part of the `build` step. It's like the `--enable-plugins` flag, but with a helm focus.

2 `--helm-command`
The flag `--helm-command` has a default value (`helm` of course) so it's not suitable as an enablement flag. A user with `helm` on their `PATH` need not awkwardly specify `'--helm-command helm'`.


3  定义一个新的命令 
Given the above, define a helper function to run `kustomize` with the flags required for `helm` use in this demo:

```
function kustomizeIt {
  kustomize build \
    --enable-helm \
    --helm-command $helmCommand \
    $DEMO_HOME/$1
}
```

### 3.2.2 Build the base and the variants

1 
Now build the `base`:

```
kustomizeIt base
```

This works, and you see an inflated chart complete with a `Secret`, `Service`, `Deployment`, etc.


2  生成的 yaml 会放到 base 文件夹 下的 名字为charts 这个 subdirectory 文件夹
As a side effect of this build, kustomize pulled the chart and placed it in the `charts` subdirectory of the base. Take a look:
```
tree $DEMO_HOME
```

If the chart had already been there, kustomize would not have tried to pull it.


3  改变生成的yaml 文件的 自动放置的位置 

To change the location of the charts, use this in your kustomization file:

> ```
> helmGlobals:
>  chartHome: charts
> ```

Change `charts` as desired, but it's best to keep it in (or below) the same directory as the `kustomization.yaml` file. If it's outside the kustomization root, the `build` command will fail unless given the flag `'--load-restrictor=none'` to disable file loading restrictions.


3  对比 两个文件的区别 

Now build the two variants `dev` and `prod` and compare their differences:

```
diff <(kustomizeIt dev) <(kustomizeIt prod) | more
```

This shows so-called _last mile hydration_ of two variants made from a common base that happens to be generated from a helm chart.


## 3.3 How does the pull work?

[](https://github.com/kubernetes-sigs/kustomize/blob/master/examples/chart.md#how-does-the-pull-work)

The command kustomize used to download the chart is something like

> ```
> $helmCommand pull \
>    --untar \
>    --untardir $DEMO_HOME/base/charts \
>    --repo https://itzg.github.io/minecraft-server-charts \
>    --version 3.1.3 \
>    minecraft
> ```

1 
The first use of kustomize above (when the `base` was expanded) fetched the chart and placed it in the `charts` directory next to the `kustomization.yaml` file.
```
kustomizeIt base
```


This chart was reused, _not_ re-fetched, with the variant expansions `prod` and `dev`.

2 
如果 chart 已经生成了, 再次执行 `kustomizeIt base` 将不会生成一个新文件, 因为 kustomize 没有Cache, 没有识别文件的版本的方式 
If a chart exists, kustomize will not overwrite it (so to suppress a pull, simply assure the chart is already in your kustomization root). kustomize won't check dates or version numbers or do anything that smells like cache management.

> kustomize is a YAML manipulator. It's not a manager of a cache of things downloaded from the internet.


## 3.4 The pull happens once.

如果你修改了 helm chart 中的 values.yaml 中的值,   再次执行 kustomizeIt prod 后, helm chart 对应的信息 mainifest yaml file 会被成功生成出来, 即使之前 老一个版本的 mainifest yaml file  已经被生成出来了. 


To show that the locally stored chart is being re-used, modify its _values_ file.

First make note of the password encoded in the production inflation:

```
test 1 == $(kustomizeIt prod | grep -c "rcon-password: Q0hBTkdFTUUh")
```
-c：只打印匹配的行数。

The above command succeeds if the value of the generated password is as shown (`Q0hBTkdFTUUh`).

Now change the password in the local values file:

```
values=$DEMO_HOME/base/charts/minecraft-3.1.3/minecraft/values.yaml

grep CHANGEME $values
sed -i 's/CHANGEME/SOMETHING_ELSE/' $values
grep SOMETHING_ELSE $values
```

Run the build, and confirm that the same `rcon-password` field in the output has a new value, confirming that the chart used was a _local_ chart, not a chart freshly downloaded from the internet:

```
test 1 == $(kustomizeIt prod | grep -c "rcon-password: U09NRVRISU5HX0VMU0Uh")
```

Finally, clean up:

```
rm -r $DEMO_HOME
```

## 3.5 Performance

kustomization.yaml 会中给出 一些helm-related fields 比如 
```
cat <<'EOF' >$DEMO_HOME/base/kustomization.yaml
helmCharts:
- name: minecraft
  includeCRDs: false
  valuesInline:
    minecraftServer:
      eula: true
      difficulty: hard
      rcon:
        enabled: true
  releaseName: moria
  version: 3.1.3
  repo: https://itzg.github.io/minecraft-server-charts
EOF
```



To recap, the helm-related kustomization fields make kustomize run

> ```
> helm pull ...
> helm template ...
> ```

_as a convenience for the user_ to generate YAML from a helm chart.

Helm's `pull` command downloads the chart. Helm's `template` command inflates the chart template, spitting the inflated template to stdout (where kustomize captures it) rather than immediately sending it to a cluster as `helm install` would.

上文的意思是, 其实在执行 `kustomize build --enable-helm`  之后,   helm pull 和 helm template 会被默认被执行

---

To improve performance, a user can retain the chart after the first pull (first pull 就是 已经 inflate 这个chart 去 生成一个 yaml  file过了 ), and commit the chart to their configuration repository (below the `kustomization.yaml` file that refers to it). kustomize only tries to pull the chart if it's not already there (只有 已经生成的那个 yaml 的manifest 不存在之后,  kustomize 才会 only tries to pull the chart to inflate it and gererate new manifest   ). 

To further improve performance, a user can inflate the chart themselves at the command line, e.g.

> ```
> helm template {releaseName} \
>     --values {valuesFile} \
>     --version {version} \
>     --repo {repo} \
>     {chartName} > {chartName}.yaml
> ```

then commit the resulting `{chartName}.yaml` file to a git repo as a configuration base, mentioning that file as a `resource` in a `kustomization.yaml` file, e.g.

> ```
> resources:
> - minecraft_v3.1.3_Chart.yaml
> ```


这时候 kustomize 就会根据 minecraft_v3.1.3_Chart.yaml 去生成的新的 manifest file
kustomize不会意识到 the Yaml 已经被helm 生成过了 

The user should choose when or if to refresh their local copy of the chart's inflation. kustomize would have no awareness that the YAML was generated by helm, and kustomize wouldn't run `helm` during the `build`. This is analogous to `Go` module vendoring.




### 3.5.1 But it's not really about performance.

 > render Helm charts with Kustomize. 带来的风险是有的


Although the `helm` related fields discussed above are handy for experimentation and development, it's best to avoid them in production.

The same argument applies to using _remote_ git URL's in other kustomization fields. Handy for experimentation, but ill-advised in production.

It's irresponsible to depend on a remote configuration that's _not under your control_. Annoying enablement flags like `'--enable-helm'` are intended to _remind_ one of a risk, but offer zero protection from risk. 
Further, they are useless are reminders, since **annoying things are immediately scripted away and forgotten**, as was done above in the `kustomizeIt` shell function.


## 3.6 Best practice

[](https://github.com/kubernetes-sigs/kustomize/blob/master/examples/chart.md#best-practice)

Don't use remote configuration that you don't control in production.

Maintain a _local, inflated fork_ of a remote configuration, and have a human rebase / reinflate that fork from time to time to capture upstream changes.



# 4 `kustomize build . --enable-helm`.


From your local environment, you can render the chart by running `kustomize build . --enable-helm`.




# 5 其他

## 5.1 ChartInflator插件

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


## 5.2 用单个清单文件定制

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

## 5.3 使用 Post Rendering 定制

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


## 5.4 enable Kustomizing Helm charts

It's possible to render Helm charts with Kustomize. Doing so requires that you pass the --enable-helm flag to the kustomize build command. This flag is not part of the Kustomize options within Argo CD. 

If you would like to render Helm charts through Kustomize in an Argo CD application, you have two options: You can either create a custom plugin, or modify the `argocd-cm` ConfigMap to include the `--enable-helm` flag globally for all Kustomize applications:


```
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  kustomize.buildOptions: --enable-helm

```

