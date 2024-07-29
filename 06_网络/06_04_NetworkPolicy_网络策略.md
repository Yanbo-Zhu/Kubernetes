
# 1 概述

  
- [网络策略](https://kubernetes.io/zh/docs/concepts/services-networking/network-policies/)指的是 Pod 间的网络隔离策略，默认情况下是互通的。

- Pod  之间的互通是通过如下三个标识符的组合来辨识的：
    - ① 其他被允许的 Pod（例外：Pod 无法阻塞对自身的访问）。
    - ② 被允许的名称空间。
    - ③ IP 组块（例外：与 Pod 运行所在的节点的通信总是被允许的， 无论 Pod 或节点的 IP 地址）。

  

![](https://cdn.nlark.com/yuque/0/2022/png/513185/1648107101582-3637c498-471a-41c2-b187-90f04a98e3f5.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_29%2Ctext_6K645aSn5LuZ%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)


# 2 Pod 隔离和非隔离

●  默认情况下，Pod 都是非隔离的（non-isolated），可以接受来自任何请求方的网络请求。 
●  如果一个 NetworkPolicy 的标签选择器选中了某个 Pod，则该 Pod 将变成隔离的（isolated），并将拒绝任何不被 NetworkPolicy 许可的网络连接。 

```

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: networkpol-01
  namespace: default
spec:
  podSelector: # Pod 选择器
    matchLabels:
      app: nginx  # 选中的 Pod 就被隔离起来了
  policyTypes: # 策略类型
  - Ingress # Ingress 入站规则、Egress 出站规则
  - Egress 
  ingress: # 入站白名单，什么能访问我
  - from:
    - podSelector: # 选中的 Pod 可以访问 spec.matchLabels 中筛选的 Pod 
        matchLabels:
          access: granted
    ports:
    - protocol: TCP
      port: 80
  egress: # 出站白名单，我能访问什么
    - to:
       - podSelector: # spec.matchLabels 中筛选的 Pod 能访问的 Pod
          matchLabels:
            app: tomcat
       - namespaceSelector: 
          matchLabels:
            kubernetes.io/metadata.name: dev # spec.matchLabels 中筛选的 Pod 能访问 dev 命名空间下的所有
      ports:
      - protocol: TCP
        port: 8080
```
