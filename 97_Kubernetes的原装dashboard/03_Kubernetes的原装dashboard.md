
之前在kubernetes中完成的所有操作都是通过命令行工具kubectl完成的。其实，为了提供更丰富的用户体验，kubernetes还开发了一个基于web的用户界面（Dashboard）。用户可以使用Dashboard部署容器化的应用，还可以监控应用的状态，执行故障排查以及管理kubernetes中各种资源。

这个没关系的，实际在生产环境中，你不会使用官方提供的 dashboard 的，一般云厂商都提供了交互非常好的界面，而且也有许多开源的组件，比如：kubesphere 、rainbond 等等，官方的 dashboard 只是让你在学习阶段用用而已。

https://gitee.com/yooome/golang/blob/main/22-k8s%E8%AF%A6%E7%BB%86%E6%95%99%E7%A8%8B-%E8%B0%83%E6%95%B4%E7%89%88/k8s%E8%AF%A6%E7%BB%86%E6%95%99%E7%A8%8B.md#10-dashboard



# 1 部署Dashboard

## 1.1 下载yaml，并运行Dashboard

```
# 下载yaml
[root@k8s-master01 ~]# wget  https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
# 修改kubernetes-dashboard的Service类型
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort  # 新增
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30009  # 新增
  selector:
    k8s-app: kubernetes-dashboard
    
# 部署
[root@k8s-master01 ~]# kubectl create -f recommended.yaml

# 查看namespace下的kubernetes-dashboard下的资源
[root@k8s-master01 ~]# kubectl get pod,svc -n kubernetes-dashboard
NAME                                            READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-c79c65bb7-zwfvw   1/1     Running   0          111s
pod/kubernetes-dashboard-56484d4c5-z95z5        1/1     Running   0          111s
NAME                               TYPE       CLUSTER-IP      EXTERNAL-IP  PORT(S)         AGE
service/dashboard-metrics-scraper  ClusterIP  10.96.89.218    <none>       8000/TCP        111s
service/kubernetes-dashboard       NodePort   10.104.178.171  <none>       443:30009/TCP   111s
```

## 1.2 创建 admin 

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

## 1.3 创建访问账户，获取token

```
# 创建账号
[root@k8s-master01-1 ~]# kubectl create serviceaccount dashboard-admin -n kubernetes-dashboard

# 授权
[root@k8s-master01-1 ~]# kubectl create clusterrolebinding dashboard-admin-rb --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:dashboard-admin

# 获取账号token
[root@k8s-master01 ~]#  kubectl get secrets -n kubernetes-dashboard | grep dashboard-admin
dashboard-admin-token-xbqhh        kubernetes.io/service-account-token   3      2m35s
[root@k8s-master01 ~]# kubectl describe secrets dashboard-admin-token-xbqhh -n kubernetes-dashboard
Name:         dashboard-admin-token-xbqhh
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: dashboard-admin
              kubernetes.io/service-account.uid: 95d84d80-be7a-4d10-a2e0-68f90222d039
Type:  kubernetes.io/service-account-token
Data
====
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6ImJrYkF4bW5XcDhWcmNGUGJtek5NODFuSXl1aWptMmU2M3o4LTY5a2FKS2cifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtYWRtaW4tdG9rZW4teGJxaGgiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiOTVkODRkODAtYmU3YS00ZDEwLWEyZTAtNjhmOTAyMjJkMDM5Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmVybmV0ZXMtZGFzaGJvYXJkOmRhc2hib2FyZC1hZG1pbiJ9.NAl7e8ZfWWdDoPxkqzJzTB46sK9E8iuJYnUI9vnBaY3Jts7T1g1msjsBnbxzQSYgAG--cV0WYxjndzJY_UWCwaGPrQrt_GunxmOK9AUnzURqm55GR2RXIZtjsWVP2EBatsDgHRmuUbQvTFOvdJB4x3nXcYLN2opAaMqg3rnU2rr-A8zCrIuX_eca12wIp_QiuP3SF-tzpdLpsyRfegTJZl6YnSGyaVkC9id-cxZRb307qdCfXPfCHR_2rt5FVfxARgg_C0e3eFHaaYQO7CitxsnIoIXpOFNAR8aUrmopJyODQIPqBWUehb7FhlU1DCduHnIIXVC_UICZ-MKYewBDLw
ca.crt:     1025 bytes
```

