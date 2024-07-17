
kustomize is a command line tool supporting template-free, structured customization of declarative configuration targetted to k8s-style objects.

Kustomize 允许用户以一个应用描述文件 （YAML 文件）为基础（Base YAML），然后通过 Overlay 的方式生成最终部署应用所需的描述文件，而不是像 Helm 那样只提供应用描述文件模板，然后通过字符替换（Templating）的方式来进行定制化

它放弃了对模板的要求，改用 Base + Overlay 的方式对应用的原始 YAML 进行`派生`。`Overlay`，顾名思义，就是覆盖。Kustomize 的 Overlay 可以在 Base 的基础上，通过对 `resource`、`generator`、`transformer` 等的定义，形成新的应用定义，不论 Base 还是 Overlay，都可以通过 `kustomize build` 生成有效的 YAML。

Kustomize 的目标是一样的，但不使用模板。相反，它在一个目录中保留**完整**版本的 YAML 文件。按照惯例，这个文件被称为 `base`，但也可以根据自己的喜好给它命名。然后可以为每个环境/场景/用例创建一个目录(或目录树)，每个目录都需要一个名为 `kustomization.yaml` 的 YAML 文件，该文件的目的是告知 Kustomize 应该考虑哪些 manifest 文件，以及需要对这些文件进行哪些修改。

# 1 Kustomize 的特色

- 功能简单清晰，kubectl 直接支持。
- 不考虑派生，仅作为应用的 YAML 组织方式也很有帮助。
- 也有自己的插件系统。例如可以用简单的 YAML 定义，使用文件生成 Configmap/Secret。


# 2 kustomize 解决了什么问题?

一般应用都会存在多套部署环境：开发环境、测试环境、生产环境，多套环境意味着存在多套 K8S 应用资源 YAML。而这么多套 YAML 之间只存在微小配置差异，比如镜像版本不同、Label 不同等，而这些不同环境下的YAML 经常会因为人为疏忽导致配置错误。再者，多套环境的 YAML 维护通常是通过把一个环境下的 YAML 拷贝出来然后对差异的地方进行修改。一些类似 Helm 等应用管理工具需要额外学习DSL 语法。总结以上，在 k8s 环境下存在多套环境的应用，经常遇到以下几个问题：

- 如何管理不同环境或不同团队的应用的 Kubernetes YAML 资源
- 如何以某种方式管理不同环境的微小差异，使得资源配置可以复用，减少 copy and change 的工作量
- 如何简化维护应用的流程，**不需要额外学习模板语法**

Kustomize 通过以下几种方式解决了上述问题：

- kustomize 通过 Base & Overlays 方式(下文会说明)方式维护不同环境的应用配置
- kustomize 使用 patch 方式复用 Base 配置，并在 Overlay 描述与 Base 应用配置的差异部分来实现资源复用
- kustomize 管理的都是 Kubernetes 原生 YAML 文件，不需要学习额外的 DSL 语法

> 本人总结就是: **不需要通过模板系统,以特定结构合并(更新)多种资源以支撑在多个环境中的部署，以消除差异.**


