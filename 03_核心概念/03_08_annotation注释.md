

更新一个或多个资源上的批注，Kubernetes 注解（annotations）为资源提供了附加元数据。与标签（labels）不同，注解不用于选择和查找资源。注解可以存储大量的数据，比如使用工具、库等为资源提供的长描述、声明检查的时间戳、联系信息或其他信息。

**是什么？**
Annotation（注解）是一种将非标识性元数据附加到对象的方式。客户端工具和库（如 `kubectl` 和 Helm）可以检索这些元数据。

**注解与标签（Labels）的区别**
虽然注解和标签都用于附加元数据，但它们在目的上有所不同：
- **标签**: 标签是用于选择对象和查找满足某些条件的对象集合。
- **注解**: 注解主要用于存储辅助数据，以便通过工具和库进行检索。


# 1 使用kubectl命令去操作anaotation

**使用 `kubectl` 添加和修改注解**
要使用 `kubectl` 为资源添加注解，你可以使用 `annotate` 命令。例如：

```bash
kubectl annotate pods my-pod example.com/some-annotation="some value"
```

这会为名为 `my-pod` 的 Pod 添加一个名为 `example.com/some-annotation` 的注解，并将其值设置为 “some value”。


**更新和删除注解**
使用同样的 `annotate` 命令，你可以修改或删除注解。例如，要更改上面示例中的注解值，只需再次运行相同的命令，并为其提供一个新值。如果要删除注解，可以使用 `-` 符号：

```
kubectl annotate pods my-pod example.com/some-annotation-
```

**查询使用注解的资源**
尽管你不能直接使用 `kubectl` 查询特定的注解值，但你可以使用 `kubectl get` 命令和 `-o json` 或 `-o yaml` 输出格式选项查看资源的所有注解。

```
kubectl get pods my-pod -o=jsonpath='{.metadata.annotations}'
```
