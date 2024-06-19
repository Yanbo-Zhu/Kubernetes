

  

# 1 概述

  
- Kubernetes 网络解决四个方面的问题：
- ① 一个 Pod 中容器之间通过`本地回路（loopback）`通信。
- ② 集群网络在不同 Pod 之间提供通信；换言之，Pod 和 Pod 之间能互相通信（通过 calico 网络插件实现 Pod 之间网络的扁平化；当然，Node 节点之间的通信也是通过 calico 网络插件）。
- ③ Service 资源允许我们对外暴露 Pod 中运行的应用程序，以支持来自集群之外的访问；换言之，Service 和 Pod 之间能互相通信。
- ④ 可以使用 Service 来发布仅供集群内部使用的服务。

  

![](https://cdn.nlark.com/yuque/0/2022/png/513185/1648106563764-6b1d4643-eef3-41b6-b836-6114582550f2.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_40%2Ctext_6K645aSn5LuZ%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

  

# 2 Kubernetes 网络架构图

## 2.1 架构图

  

![](https://cdn.nlark.com/yuque/0/2022/jpeg/513185/1648106578792-fd51f183-8d08-4cba-a3db-a11be6c798cf.jpeg?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_156%2Ctext_6K645aSn5LuZ%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

  
## 2.2 访问流程



![](https://cdn.nlark.com/yuque/0/2022/png/513185/1648106586547-007fd6bc-549b-46d7-bf04-3a86ee5ae85b.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_28%2Ctext_6K645aSn5LuZ%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)







