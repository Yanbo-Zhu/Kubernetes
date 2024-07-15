

# 1 部署 Persistent Volume（PV）和 Persistent Volume Claim (PVC)

主要針對 K8s 的 PV / PVC 套件安裝，安裝之後，才能夠跑 `StatefulSet`。
這邊安裝的是 `rook`，參考 [官方文件](https://rook.io/docs/rook/v0.9/ceph-quickstart.html)


## 1.1 建立 namespace、CRD、RBAC 相關資源

第一部會建立 namespace 給 rook 使用，然後建立 RBAC 相關資源，包含以下：
- `Namespace`: rook-ceph
- `CRD (Custom Resource Definition)`:
    - cephclusters
    - cephfilesystems
    - cephnfses
    - cephobjectstores
    - cephobjectstoreusers
    - cephblockpools
    - volumes
- `RBAC 相關`:
    - `ClusterRole`: rook-ceph-cluster-mgmt, rook-ceph-global, rook-ceph-mgr-cluster, rook-ceph-mgr-system
    - `Role`: rook-ceph-system, rook-ceph-osd, rook-ceph-mgr
    - `ServiceAccount`: rook-ceph-system, rook-ceph-osd, rook-ceph-mgr
    - `RoleBinding`: rook-ceph-system, rook-ceph-cluster-mgmt, rook-ceph-osd, rook-ceph-mgr, rook-ceph-mgr-system
    - `ClusterRoleBinding`: rook-ceph-global, rook-ceph-mgr-cluster


```
~# kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/common.yaml  
namespace/rook-ceph created  
  
customresourcedefinition.apiextensions.k8s.io/cephclusters.ceph.rook.io created  
customresourcedefinition.apiextensions.k8s.io/cephfilesystems.ceph.rook.io created  
customresourcedefinition.apiextensions.k8s.io/cephnfses.ceph.rook.io created  
customresourcedefinition.apiextensions.k8s.io/cephobjectstores.ceph.rook.io created  
customresourcedefinition.apiextensions.k8s.io/cephobjectstoreusers.ceph.rook.io created  
customresourcedefinition.apiextensions.k8s.io/cephblockpools.ceph.rook.io created  
customresourcedefinition.apiextensions.k8s.io/volumes.rook.io created  
  
clusterrole.rbac.authorization.k8s.io/rook-ceph-cluster-mgmt created  
role.rbac.authorization.k8s.io/rook-ceph-system created  
clusterrole.rbac.authorization.k8s.io/rook-ceph-global created  
clusterrole.rbac.authorization.k8s.io/rook-ceph-mgr-cluster created  
serviceaccount/rook-ceph-system created  
rolebinding.rbac.authorization.k8s.io/rook-ceph-system created  
clusterrolebinding.rbac.authorization.k8s.io/rook-ceph-global created  
serviceaccount/rook-ceph-osd created  
serviceaccount/rook-ceph-mgr created  
role.rbac.authorization.k8s.io/rook-ceph-osd created  
clusterrole.rbac.authorization.k8s.io/rook-ceph-mgr-system created  
role.rbac.authorization.k8s.io/rook-ceph-mgr created  
rolebinding.rbac.authorization.k8s.io/rook-ceph-cluster-mgmt created  
rolebinding.rbac.authorization.k8s.io/rook-ceph-osd created  
rolebinding.rbac.authorization.k8s.io/rook-ceph-mgr created  
rolebinding.rbac.authorization.k8s.io/rook-ceph-mgr-system created  
clusterrolebinding.rbac.authorization.k8s.io/rook-ceph-mgr-cluster created
```


## 1.2 建立 deployment: rook

建立相關的 pods
```
~# kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/operator.yaml  
  
## 第一次會跑一下子， rook/ceph 大小約 700MB  
~# kubectl get po -n rook-ceph  
NAME                                  READY   STATUS    RESTARTS   AGE  
rook-ceph-agent-dcgb8                 1/1     Running   0          2m  
rook-ceph-agent-jcpmd                 1/1     Running   0          2m  
rook-ceph-operator-65b65fbd66-9m58m   1/1     Running   0          3m35s  
rook-discover-88mf5                   1/1     Running   0          2m  
rook-discover-wggnl                   1/1     Running   0          2m
```

這次的部署包含 agent, operator, discover, 等待所有的 pod 都是 running 狀態後，就可以部署 Ceph Cluster.






## 1.3 部署 Ceph Cluster

部署 Ceph Cluster

```
~# kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/cluster.yaml  
cephcluster.ceph.rook.io/rook-ceph created  
  
## 等一下，會出現 rook-ceph-mon-a  
~# kubectl get po -n rook-ceph  
rook-ceph-agent-dcgb8                              1/1     Running     0          127m  
rook-ceph-agent-jcpmd                              1/1     Running     2          127m  
rook-ceph-mgr-a-77d8645896-8blh4                   1/1     Running     0          118m  
rook-ceph-mon-a-9cbbbf7b-2w6zk                     1/1     Running     1          123m  
rook-ceph-mon-b-775ff945c5-77vtv                   1/1     Running     0          122m  
rook-ceph-mon-c-59695fb97b-drnww                   1/1     Running     1          122m  
rook-ceph-operator-65b65fbd66-9m58m                1/1     Running     1          129m  
rook-discover-88mf5                                1/1     Running     0          127m  
rook-discover-wggnl                                1/1     Running     1          127m
```


完成部署後，會出現以下：
- Ceph Monitor: 三個
- Ceph Manager: 一個

Deployment
```
~$ kubectl get deploy -n rook-ceph  
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE  
rook-ceph-mgr-a      1/1     1            1           119m  
rook-ceph-mon-a      1/1     1            1           124m  
rook-ceph-mon-b      1/1     1            1           123m  
rook-ceph-mon-c      1/1     1            1           123m  
rook-ceph-operator   1/1     1            1           130m
```



















