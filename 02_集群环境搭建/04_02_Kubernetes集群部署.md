
> 该笔记中省略了很多步骤, 详细的见 https://www.yuque.com/fairy-era/yg511q/wuhwio#faec20d6

# 1 最终目标

- 在所有节点上安装Docker 和kubeadm
- 部署Kubernetes Master
- 部署容器网络插件
- 部署Kubernetes Node，将节点加入Kubernetes 集群中
- 部署Dashboard Web 页面，可视化查看Kubernetes 资源


# 2 Kubernetes 和 Docker 之间的版本对应关系

- [官方文档地址](https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG)。

![](https://cdn.nlark.com/yuque/0/2022/png/513185/1646803776210-6d82129c-9501-4847-9e30-c2978a7d0d82.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_55%2Ctext_6K645aSn5LuZ%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

  

从文档中，我们可以知道 Docker 的版本是 v20.10 ，对应的 Kubernetes 的版本是 v1.21 。


# 3 主机安装 

![](image/Pasted%20image%2020240610160307.png)

| 角色     | IP地址      | 组件                              |
| :------- | :---------- | :-------------------------------- |
| master01 | 192.168.5.3 | docker，kubectl，kubeadm，kubelet |
| node01   | 192.168.5.4 | docker，kubectl，kubeadm，kubelet |
| node02   | 192.168.5.5 | docker，kubectl，kubeadm，kubelet |


# 4 环境初始化

## 4.1 升级系统内核


检查操作系统的版本

```powershell
# 此方式下安装kubernetes集群要求Centos版本要在7.5或之上
[root@master ~]# cat /etc/redhat-release
Centos Linux 7.5.1804 (Core)
```

  
- 查看当前系统的版本：


```
cat /etc/redhat-release
```

  

![](https://cdn.nlark.com/yuque/0/2022/gif/513185/1646803808153-9bbdab71-9ded-4788-aac3-115966be62bd.gif)


  

- 查看当前系统的内核：
```
uname -sr
```

```
# 查看启动顺序
yum install -y grub2-pc
grub2-editenv list
```

```
# 查看可用内核版本及启动顺序
sudo awk -F\' '$1=="menuentry " {print i++ " : " $2}' /boot/grub2/grub.cfg
```

![8.gif](https://cdn.nlark.com/yuque/0/2022/gif/513185/1646803813902-abd33b92-34a5-41da-bc2c-473d5dc83300.gif)



## 4.2 关闭防火墙

- 如果是虚拟机则需要让三台机器互通，最简单的做法就是关闭防火墙。

```
systemctl stop firewalld
```

```
systemctl disable firewalld
```

  

![](https://cdn.nlark.com/yuque/0/2022/gif/513185/1646803784469-61bd5119-b316-406f-9935-0db4bec4d218.gif)

  
- 如果是云厂商提供的云服务器则需要让三台机器的私有 IP 互通，不懂的，请查看[《云平台操作》](https://www.yuque.com/fairy-era/yg511q/qia5qz)，而且还需要设置安全组策略放行指定的端口


## 4.3 主机名解析

为了方便集群节点间的直接调用，在这个配置一下主机名解析，企业中推荐使用内部DNS服务器

```powershell
# 主机名成解析 编辑三台服务器的/etc/hosts文件，添加下面内容
192.168.90.100 master
192.168.90.106 node1
192.168.90.107 node2
```

## 4.4 时间同步

kubernetes要求集群中的节点时间必须精确一直，这里使用chronyd服务从网络同步时间

企业中建议配置内部的会见同步服务器

```powershell
# 启动chronyd服务
[root@master ~]# systemctl start chronyd
[root@master ~]# systemctl enable chronyd
[root@master ~]# date
```

## 4.5 禁用iptable和firewalld服务

kubernetes和docker 在运行的中会产生大量的iptables规则，为了不让系统规则跟它们混淆，直接关闭系统的规则

```powershell
# 1 关闭firewalld服务
[root@master ~]# systemctl stop firewalld
[root@master ~]# systemctl disable firewalld
# 2 关闭iptables服务
[root@master ~]# systemctl stop iptables
[root@master ~]# systemctl disable iptables
```

## 4.6 禁用selinux

selinux是linux系统下的一个安全服务，如果不关闭它，在安装集群中会产生各种各样的奇葩问题

```powershell
# 编辑 /etc/selinux/config 文件，修改SELINUX的值为disable
# 注意修改完毕之后需要重启linux服务
SELINUX=disabled
```

## 4.7 禁用swap分区

swap分区指的是虚拟内存分区，它的作用是物理内存使用完，之后将磁盘空间虚拟成内存来使用，启用swap设备会对系统的性能产生非常负面的影响，因此kubernetes要求每个节点都要禁用swap设备，但是如果因为某些原因确实不能关闭swap分区，就需要在集群安装过程中通过明确的参数进行配置说明

```powershell
# 编辑分区配置文件/etc/fstab，注释掉swap分区一行
# 注意修改完毕之后需要重启linux服务
vim /etc/fstab
注释掉 /dev/mapper/centos-swap swap
# /dev/mapper/centos-swap swap
```

## 4.8 修改linux的内核参数

```powershell
# 修改linux的内核采纳数，添加网桥过滤和地址转发功能
# 编辑/etc/sysctl.d/kubernetes.conf文件，添加如下配置：
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1

# 重新加载配置
[root@master ~]# sysctl -p
# 加载网桥过滤模块
[root@master ~]# modprobe br_netfilter
# 查看网桥过滤模块是否加载成功
[root@master ~]# lsmod | grep br_netfilter
```

## 4.9 配置ipvs功能

在Kubernetes中Service有两种带来模型，一种是基于iptables的，一种是基于ipvs的两者比较的话，ipvs的性能明显要高一些，但是如果要使用它，需要手动载入ipvs模块

```powershell
# 1.安装ipset和ipvsadm
[root@master ~]# yum install ipset ipvsadm -y
# 2.添加需要加载的模块写入脚本文件
[root@master ~]# cat <<EOF> /etc/sysconfig/modules/ipvs.modules
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
# 3.为脚本添加执行权限
[root@master ~]# chmod +x /etc/sysconfig/modules/ipvs.modules
# 4.执行脚本文件
[root@master ~]# /bin/bash /etc/sysconfig/modules/ipvs.modules
# 5.查看对应的模块是否加载成功
[root@master ~]# lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

# 5 安装docker

```powershell
# 1、切换镜像源
[root@master ~]# wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo

# 2、查看当前镜像源中支持的docker版本
[root@master ~]# yum list docker-ce --showduplicates

# 3、安装特定版本的docker-ce
# 必须制定--setopt=obsoletes=0，否则yum会自动安装更高版本
[root@master ~]# yum install --setopt=obsoletes=0 docker-ce-18.06.3.ce-3.el7 -y

# 4、添加一个配置文件
#Docker 在默认情况下使用Vgroup Driver为cgroupfs，而Kubernetes推荐使用systemd来替代cgroupfs
[root@master ~]# mkdir /etc/docker
[root@master ~]# cat <<EOF> /etc/docker/daemon.json
{
	"exec-opts": ["native.cgroupdriver=systemd"],
	"registry-mirrors": ["https://kn0t2bca.mirror.aliyuncs.com"]
}
EOF

# 5、启动dokcer
[root@master ~]# systemctl restart docker
[root@master ~]# systemctl enable docker
```



● 三台机器上都安装 Docker 。
● 卸载旧版本：

```
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```


![24.gif](https://cdn.nlark.com/yuque/0/2022/gif/513185/1646804009924-2284a5f0-6f76-4bd0-93fe-215820dc3961.gif)



# 6 安装Kubernetes组件

```powershell
# 1、由于kubernetes的镜像在国外，速度比较慢，这里切换成国内的镜像源
# 2、编辑/etc/yum.repos.d/kubernetes.repo,添加下面的配置
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgchech=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
			http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg

# 3、安装kubeadm、kubelet和kubectl
[root@master ~]# yum install --setopt=obsoletes=0 kubeadm-1.17.4-0 kubelet-1.17.4-0 kubectl-1.17.4-0 -y
# 或者 yum install -y kubelet-1.21.10 kubeadm-1.21.10 kubectl-1.21.10

# 4、配置kubelet的cgroup
# 为了实现 Docker 使用的 cgroup drvier 和 kubelet 使用的 cgroup drver 一致，建议修改 /etc/sysconfig/kubelet 文件的内容：
#编辑/etc/sysconfig/kubelet, 添加下面的配置
KUBELET_CGROUP_ARGS="--cgroup-driver=systemd"
KUBE_PROXY_MODE="ipvs"

# 5、设置kubelet开机自启
[root@master ~]# systemctl enable kubelet
```





![36.gif](https://cdn.nlark.com/yuque/0/2022/gif/513185/1646804110728-a5b9b17c-0c89-403a-91e7-332ac29c82b0.gif)


![37.gif](https://cdn.nlark.com/yuque/0/2022/gif/513185/1646804116700-fc2f02c0-cce5-447d-96b0-6f8264c3980e.gif)


# 7 集群安装

## 7.1 准备集群镜像

添加阿里云的 Kubernetes 的 YUM 源
● 由于 Kubernetes 的镜像源在国外，非常慢，这里切换成国内的阿里云镜像源（三台机器均需执行下面命令）：

```
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

```


![](https://cdn.nlark.com/yuque/0/2022/gif/513185/1646804094469-cc9f5cce-2941-4fce-8caa-695b242e9c45.gif)



查看和下载 Kubernetes 安装所需镜像

```powershell
# 在安装kubernetes集群之前，必须要提前准备好集群需要的镜像，所需镜像可以通过下面命令查看
[root@master ~]# kubeadm config images list

```

![](image/Pasted%20image%2020240610165605.png)



```powershell
# 下载镜像
# 此镜像kubernetes的仓库中，由于网络原因，无法连接，下面提供了一种替换方案
images=(
	kube-apiserver:v1.17.4
	kube-controller-manager:v1.17.4
	kube-scheduler:v1.17.4
	kube-proxy:v1.17.4
	pause:3.1
	etcd:3.4.3-0
	coredns:1.6.5
)

for imageName in ${images[@]};do
	docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
	docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
	docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName 
done

```

![](image/Pasted%20image%2020240610165714.png)



给 coredns 镜像重新打 tag ：
```
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:v1.8.0 registry.cn-hangzhou.aliyuncs.com/google_containers/coredns/coredns:v1.8.0
```


也可以使用如下的命令快速安装：
```
kubeadm config images pull --kubernetes-version=v1.21.10 \
--image-repository registry.aliyuncs.com/google_containers
```


## 7.2 部署 Kubernetes 的 Master 节点

>下面的操作只需要在master节点上执行即可/ 部署 Kubernetes 的 Master 节点

```powershell
# 创建集群
[root@master ~]# kubeadm init \
	--apiserver-advertise-address=192.168.90.100 \  # nmaster 节点的ip地址 可能需要修改 
	--image-repository registry.aliyuncs.com/google_containers \
	--kubernetes-version=v1.17.4 \
	--service-cidr=10.96.0.0/12 \
	--pod-network-cidr=10.244.0.0/16

# 创建必要文件
[root@master ~]# mkdir -p $HOME/.kube  # kubelet 要读取的配置文件 
[root@master ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@master ~]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


注意：
● apiserver-advertise-address 一定要是主机的 IP 地址。 
● apiserver-advertise-address 、service-cidr 和 pod-network-cidr 不能在同一个网络范围内。
● 不要使用 172.17.0.1/16 网段范围，因为这是 Docker 默认使用的。


日志：
```shell

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.65.100:6443 --token tluojk.1n43p0wemwehcmmh \
	--discovery-token-ca-cert-hash sha256:c50b25a5e00e1a06cef46fa5d885265598b51303f1154f4b582e0df21abfa7cb

```


根据日志提示操作，在 192.168.65.100 执行如下命令：
```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```shell
# 如果是 root 用户，还可以执行如下命令
export KUBECONFIG=/etc/kubernetes/admin.conf
```


默认的 token 有效期为 24 小时，当过期之后，该 token 就不能用了，这时可以使用如下的命令创建 token 

```
kubeadm token create --print-join-command
```


```
# 生成一个永不过期的token
kubeadm token create --ttl 0 --print-join-command
```

## 7.3 部署 Kubernetes 的 Node节点


> 下面的操作只需要在node节点上执行即可

将node 加入 cluster 
```powershell
kubeadm join 192.168.0.100:6443 --token awk15p.t6bamck54w69u4s8 \
    --discovery-token-ca-cert-hash sha256:a94fa09562466d32d29523ab6cff122186f1127599fa4dcd5fa0152694f17117 
```

在master上查看节点信息

```powershell
[root@master ~]# kubectl get nodes
NAME    STATUS   ROLES     AGE   VERSION
master  NotReady  master   6m    v1.17.4
node1   NotReady   <none>  22s   v1.17.4
node2   NotReady   <none>  19s   v1.17.4
```

## 7.4 安装网络插件，只在master节点操作即可


![](image/Pasted%20image%2020240610170605.png)

NotReady 是因为还没有为 cluster 安装网络 


● Kubernetes 支持多种网络插件，比如 flannel、calico、canal 等，任选一种即可，本次选择 calico（在 192.168.65.100 节点上执行，网络不行，请点这里
```powershell
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

![](image/Pasted%20image%2020240610170748.png)



由于外网不好访问，如果出现无法访问的情况，可以直接用下面的 记得文件名是kube-flannel.yml，位置：/root/kube-flannel.yml内容：
```powershell
https://github.com/flannel-io/flannel/tree/master/Documentation/kube-flannel.yml
```


也可手动拉取指定版本
docker pull quay.io/coreos/flannel:v0.14.0              #拉取flannel网络，三台主机
docker images                  #查看仓库是否拉去下来


```个人笔记```
若是集群状态一直是 notready,用下面语句查看原因，
journalctl -f -u kubelet.service
若原因是： cni.go:237] Unable to update cni config: no networks found in /etc/cni/net.d
mkdir -p /etc/cni/net.d                    #创建目录给flannel做配置文件
vim /etc/cni/net.d/10-flannel.conf         #编写配置文件
```powershell

{
 "name":"cbr0",
 "cniVersion":"0.3.1",
 "type":"flannel",
 "deledate":{
    "hairpinMode":true,
    "isDefaultGateway":true
  }

}

```


安装网络插件, 使用配置文件启动 flannel 
kubectl apply -f kube-flannel.yml

![](image/Pasted%20image%2020240610171003.png)

查看部署 CNI 网络插件进度：
kubectl get pods -n kube-system
watch kubectl get pods -n kube-system

![45.gif](https://cdn.nlark.com/yuque/0/2022/gif/513185/1646804222817-801217df-dae8-46b9-a47b-687cd7aec013.gif)


## 7.5 查看节点状态

在 Master（192.168.65.100）节点上查看节点状态：

> kubectl get nodes

![46.gif](https://cdn.nlark.com/yuque/0/2022/gif/513185/1646804227768-fa477b14-6942-460b-93db-efdb9737f0d6.gif)


## 7.6 设置 kube-proxy 的 ipvs 模式

● 在 Master（192.168.65.100）节点设置 kube-proxy 的 ipvs 模式：

```
kubectl edit cm kube-proxy -n kube-system
```


```yaml
apiVersion: v1
data:
  config.conf: |-
    apiVersion: kubeproxy.config.k8s.io/v1alpha1
    bindAddress: 0.0.0.0
    bindAddressHardFail: false
    clientConnection:
      acceptContentTypes: ""
      burst: 0
      contentType: ""
      kubeconfig: /var/lib/kube-proxy/kubeconfig.conf
      qps: 0
    clusterCIDR: 10.244.0.0/16
    configSyncPeriod: 0s
    conntrack:
      maxPerCore: null
      min: null
      tcpCloseWaitTimeout: null
      tcpEstablishedTimeout: null
    detectLocalMode: ""
    enableProfiling: false
    healthzBindAddress: ""
    hostnameOverride: ""
    iptables:
      masqueradeAll: false
      masqueradeBit: null
      minSyncPeriod: 0s
      syncPeriod: 0s
    ipvs:
      excludeCIDRs: null
      minSyncPeriod: 0s
      scheduler: ""
      strictARP: false
      syncPeriod: 0s
      tcpFinTimeout: 0s
      tcpTimeout: 0s
      udpTimeout: 0s
    kind: KubeProxyConfiguration
    metricsBindAddress: ""
    mode: ""
    nodePortAddresses: null
      minSyncPeriod: 0s
      syncPeriod: 0s
    ipvs:
      excludeCIDRs: null
      minSyncPeriod: 0s
      scheduler: ""
      strictARP: false
      syncPeriod: 0s
      tcpFinTimeout: 0s
      tcpTimeout: 0s
      udpTimeout: 0s
    kind: KubeProxyConfiguration
    metricsBindAddress: ""
    mode: "ipvs" # 修改此处
...
```


删除 kube-proxy ，让 Kubernetes 集群自动创建新的 kube-proxy ：
```
kubectl delete pod -l k8s-app=kube-proxy -n kube-system
```



## 7.7 让 Node 节点也能使用 kubectl 命令

默认情况下，只有 Master 节点才有 kubectl 命令，但是有些时候我们也希望在 Node 节点上执行 kubectl 命令：

```
# 192.168.65.101 和 192.168.65.102
mkdir -pv ~/.kube
touch ~/.kube/config

# 192.168.65.100
scp /etc/kubernetes/admin.conf root@192.168.65.101:~/.kube/config

# 192.168.65.100
scp /etc/kubernetes/admin.conf root@192.168.65.102:~/.kube/config

```

## 7.8 使用kubeadm reset重置集群

```
#在master节点之外的节点进行操作
kubeadm reset
systemctl stop kubelet
systemctl stop docker
rm -rf /var/lib/cni/
rm -rf /var/lib/kubelet/*
rm -rf /etc/cni/
ifconfig cni0 down
ifconfig flannel.1 down
ifconfig docker0 down
ip link delete cni0
ip link delete flannel.1
##重启kubelet
systemctl restart kubelet
##重启docker
systemctl restart docker
```

## 7.9 重启kubelet和docker

```powershell
# 重启kubelet
systemctl restart kubelet
# 重启docker
systemctl restart docker
```

使用配置文件启动fannel

```powershell
kubectl apply -f kube-flannel.yml
```

等待它安装完毕 发现已经是 集群的状态已经是Ready

![](image/Pasted%20image%2020240610162646.png)
## 7.10 kubeadm中的命令

```powershell
# 生成 新的token
[root@master ~]# kubeadm token create --print-join-command
```


# 8 集群测试

## 8.1 创建一个nginx服务

```powershell
kubectl create deployment nginx  --image=nginx:1.14-alpine
```

## 8.2 暴露端口

```powershell
kubectl expose deployment nginx  --port=80 --target-port=80  --type=NodePort
```

## 8.3 查看服务

```powershell
kubectl get pod,svc   svc就是 service , 写成 service 也行 
```


![](image/Pasted%20image%2020240610171539.png)

30518 是暴露给外部的端口号 
通过cluster 中的 node 的ip 加上 30518 端口号 , 就可以访问 nginx 的80 端口号 
## 8.4 查看pod

![](image/Pasted%20image%2020240610163619.png)

浏览器测试结果：

![](image/Pasted%20image%2020240610163636.png)




