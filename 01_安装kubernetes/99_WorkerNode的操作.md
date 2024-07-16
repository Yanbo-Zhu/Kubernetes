

# 1 刪除 Worker Node

- 了解如何刪除 worker node
- 了解刪除過程中會發生的事情

打算刪除 `k8s14-worker02-u1604` 這個 worker node，先檢查現況

```
# 1. 確認現在的 node  
~$ k get no  
NAME                   STATUS   ROLES    AGE     VERSION  
k8s14-master01-u1604   Ready    master   10d     v1.14.0  
k8s14-worker02-u1604   Ready    <none>   4m31s   v1.14.0  
k8s14-worker03-u1604   Ready    <none>   32m     v1.14.0  
k8s14-worker04-u1604   Ready    <none>   24m     v1.14.0  
  
## 檢查有哪些 pod 跑在 worker02  
~$ k get po -o wide | grep worker02  
kube-proxy-zx6fn   1/1     Running   0    3m53s   192.168.2.16   k8s14-worker02-u1604  
weave-net-fkb5w    2/2     Running   1    3m53s   192.168.2.16   k8s14-worker02-u1604  
  
## 2. 到 worker02 裡面，檢查 docker ps  
root@k8s14-worker02-u1604:~# docker ps  
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS               NAMES  
239c836beb6a        047e0878ff14           "/tini -- /usr/local…"   3 minutes ago       Up 3 minutes                            k8s_rook-ceph-agent_rook-ceph-agent-bsgqk_rook-ceph_71d9fc74-605e-11e9-a972-92cde7b04430_1  
fe68d8f88ed1        1f394ae9e226           "/home/weave/launch.…"   3 minutes ago       Up 3 minutes                            k8s_weave_weave-net-fkb5w_kube-system_6bd4a2fa-605e-11e9-a972-92cde7b04430_1  
ae00e498728f        789b7f496034           "/usr/bin/weave-npc"     4 minutes ago       Up 4 minutes                            k8s_weave-npc_weave-net-fkb5w_kube-system_6bd4a2fa-605e-11e9-a972-92cde7b04430_0  
... 略 ...
```

開始執行刪除 worker node，主要步驟如下：
1. Drain Node
2. Delete Node

```
# 1. Drain Node, 同時刪除一些資料  
~$ kubectl drain k8s14-worker02-u1604 --delete-local-data --force --ignore-daemonsets  
node/k8s14-worker02-u1604 cordoned  
WARNING: Ignoring DaemonSet-managed pods: kube-proxy-zx6fn, weave-net-fkb5w, rook-ceph-agent-bsgqk, rook-discover-p5ptq, weave-scope-agent-wsmd2  
pod/rook-ceph-mon-b-775ff945c5-rzzt6 evicted  
  
## 1-1. 確認狀態  
~$ kubectl get no  
NAME                   STATUS                     ROLES    AGE     VERSION  
k8s14-master01-u1604   Ready                      master   10d     v1.14.0  
k8s14-worker02-u1604   Ready,SchedulingDisabled   <none>   8m36s   v1.14.0  
k8s14-worker03-u1604   Ready                      <none>   36m     v1.14.0  
k8s14-worker04-u1604   Ready                      <none>   28m     v1.14.0  
  
## 1-2. Begin of Checking: 沒啥變化  
### 檢查跑在 worker02 上的 pod  
~$ k get po -o wide | grep worker02  
kube-proxy-zx6fn       1/1     Running   0    8m56s   192.168.2.16   k8s14-worker02-u1604  
weave-net-fkb5w        2/2     Running   1    8m56s   192.168.2.16   k8s14-worker02-u1604  
  
### 到 worker02 裡面，檢查 docker ps  
root@k8s14-worker02-u1604:~# docker ps  
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS               NAMES  
239c836beb6a        047e0878ff14           "/tini -- /usr/local…"   3 minutes ago       Up 3 minutes                            k8s_rook-ceph-agent_rook-ceph-agent-bsgqk_rook-ceph_71d9fc74-605e-11e9-a972-92cde7b04430_1  
fe68d8f88ed1        1f394ae9e226           "/home/weave/launch.…"   3 minutes ago       Up 3 minutes                            k8s_weave_weave-net-fkb5w_kube-system_6bd4a2fa-605e-11e9-a972-92cde7b04430_1  
ae00e498728f        789b7f496034           "/usr/bin/weave-npc"     4 minutes ago       Up 4 minutes                            k8s_weave-npc_weave-net-fkb5w_kube-system_6bd4a2fa-605e-11e9-a972-92cde7b04430_0  
... 略 ...  
## End of Checking: 沒啥變化  
  
  
# 2. 刪除 worker01 node  
~$ kubectl delete node k8s14-worker02-u1604  
node "k8s14-worker02-u1604" deleted  
  
## 2-1. 檢查 node  
~$ k get no  
NAME                   STATUS   ROLES    AGE   VERSION  
k8s14-master01-u1604   Ready    master   10d   v1.14.0  
k8s14-worker03-u1604   Ready    <none>   42m   v1.14.0  
k8s14-worker04-u1604   Ready    <none>   33m   v1.14.0  
  
  
## 2-2. 到 worker02 裡執行 kubeadm reset 清除資料, 如下:  
root@k8s14-worker02-u1604:~# kubeadm reset  
[reset] WARNING: Changes made to this host by 'kubeadm init' or 'kubeadm join' will be reverted.  
[reset] Are you sure you want to proceed? [y/N]: y  
[preflight] Running pre-flight checks  
W0416 23:57:25.138327   18457 reset.go:234] [reset] No kubeadm config, using etcd pod spec to get data directory  
[reset] No etcd config found. Assuming external etcd  
[reset] Please manually reset etcd to prevent further issues  
[reset] Stopping the kubelet service  
[reset] unmounting mounted directories in "/var/lib/kubelet"  
[reset] Deleting contents of stateful directories: [/var/lib/kubelet /etc/cni/net.d /var/lib/dockershim /var/run/kubernetes]  
[reset] Deleting contents of config directories: [/etc/kubernetes/manifests /etc/kubernetes/pki]  
[reset] Deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]  
  
The reset process does not reset or clean up iptables rules or IPVS tables.  
If you wish to reset iptables, you must do so manually.  
For example:  
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X  
  
If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)  
to reset your systems IPVS tables.  
  
  
### (optional) 到 worker02 機器裡面看看 docker ps, container 很快就刪完了，沒有 container 再跑了  
root@k8s14-worker02-u1604:~# docker ps  
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS               NAMES  
bf7310c8d3c6        d9ece03f45e7           "/home/weave/entrypo…"   9 minutes ago       Up 9 minutes                            k8s_scope-agent_weave-scope-agent-wsmd2_weave_6bd48236-605e-11e9-a972-92cde7b04430_0  
79d7b777a525        k8s.gcr.io/pause:3.1   "/pause"                 9 minutes ago       Up 9 minutes                            k8s_POD_weave-scope-agent-wsmd2_weave_6bd48236-605e-11e9-a972-92cde7b04430_0  
  
root@k8s14-worker02-u1604:~# docker ps  
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES  
root@k8s14-worker02-u1604:~#  
  
### (optional) docker images 還在  
root@k8s14-worker02-u1604:~# docker images  
REPOSITORY                                          TAG                 IMAGE ID            CREATED             SIZE  
rook/ceph                                           master              047e0878ff14        3 days ago          698MB  
wordpress                                           latest              837092bc87de        5 days ago          421MB  
istio/proxyv2                                       1.1.2               c7fb421f087e        12 days ago         378MB  
... 略 ...
```




