
kustomize is a command line tool supporting template-free, structured customization of declarative configuration targetted to k8s-style objects.

Kustomize 允许用户以一个应用描述文件 （YAML 文件）为基础（Base YAML），然后通过 Overlay 的方式生成最终部署应用所需的描述文件，而不是像 Helm 那样只提供应用描述文件模板，然后通过字符替换（Templating）的方式来进行定制化

它放弃了对模板的要求，改用 Base + Overlay 的方式对应用的原始 YAML 进行`派生`。`Overlay`，顾名思义，就是覆盖。Kustomize 的 Overlay 可以在 Base 的基础上，通过对 `resource`、`generator`、`transformer` 等的定义，形成新的应用定义，不论 Base 还是 Overlay，都可以通过 `kustomize build` 生成有效的 YAML。

Kustomize 的目标是一样的，但不使用模板。相反，它在一个目录中保留**完整**版本的 YAML 文件。按照惯例，这个文件被称为 `base`，但也可以根据自己的喜好给它命名。然后可以为每个环境/场景/用例创建一个目录(或目录树)，每个目录都需要一个名为 `kustomization.yaml` 的 YAML 文件，该文件的目的是告知 Kustomize 应该考虑哪些 manifest 文件，以及需要对这些文件进行哪些修改。

# 1 Kustomize 的特色

- 功能简单清晰，kubectl 直接支持。
- 不考虑派生，仅作为应用的 YAML 组织方式也很有帮助。
- 也有自己的插件系统。例如可以用简单的 YAML 定义，使用文件生成 Configmap/Secret。

- **声明式配置**: 允许你以声明式的方式定义和管理 Kubernetes 对象，例如部署、Daemonsets、服务、ConfigMaps 等，为多个环境提供支持，而无需修改原始的 YAML 文件
- **配置层叠**: 通过利用层叠来保留应用和组件的基本设置，并通过覆盖声明式的 YAML 文档（称为补丁）来选择性地覆盖默认设置
- **集成与独立使用**: Kustomize 可以作为一个独立的工具使用，或者与 kubectl 结合使用。从 `Kubernetes 1.14` 版本开始，kubectl 也开始支持使用 kustomization 文件来管理 Kubernetes 对象


# 2 版本关系 & kubectl 集成


要查找kubectl最新版本中嵌入的kustomize版本，请运行 `kubectl version` ：

```bash
$ kubectl version --short --client
Client Version: v1.26.0
Kustomize Version: v4.5.7
```

kubectl v1.14中添加了v2.0.3的kustomize构建流。kubectl中的kustomize流在kubectl v1.21更新到v4.0.5之前一直冻结在v2.0.3。它将定期更新，这些更新将反映在Kubernetes发布说明中。

|Kubectl version|Kustomize version|
|---|---|
|< v1.14|n/a|
|v1.14-v1.20|v2.0.3 v2.03|
|v1.21|v4.0.5 V4.05|
|v1.22|v4.2.0 v4.2 0|
|v1.23|v4.4.1 V4.1|
|v1.24|v4.5.4|
|v1.25|v4.5.7|
|v1.26|v4.5.7|
|v1.27|v5.0.1|

# 3 kustomize 解决了什么问题?

一般应用都会存在多套部署环境：开发环境、测试环境、生产环境，多套环境意味着存在多套 K8S 应用资源 YAML。而这么多套 YAML 之间只存在微小配置差异，比如镜像版本不同、Label 不同等，而这些不同环境下的YAML 经常会因为人为疏忽导致配置错误。再者，多套环境的 YAML 维护通常是通过把一个环境下的 YAML 拷贝出来然后对差异的地方进行修改。一些类似 Helm 等应用管理工具需要额外学习DSL 语法。总结以上，在 k8s 环境下存在多套环境的应用，经常遇到以下几个问题：

- 如何管理不同环境或不同团队的应用的 Kubernetes YAML 资源
- 如何以某种方式管理不同环境的微小差异，使得资源配置可以复用，减少 copy and change 的工作量
- 如何简化维护应用的流程，**不需要额外学习模板语法**

