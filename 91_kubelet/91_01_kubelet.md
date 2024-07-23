

- 核心文件夹：`/etc/kubernetes` 。
- kubelet 额外参数配置： `/etc/sysconfig/kubelet`。
- kubelet配置位置： `/var/lib/kubelet/config.yaml`。

# 1 kubelet 

kubelet 的配置文件 
[root@master ~]# mkdir -p $HOME/.kube  # kubelet 要读取的配置文件 
[root@master ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@master ~]# sudo chown $(id -u):$(id -g) $HOME/.kube/config