令牌：

kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')



----
方法2

To log on to the Kubernetes dashboard, you need a long-living token which you may fetch using kubectl as configured in the previous section. 

1. Run the following command to fetch the token:  
    `kubectl get secret admin-user -n kubernetes-dashboard -o jsonpath={".data.token"} | base64 -d`
2. Copy the displayed token and store it. It can be used to access the Kubernetes Dashboard from now on. 
3. Open a browser and connect to the Kubernetes host. Replace $kubernetes_host with the FQDN of the host.  
    `https://$kubernetes_host:31443`  
    Ignore the warning regarding the certificate.
4. Paste the token into the respective field.
    1. ![](image/image-2023-12-29_14-47-13.png)


You are now connected as cluster admin to the Kubernetes Dashboard. Note that in this role, you may not only monitor, but also reconfigure and inherently misconfigure / destroy resources in the Kubernetes cluster. On the other hand, this is a Dedicated Single Node Cluster, hence the only system that may be harmed is this dedicated cluster and the IVU.plan environment within.


## 1.4 通过浏览器访问Dashboard的UI

在登录页面上输入上面的token
访问地址：https://192.168.65.100:30009 。

![](../98_Addons/image/Pasted%20image%2020240610191030.png)

![image-20200520144548997](https://gitee.com/yooome/golang/raw/main/22-k8s%E8%AF%A6%E7%BB%86%E6%95%99%E7%A8%8B-%E8%B0%83%E6%95%B4%E7%89%88/Kubenetes.assets/image-20200520144548997.png)

出现下面的页面代表成功

![image-20200520144959353](https://gitee.com/yooome/golang/raw/main/22-k8s%E8%AF%A6%E7%BB%86%E6%95%99%E7%A8%8B-%E8%B0%83%E6%95%B4%E7%89%88/Kubenetes.assets/image-20200520144959353.png)


# 2 使用DashBoard




![](../98_Addons/image/Pasted%20image%2020240610191047.png)

**查看**

选择指定的命名空间`dev`，然后点击`Deployments`，查看dev空间下的所有deployment

![img](https://gitee.com/yooome/golang/raw/main/22-k8s%E8%AF%A6%E7%BB%86%E6%95%99%E7%A8%8B-%E8%B0%83%E6%95%B4%E7%89%88/Kubenetes.assets/image-20200520154628679.png)

**扩缩容**

在`Deployment`上点击`规模`，然后指定`目标副本数量`，点击确定

![img](https://gitee.com/yooome/golang/raw/main/22-k8s%E8%AF%A6%E7%BB%86%E6%95%99%E7%A8%8B-%E8%B0%83%E6%95%B4%E7%89%88/Kubenetes.assets/image-20200520162605102.png)

**编辑**

在`Deployment`上点击`编辑`，然后修改`yaml文件`，点击确定

![image-20200520163253644](https://gitee.com/yooome/golang/raw/main/22-k8s%E8%AF%A6%E7%BB%86%E6%95%99%E7%A8%8B-%E8%B0%83%E6%95%B4%E7%89%88/Kubenetes.assets/image-20200520163253644.png)

**查看Pod**

点击`Pods`, 查看pods列表

![img](https://gitee.com/yooome/golang/raw/main/22-k8s%E8%AF%A6%E7%BB%86%E6%95%99%E7%A8%8B-%E8%B0%83%E6%95%B4%E7%89%88/Kubenetes.assets/image-20200520163552110.png)

**操作Pod**

选中某个Pod，可以对其执行日志（logs）、进入执行（exec）、编辑、删除操作

![img](https://gitee.com/yooome/golang/raw/main/22-k8s%E8%AF%A6%E7%BB%86%E6%95%99%E7%A8%8B-%E8%B0%83%E6%95%B4%E7%89%88/Kubenetes.assets/image-20200520163832827.png)

> Dashboard提供了kubectl的绝大部分功能，这里不再一一演示

