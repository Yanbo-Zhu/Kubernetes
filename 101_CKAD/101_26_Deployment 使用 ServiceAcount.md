
更新在 namespace frontend 中的 Deployment 使其使用现有的 ServiceAccount app

[https://kubernetes.io/zh-cn/docs/concepts/security/service-accounts/](https://kubernetes.io/zh-cn/docs/concepts/security/service-accounts/)


# 1 解题 

环境准备

```
# 创建ns 和 serviceaccount
cat serviceaccount.yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: frontend
spec: {}
status: {}
---
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: null
  name: app
  namespace: frontend
# 创建一个测试应用
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: frontend-deployment
  name: frontend-deployment
  namespace: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend-deployment
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: frontend-deployment
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}


```


1  修改serviceaccount
kubectl -n frontend get deployments.apps
kubectl -n frontend set serviceaccount deployments frontend-deployment app
kubectl -n frontend describe deployments.apps frontend-deployment

2 或者直接编辑
kubectl -n frontend edit deployment frontend-deployment
增加
       serviceAccount: app
       serviceAccountName: app