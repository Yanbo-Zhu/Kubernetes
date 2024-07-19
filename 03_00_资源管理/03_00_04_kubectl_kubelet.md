
- 核心文件夹：`/etc/kubernetes` 。
- kubelet 额外参数配置： `/etc/sysconfig/kubelet`。
- kubelet配置位置： `/var/lib/kubelet/config.yaml`。

# 1 kubelet 

kubelet 的配置文件 
[root@master ~]# mkdir -p $HOME/.kube  # kubelet 要读取的配置文件 
[root@master ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@master ~]# sudo chown $(id -u):$(id -g) $HOME/.kube/config


# 2 kubectl


> 扩展：kubectl可以在node节点上运行吗 ?

kubectl的运行是需要进行配置的，它的配置文件是$HOME/.kube，如果想要在node节点运行此命令，需要将master上的.kube文件复制到node节点上，即在master节点上执行下面操作：

```shell
scp  -r  HOME/.kube   node1: HOME/
```




## 2.1 use kubectl to connect a cluster 

https://confluence.ivu.de/display/SYS/k8s+Single+Node+Clusters+by+Puppet+for+QS24#k8sSingleNodeClustersbyPuppetforQS24-Gettingstartedwiththek8sSingleNodeCluster

1. Log on to the Kubernetes host using SSH. 
2. Become root. 
3. Use command 'kubectl'
4. Alternatively, use 'k0s kc' to make use of the command completion.

To control the cluster with kubectl remotely from another host, perform the following steps.

1. Install kubectl on the host you want to use.

1. For Linux, use [https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/), but stop at (excluding) 'Verify kubectl configuration'.
2. For Windows, use [https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/), but stop at (excluding) 'Verify kubectl configuration'.

3. Create a directory .kube (mind the dot) in the home directory of the user you want to grant control over the cluster.  
    Linux home is `/home/<username> `by default, or /root for root.  
    Windows home is `C:\Users\<username>` by default.
4. Install the configuration file for access to your cluster, either

1. On a single-node AWS system download [`http://e20-<environment>-a01.ivu-cloud.local/kubeconfig.txt`] (`http://e20-%3cenvironment%3e-a01.ivu-cloud.local/kubeconfig.txt` ), rename the resulting file to config and copy it into the directory created in the previous step, or
2. Copy the file /root/.kube/config from the Kubernetes Host into the directory created in the previous step.

6. Use command 'kubectl'.

You are now connected as cluster admin to Kubernetes. Note that in this role, you may not only monitor, but also reconfigure and inherently misconfigure / destroy resources in the Kubernetes cluster. On the other hand, this is a Dedicated Single Node Cluster, hence the only system that may be harmed is this dedicated cluster and the IVU.plan environment within.





