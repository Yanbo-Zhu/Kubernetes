
部署 kubernetes-dashboard，參考 官方 說明

```
~# kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml  
kubectl apply -f kubernetes-dashboard.yaml  
secret/kubernetes-dashboard-certs unchanged  
secret/kubernetes-dashboard-csrf unchanged  
serviceaccount/kubernetes-dashboard unchanged  
role.rbac.authorization.k8s.io/kubernetes-dashboard-minimal unchanged  
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard-minimal unchanged  
deployment.apps/kubernetes-dashboard created  
service/kubernetes-dashboard unchanged  
  
~# kubectl get pods -n kube-system | grep dashboard  
... 略 ...  
kubernetes-dashboard-5dd89b9875-5mx9n        1/1       Running   0          16s  
... 略 ...  
  
## 啟動 proxy  
~# kubectl proxy  
  
## 開啟網頁瀏覽以下位址  
# http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/  
# 透過 RBAC 取得 Token 登入
```

# 1 使用 RBAC (Role-Base Access Control)

參考：[https://github.com/kubernetes/dashboard/wiki/Creating-sample-user](https://github.com/kubernetes/dashboard/wiki/Creating-sample-user)

admin.yaml
```
# admin-user.yaml  
apiVersion: v1  
kind: ServiceAccount  
metadata:  
  name: admin-user  
  namespace: kube-system  
  
# admin-role.yaml  
apiVersion: rbac.authorization.k8s.io/v1  
kind: ClusterRoleBinding  
metadata:  
  name: admin-user  
roleRef:  
  apiGroup: rbac.authorization.k8s.io  
  kind: ClusterRole  
  name: cluster-admin  
subjects:  
- kind: ServiceAccount  
  name: admin-user  
  namespace: kube-system
```


然後執行 kubectl apply -f admin.yaml，好了之後取得 admin-user token
```
~$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')  
Name:         admin-user-token-txkst  
Namespace:    kube-system  
Labels:       <none>  
Annotations:  kubernetes.io/service-account.name: admin-user  
              kubernetes.io/service-account.uid: fb1c2dcb-5ebf-11e9-8d1e-92cde7b04430  
  
Type:  kubernetes.io/service-account-token  
  
Data  
====  
ca.crt:     1025 bytes  
namespace:  11 bytes  
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5
```


複製 token 回到 dashboard 選擇 token 方式登入。












