
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



