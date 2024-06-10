
这个没关系的，实际在生产环境中，你不会使用官方提供的 dashboard 的，一般云厂商都提供了交互非常好的界面，而且也有许多开源的组件，比如：kubesphere 、rainbond 等等，官方的 dashboard 只是让你在学习阶段用用而已。


1 Create 
下载 yaml 文件（网速不行，请点这里）：

wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.1/aio/deploy/recommended.yaml

修改 yaml 文件

vim recommended.yaml


```yaml
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort # 新增
  ports:
  - port: 443
    targetPort: 8443
    nodePort: 30009 # 新增
selector:
    k8s-app: kubernetes-dashboard
    
```


kubectl apply -f recommended.yaml

-----------------


2 创建账户，并授权：

```
vim dash-admin.yaml
```


```yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: kubernetes-dashboard
    namespace: kubernetes-dashboard
```



kubectl create -f dash-admin.yaml

------------


令牌：

kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')

访问地址：https://192.168.65.100:30009 。

![](image/Pasted%20image%2020240610191030.png)

![](image/Pasted%20image%2020240610191047.png)