Kustomize 通过以下几种方式解决了上述问题：

- kustomize 通过 Base & Overlays 方式(下文会说明)方式维护不同环境的应用配置
- kustomize 使用 patch 方式复用 Base 配置，并在 Overlay 描述与 Base 应用配置的差异部分来实现资源复用
- kustomize 管理的都是 Kubernetes 原生 YAML 文件，不需要学习额外的 DSL 语法

> 本人总结就是: **不需要通过模板系统,以特定结构合并(更新)多种资源以支撑在多个环境中的部署，以消除差异.**



# 4 术语

kustomize中有几个常用的关键字术语，这里简单介绍一下，方便后续的使用

## 4.1 **base**
base 指的是一个 kustomization , 任何的 kustomization 包括 overlay (后面提到)，都可以作为另一个 kustomization 的 base (简单理解为基础目录)。base 中描述了共享的内容，如资源和常见的资源配置

指会被其它定制（kustomization）引用的定制
包括Overlay在内的任何定制，都可以作为其它定制的Base
Base 和 Overlay 可以作为 Git 仓库中的唯一内容，用于简单的 GitOps 管理。对仓库的变更可以触发构建、测试以及部署过程
两个kustomization不能形成依赖环

## 4.2 **overlay**

overlay 是一个 kustomization, 它修改(并因此依赖于)另外一个 kustomization. overlay 中的 kustomization指的是一些其它的 kustomization, 称为其 base. 没有 base, overlay 无法使用，并且一个 overlay 可以用作 另一个 overlay 的 base(基础)。简而言之，overlay 声明了与 base 之间的差异。通过 overlay 来维护基于 base 的不同 variants(变体)，例如开发、QA 和生产环境的不同 variants

依赖于其它 Kustomization 的 Kustomization，被它依赖的叫Base

Overlay通常包括多个变体，这些变体可能针对不同的运行环境，例如development、QA、production。变体使用的资源基本是一致的，只有一些简单的差异，例如 Deployment 的实例数量、特定 Pod 的 CPU 资源、ConfigMap 中的数据源定义等


## 4.3 **variant**
将Overlay应用到Base后的产物

variant 是在集群中将 overlay 应用于 base 的结果。例如开发和生产环境都修改了一些共同 base 以创建不同的 variant。这些 variant 使用相同的总体资源，并与简单的方式变化，例如 deployment 的副本数、ConfigMap使用的数据源等。简而言之，variant 是含有同一组 base 的不同 kustomization


## 4.4 **patch**
修改文件的一般说明。文件路径，指向一个声明了 kubernetes API patch 的 YAML 文件

资源补丁，支持两种风格：

1. patchStrategicMerge：这种补丁就像不完整的资源定义，仅仅包含部分字段。缺省情况下，对应字段会被覆盖，如果目标字段是字符串，是适当的。如果目标字段是列表，则行为可能不符合预期。可以使用[指令](https://git.k8s.io/community/contributors/devel/sig-api-machinery/strategic-merge-patch.md)来修改默认行为
2. patchJson6902：基于[JSONPatch](https://tools.ietf.org/html/rfc6902)方式来修改资源
## 4.5 kustomization

可以指 kustomization.yaml 这个文件，或者是包含了 kustomization.yaml 及其所有直接引用文件的相对路径

一个定制可以是：

    名为kustomization.yaml 的文件
    压缩包（包含 YAML 文件以及它的引用文件）
    Git 压缩包
    Git 仓库的 URL

kustomization.yaml文件的字段，分为四类：

    resources：待定制的现存资源，包括resources、crds等字段
    generator：将要创建的新资源，包括configMapGenerator、secretGenerator、generators等字段
    transformer：对前面提到的新旧资源进行处理的方式。包括namePrefix、nameSuffix、images、commonLabels 等
    meta：会对上面几种字段产生影响。包括vars、namespace、apiVersion、kind 等

## 4.6 Kustomization Root

直接包含kustomization.yaml文件的目录

