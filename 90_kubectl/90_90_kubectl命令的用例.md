

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


# 1 Case

## 1.1 Explore Kubernetes resources

```
# List all namespaces
kubectl get namespace
 
# List everything in namespace kubernetes-dashboard (-n kubernetes-dashboard)
kubectl get all -n kubernetes-dashboard
 
# List all pods in namespace kubernetes-dashboard (-n ...)
kubectl get pods -n kubernetes-dashboard
 
# List all pods in all namespaces (-A)
kubectl get pods -A
 
# Get further information on a particular Pod
kubectl describe pod kubernetes-dashboard-78f87ddfc-nbw49 -n kubernetes-dashboard
 
# List all deployments in default namespace
kubectl get deployment
 
# Get further information on a particular Deployment
kubectl describe deployment test-rail-rail-foreground
 
# ... in a similar way with the combination of get and describe, further resources can be explored, like:
# service, configmap, secret, replicaset, node, daemonset, ...
 
# Get events
kubectl events -A
```

Alternatively, you can click through the Kubernetes Dashboard and view Kubernetes resources. Remember to select the right namespace on the upper left, or "All namespaces".


## 1.2 Check the status of Pods

Use the following commands

```
# List all pods in all namespaces (-A)
kubectl get pods -A
 
# List pods in default namespace (Note: '-n default' can be omitted since it is the default)
kubectl get pods -n default
 
# Example output:
NAME                                         READY   STATUS    RESTARTS       AGE
test-rail-rail-foreground-57bc46df68-whb69   1/1     Running   1 (6d1h ago)   22d
```

If READY indicates all pods are ready and STATUS indicates they are Running, everything should be fine. 


## 1.3 Analyse failing Pods

If a Pod fails or causes issues, try to get more information to idenfity the root cause: 

```
# Show pod details
kubectl describe pod test-rail-rail-foreground-57bc46df68-whb69 -n default
# Here you can see volumes, secrets, configmaps and configuration used by the pod,
# as well as associated events, which allow further inspection. Often this helps to
# understand what could be wrong.
 
# Show logs of pod
kubectl logs test-rail-rail-foreground-57bc46df68-whb69 -n default
```

## 1.4 Restart a Pod
In some cases you may want to restart a Pod. Refer to the following page to find various ways how to do that: [https://spacelift.io/blog/restart-kubernetes-pods-with-kubectl](https://spacelift.io/blog/restart-kubernetes-pods-with-kubectl)


## 1.5 Rebuild the whole cluster

This will remove the Kubernetes cluster and all configuration and resources in it completely. The next Puppet run on the Kubernetes host will restore its initial state, undoing all manual modifications. 

This instruction is specific to the Kubernetes distro 'k0s'.

Execute the following commands on the Kubernetes Single Node Cluster host. Ignore potential error messages. 
```
puppet agent --disable
systemctl stop k0scontroller.service
k0s stop
k0s reset
rm -rf /etc/k0s
rm -rf /etc/k8s
rm -rf /root/.kube
puppet agent --enable
```

After the next puppet run, the Cluster and the IVU.plan deployment inside will be rebuilt with its original configuration. 

# 2 Shell into the container

To get a shell access to the container running inside the application pod, all you have to do is:

`kubectl exec -ti --namespace <your namespace> <your pod name> -- sh`

This will open a shell inside of your application container. You can now execute any command you need.

