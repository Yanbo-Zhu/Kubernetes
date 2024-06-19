

kubectl 是一个命令行工具，用于与Kubernetes集群和其中的 pod 通信。使用它你可以查看集群的状态，列出集群中的所有 pod，进入 pod 中执行命令等。你还可以使用 YAML 文件定义资源对象，然后使用kubectl 将其应用到集群中。

查看节点状态：

`kubectl get node`

![](https://cdn.nlark.com/yuque/0/2022/png/28915315/1663748785931-438ecd5b-bee8-49c9-997c-9ffc5d1f7b1f.png)

`kubectl get pod -A`

![](https://cdn.nlark.com/yuque/0/2022/png/28915315/1663748833731-5fc73596-d048-4f97-9ce1-bf4d95fb388a.png)

`minikube ssh`进入容器，查看kubelet状态`systemctl status kubelet`

![](https://cdn.nlark.com/yuque/0/2022/png/28915315/1663748921853-09286afb-7cbc-4f52-9f97-4d86f9a66c0d.png)


```powershell
# 1、查看pod 状态的的命令
kubectl get pod -n kube-system -o wide
# 2、删除pod
kubectl delete pod/kub-flannel-ds-xxxx -n kube-system --grace-period=0 --force
# 3、生成 新的token
[root@master ~]# kubeadm token create --print-join-command
```
