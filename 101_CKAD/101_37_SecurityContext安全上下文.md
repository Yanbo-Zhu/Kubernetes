

# 1 题目要求

在 **test** 命名空间，有一个名为 **secnginx** 的 pod，修改此 pod，为容器添加**CAP_NET_ADMIN** 和 **CAP_SYS_TIME** 权能

# 2 参考

[https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/security-context/#set-capabilities-for-a-container](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/security-context/#set-capabilities-for-a-container)

```
# 参考
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-4
spec:
  containers:
  - name: sec-ctx-4
    image: gcr.io/google-samples/node-hello:1.0
    securityContext:
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]

```


# 3 解题 

```
# 提前创建ns
kubectl create ns test
# 根据题目要求创建pod,配置
kubectl run secnginx --image=nginx --dry-run=client -o yaml > 26-ecurity-context.yaml

# 修改配置
vim 26-ecurity-context.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  namespace: test # Specify the namespace here
  labels:
    run: secnginx
  name: secnginx # 题目要求
spec:
  containers:
  - image: nginx
    name: secnginx
    securityContext:  # 添加配置
      capabilities:  # 添加配置
        add: ["NET_ADMIN", "SYS_TIME"]  # 添加配置
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

# 应用配置,注意我的配置文件中没有写namespace,下面命令加上了ns
kubectl -n test apply -f  26-ecurity-context.yaml


```