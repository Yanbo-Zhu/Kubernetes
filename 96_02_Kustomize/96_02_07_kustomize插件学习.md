

https://nsddd.top/zh/posts/kubernetes-for-kustomize-learning/


在Kustomize的GitHub仓库中，有一些插件可以用来扩展其功能。以下是对这些插件的简要介绍：

1. **Exec插件**：此插件可以运行可执行脚本作为一个 [插件](https://github.com/badjware/kustomize-plugins)。
2. **RemoteResources生成器**：此插件可以从远程位置下载 [Kubernetes资源](https://github.com/badjware/kustomize-plugins)。
3. **PlaceholderTransformer转换器**：此插件可以在Kubernetes资源中执行任意的键/值替换。
4. **SSMParameterPlaceholderTransformer转换器**：此插件可以在Kubernetes资源中执行任意的键/值替换，并从AWS系统管理器参数中获取值。
5. **EnvironmentPlaceholderTransformer转换器**：此插件可以在Kubernetes资源中执行任意的键/值替换，并从环境变量中获取值。

其他相关信息包括，用户可以创建转换器或生成器插件，以实现新的行为，这通常意味着需要编写代码，例如Go插件、Go二进制文件、C++ [二进制文件或Bash脚本等](https://github.com/kubernetes-sigs/kustomize/blob/master/examples/configureBuiltinPlugin.md)。在2020年3月时，Kustomize的外部插件还处于alpha功能阶段，所以需要使用`--enable_alpha_plugins`标志来调用构建。

同时，还有一些其他的GitHub仓库也提供了Kustomize插件的集合，例如badjware/kustomize-plugins仓库，sapcc/kustomize-plugins仓库和pollination/kustomize-plugins仓库，其中一些插件可以用来生成Kubernetes secrets，从GCP的密封秘密中生成等。

这些插件通过编写代码，使得用户可以扩展Kustomize的功能，以满足特定的需求，例如通过执行任意的键/值替换来修改Kubernetes资源。
