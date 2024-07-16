
https://rickhw.github.io/2019/03/17/Container/Install-K8s-with-Kubeadm/


本文整理的是半手動的安裝筆記，也就是以 `kubeadm` 為主，嘗試過的排列組合 (K8s version x OS x Hypervisor) 如下：

- QNAP Virtualzation Station
    - Ubuntu 20.04:
        - 1.21.1
        - 1.23.4 (2022/03)
    - Ubuntu 18.04:
        - 1.16.14
        - 1.18.0, 1.18.6
        - 1.14.7, 1.14.9
- Proxmox
    - Ubuntu 18.04:
        - 1.18.0, 1.18.6
        - 1.14.7, 1.14.9
    - Ubuntu 16.04:
        - 1.11.3
- VMWare Fusion (macOS)
    - Ubuntu 18.04:
        - 1.18.0, 1.18.6
        - 1.14.7, 1.14.9
    - Ubuntu 16.04:
        - 1.11.3
- AWS EC2
    - Ubuntu 16.04:
        - 1.11.3



# 1 準備 Base Image

這個 Base Image 會用在安裝 Master / Worker Node.

- 虛擬環境：
    - 在 VMWare 上，將 Guest OS 的網路設定為 Bridge Mode。
    - [Proxmox](https://www.proxmox.com/) 上，同樣將 Guest OS 的網路設定為 Bridge Mode
- 安裝 Ubuntu
    - 關閉 swap: `swapoff -a`
    - 註解 `/etc/fstab`
- 注意：
    - LAN 裡面的 DHCP Lease Time 可以調大一點，避免太早過期，下次 Cluster 跑不起來。
    - 如果要使用 PV 的話，Disk Size 不能太小，建議 80GiB 以上。
    - VMWare 如果使用 Linked Clone 會造成 Disk Size 無法調整。


## 1.1 安裝 CRI: Docker-CE

Container Rumtime Interface 

因為 `CRI (Container Runtime Interface)` 最近已經從 CNCF 畢業了，所以除了 Docker，還有其他可以選擇。但這裡還是先以 docker 為主。

以下參考自：[https://kubernetes.io/docs/setup/cri/](https://kubernetes.io/docs/setup/cri/)

```
docker 之外其他 CRI 選擇，有名的如 CRI-O , containerd, rtk, kata container 
```

```
# Install Docker CE
## Set up the repository:
### Update the apt package index
apt-get update

### Install packages to allow apt to use a repository over HTTPS
apt-get install apt-transport-https ca-certificates curl software-properties-common

### Add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

### Add docker apt repository.
add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"

## Install docker ce.
apt-get update
# 指定版本, v18.06.2
apt-get install docker-ce=18.06.2~ce~3-0~ubuntu
# 安裝最新版
# apt-get install docker-ce

# Setup daemon.
## Log 的大小, Storage Driver
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

mkdir -p /etc/systemd/system/docker.service.d

# Restart docker.
systemctl daemon-reload
systemctl restart docker
```



確認版本:
```
~# docker version  
Client:  
 Version:           18.06.2-ce  
 API version:       1.38  
 Go version:        go1.10.3  
 Git commit:        6d37f41  
 Built:             Sun Feb 10 03:48:06 2019  
 OS/Arch:           linux/amd64  
 Experimental:      false  
  
Server:  
 Engine:  
  Version:          18.06.2-ce  
  API version:      1.38 (minimum version 1.12)  
  Go version:       go1.10.3  
  Git commit:       6d37f41  
  Built:            Sun Feb 10 03:46:30 2019  
  OS/Arch:          linux/amd64  
  Experimental:     false
```






## 1.2 安裝 kubeadm, kubectl, kubelet

主要參考自 [官方文件](https://kubernetes.io/docs/setup/independent/install-kubeadm/) ，不過最新版 kubeadm 無法順利初始化，所以這個紀錄以 `1.11.3` 為範例。

準備 repository:
```
apt-get update  
apt-get install -y apt-transport-https curl  
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -  
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list  
deb https://apt.kubernetes.io/ kubernetes-xenial main  
EOF  
apt-get update
```


指定 kubeadm 版本：
```
## 找到可用的版本  
apt-cache madison kubeadm  
  
## 指定版本  
K_VER="1.14.9-00"  
  
## ubuntu 16.04 之後  
apt install -y kubelet=${K_VER} kubectl=${K_VER} kubeadm=${K_VER}  
# 或者使用 apt-get  
#apt-get install -y kubelet=${K_VER} kubectl=${K_VER} kubeadm=${K_VER}
```


安裝最新版本 kubeadm 如下：
```
apt install -y kubelet kubeadm kubectl  
# 鎖定版本，避免 apt upgrade 被更新了  
## 如果是自己安裝的 Base Image，記得一定要做這件事情，不然可能有些 Node 會自己升級 K8s 版本。  
apt-mark hold kubelet kubeadm kubectl
```


## 1.3 確認安裝版本：kubeadm / kubectl

如果是安裝指定版本，範例是 v1.11

```
~# kubeadm version  
kubeadm version: &version.Info{Major:"1", Minor:"11", GitVersion:"v1.11.3", GitCommit:"a4529464e4629c21224b3d52edfe0ea91b072862", GitTreeState:"clean", BuildDate:"2018-09-09T17:59:42Z", GoVersion:"go1.10.3", Compiler:"gc", Platform:"linux/amd64"}  
  
~# kubectl version  
Client Version: version.Info{Major:"1", Minor:"11", GitVersion:"v1.11.3", GitCommit:"a4529464e4629c21224b3d52edfe0ea91b072862", GitTreeState:"clean", BuildDate:"2018-09-09T18:02:47Z", GoVersion:"go1.10.3", Compiler:"gc", Platform:"linux/amd64"}
```


最新版的確認 (v1.13, 2019/03/07):
```
~# kubeadm version  
kubeadm version: &version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.4", GitCommit:"c27b913fddd1a6c480c229191a087698aa92f0b1", GitTreeState:"clean", BuildDate:"2019-02-28T13:35:32Z", GoVersion:"go1.11.5", Compiler:"gc", Platform:"linux/amd64"}  
  
~# kubectl version  
Client Version: version.Info{Major:"1", Minor:"13", GitVersion:"v1.13.4", GitCommit:"c27b913fddd1a6c480c229191a087698aa92f0b1", GitTreeState:"clean", BuildDate:"2019-02-28T13:37:52Z", GoVersion:"go1.11.5", Compiler:"gc", Platform:"linux/amd64"}
```


## 1.4 預先下載 docker image

後面會執行 master, worker node 安裝，而實際上還是要先 pull docker image，這個動作可以先下載好：

```
~# kubeadm config images pull  
I1122 23:17:55.359113    2290 version.go:240] remote version is much newer: v1.16.3; falling back to: stable-1.14  
[config/images] Pulled k8s.gcr.io/kube-apiserver:v1.14.9  
[config/images] Pulled k8s.gcr.io/kube-controller-manager:v1.14.9  
[config/images] Pulled k8s.gcr.io/kube-scheduler:v1.14.9  
[config/images] Pulled k8s.gcr.io/kube-proxy:v1.14.9  
[config/images] Pulled k8s.gcr.io/pause:3.1  
[config/images] Pulled k8s.gcr.io/etcd:3.3.10  
[config/images] Pulled k8s.gcr.io/coredns:1.3.1  
  
~# docker images  
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE  
k8s.gcr.io/kube-proxy                v1.14.9             636041c2a488        9 days ago          82.1MB  
k8s.gcr.io/kube-apiserver            v1.14.9             5811259ed0c9        9 days ago          209MB  
k8s.gcr.io/kube-controller-manager   v1.14.9             07193a77f264        9 days ago          157MB  
k8s.gcr.io/kube-scheduler            v1.14.9             0f036524b7a2        9 days ago          81.6MB  
k8s.gcr.io/coredns                   1.3.1               eb516548c180        10 months ago       40.3MB  
k8s.gcr.io/etcd                      3.3.10              2c4adeb21b4f        11 months ago       258MB  
k8s.gcr.io/pause                     3.1                 da86e6ba6ca1        23 months ago       742kB
```

## 1.5 製作 Base Image

完成上述動作，將機器關機，製作成 Base Image.

- Proxmox: convert to template



# 2 安裝 Kubernetes Cluster

- 安裝 Master Node
- 加入 Worker Node

## 2.1 安裝 Master Node

為了便於辨識，先改機器名字：

ubuntu 16.04:
1. `/etc/hostname`: k8s-master01-u1604
2. `/etc/hosts`: k8s-master01-u1604
3. `reboot`

ubuntu 18.04:
1. 關閉 cloud-init 自動更改 hostname 的設定： `/etc/cloud/cloud.cfg` ，把 preserve_hostname 改成 false.
    - 或者關閉 `cloud-init`: `systemctl disable cloud-init`
2. `hostnamectl set-hostname u1804-k118-m01`
3. `reboot`

### 2.1.1 初始化 kubeadm

在 Master Node 裡，用以下指令開始初始 kubeadmin，其中注意參數 `‐‐pod‐network‐cidr` 的指定。
```
# 針對網路設定，有底下的初始化：預設、指定 flannel 的網段
# 不指定參數，使用預設
kubeadm init
#  --kubernetes-version=v1.14.7  # 指定 k8s 版本

# CNI: for flannel
#kubeadm init \
#    --pod-network-cidr=10.244.0.0/16 \
#    --kubernetes-version=v1.14.7

# CNI: for calico
# 注意：如果你的 Node 網路 CIDR 已經在 192.168.0.0/16，請避開，避免安裝有問題。
# kubeadm init \
#    --pod-network-cidr=192.168.244.0/24 \
#     --kubernetes-version=v1.14.7 \

# 安裝出現以下 error:
#    error execution phase preflight: [preflight] Some fatal errors occurred:
#       [ERROR NumCPU]: the number of available CPUs 1 is less than the required 2
# 增加參數 --ignore-preflight-errors=NumCPU


## 順利的話，執行過程約 2-3 分鐘，最後會出現以下訊息，
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 192.168.2.16:6443 --token s8o9wi.dylbvs735sy53mmq --discovery-token-ca-cert-hash sha256:0c16a05978533ca8f44af6e779162a1c99516fa2a4acd81915f0379755a856bc
```


執行最後描述的這段：
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


查看 docker ps，會出現一堆 container 已經在跑：
```
~# docker ps  
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS               NAMES  
ab574cbeb1f2        2ed65dca1a98           "/usr/local/bin/kube…"   33 seconds ago      Up 33 seconds                           k8s_kube-proxy_kube-proxy-7cfkg_kube-system_83851117-48bf-11e9-b533-000c29d7e00b_0  
5bb5811d11d4        k8s.gcr.io/pause:3.1   "/pause"                 34 seconds ago      Up 33 seconds                           k8s_POD_kube-proxy-7cfkg_kube-system_83851117-48bf-11e9-b533-000c29d7e00b_0  
acc7433382dc        b8df3b177be2           "etcd --advertise-cl…"   56 seconds ago      Up 55 seconds                           k8s_etcd_etcd-k8s-master01-u1604_kube-system_f09a86c0e59bd660bdd359cf6d46e2be_0  
bf812ade4168        14028d7dcbf9           "kube-scheduler --ad…"   56 seconds ago      Up 55 seconds                           k8s_kube-scheduler_kube-scheduler-k8s-master01-u1604_kube-system_cbb979db2eb698a42e58c4ca7edd7b16_0  
3951b23da250        abbc2fa179b7           "kube-controller-man…"   56 seconds ago      Up 55 seconds                           k8s_kube-controller-manager_kube-controller-manager-k8s-master01-u1604_kube-system_fc391fbab6130026480db4a97e595c16_0  
9c146a4aae4b        6de771eabf8c           "kube-apiserver --au…"   56 seconds ago      Up 55 seconds                           k8s_kube-apiserver_kube-apiserver-k8s-master01-u1604_kube-system_f09f833b5c32ac560364b59f58055df6_0
```



同樣的，查看 docker images，應該已經抓了一堆東西。
```
~# docker images  
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE  
k8s.gcr.io/kube-proxy-amd64                v1.11.8             2ed65dca1a98        2 weeks ago         98.1MB  
k8s.gcr.io/kube-apiserver-amd64            v1.11.8             6de771eabf8c        2 weeks ago         187MB  
k8s.gcr.io/kube-controller-manager-amd64   v1.11.8             abbc2fa179b7        2 weeks ago         155MB  
k8s.gcr.io/kube-scheduler-amd64            v1.11.8             14028d7dcbf9        2 weeks ago         56.9MB  
k8s.gcr.io/coredns                         1.1.3               b3b94275d97c        9 months ago        45.6MB  
k8s.gcr.io/etcd-amd64                      3.2.18              b8df3b177be2        11 months ago       219MB  
k8s.gcr.io/pause                           3.1                 da86e6ba6ca1        15 months ago       742kB
```


要注意 Node Disk 的容量


### 2.1.2 配置 KUBECONFIG

為了在工作環境可以直接對 Master Node 下指令，執行以下步驟：
1. 從 Master Node 裡複製 `$HOME/.kube/config` 內容到工作站 (Ex: Macbook)
2. 把內容存在 `$HOME/.kube/config` 裡，或者另外取名字，例如 `$HOME/.kube/config_cluster1.yaml`
3. 初始化 Config: `export KUBECONFIG=$HOME/.kube/config_cluster1.yaml`

可以把初始化寫入 `$HOME/bashrc` 裡，下次開啟 terminal 時就可以使用。


### 2.1.3 確認 Master Node 狀態

取得 node 狀態，`k8s-master01-u1604` 還沒有 ready。可以從 describe 中看到這段關鍵訊息：`network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized`

```
~# kubectl get nodes  
NAME                 STATUS     ROLES     AGE       VERSION  
k8s-master01-u1604   NotReady   master    2m        v1.11.3  
  
~# kubectl describe node k8s-master01-u1604  
... 略 ...  
  
Conditions:  
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message  
  ... 略 ...  
  Ready            False   Sun, 17 Mar 2019 22:20:59 +0800   Sun, 17 Mar 2019 22:18:02 +0800   KubeletNotReady              runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized  
  
... 略 ...
```


取得 kube-system pods 狀態，可以看到 `coredns` 還沒 ready
```
~# kubectl get pods -n kube-system  
NAME                                         READY     STATUS    RESTARTS   AGE  
coredns-78fcdf6894-j88px                     0/1       Pending   0          4m  
coredns-78fcdf6894-lvlf7                     0/1       Pending   0          4m  
etcd-k8s-master01-u1604                      1/1       Running   0          3m  
kube-apiserver-k8s-master01-u1604            1/1       Running   0          3m  
kube-controller-manager-k8s-master01-u1604   1/1       Running   0          3m  
kube-proxy-7cfkg                             1/1       Running   0          4m  
kube-scheduler-k8s-master01-u1604            1/1       Running   0          3m
```


## 2.2 部署 CNI `Container Network Interface`

[CNI](https://github.com/containernetworking/cni) 全名是 `Container Network Interface`，是 K8s 針對容器網路介面的規範定義。

底下整理三個常見的 CNI 實作安裝筆記：
1. Flannel
2. Calico
3. Weave Net

其他常見的還有：
1. AWS VPC CNI


### 2.2.1 Flannel

[Flannel](https://github.com/coreos/flannel) 是 CoreOS 實作的 CNI，主要是 Layer 3 / IPv4 的 overlay network. Flannel 包含了三種不同的封包封裝與路由模式：
1. UDP (Data Switch Beteeen User and System Spaces)
2. VxLAN (預設)
3. host-gw

```
## Flannel  
# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml  
# kubectl delete -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml  
  
## Before apply  
~$ k get po -o wide  
NAME                                   READY   STATUS    RESTARTS   AGE     IP             NODE           NOMINATED NODE   READINESS GATES  
coredns-6dcc67dcbc-4h2d5               0/1     Pending   0          9m26s   <none>         <none>         <none>           <none>  
coredns-6dcc67dcbc-h6x9m               0/1     Pending   0          9m26s   <none>         <none>         <none>           <none>  
etcd-k8s-v114-m01                      1/1     Running   0          8m28s   192.168.2.16   k8s-v114-m01   <none>           <none>  
kube-apiserver-k8s-v114-m01            1/1     Running   0          8m39s   192.168.2.16   k8s-v114-m01   <none>           <none>  
kube-controller-manager-k8s-v114-m01   1/1     Running   0          8m38s   192.168.2.16   k8s-v114-m01   <none>           <none>  
kube-proxy-dk8gn                       1/1     Running   0          5m2s    192.168.2.17   k8s-v114-w02   <none>           <none>  
kube-proxy-gtfkz                       1/1     Running   1          8m45s   192.168.2.20   k8s-v114-w01   <none>           <none>  
kube-proxy-jbmnx                       1/1     Running   0          9m26s   192.168.2.16   k8s-v114-m01   <none>           <none>  
kube-scheduler-k8s-v114-m01            1/1     Running   0          8m20s   192.168.2.16   k8s-v114-m01   <none>           <none>  
  
## Apply  
~$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml  
  
podsecuritypolicy.policy/psp.flannel.unprivileged created  
clusterrole.rbac.authorization.k8s.io/flannel created  
clusterrolebinding.rbac.authorization.k8s.io/flannel created  
serviceaccount/flannel created  
configmap/kube-flannel-cfg created  
daemonset.apps/kube-flannel-ds-amd64 created  
daemonset.apps/kube-flannel-ds-arm64 created  
daemonset.apps/kube-flannel-ds-arm created  
daemonset.apps/kube-flannel-ds-ppc64le created  
daemonset.apps/kube-flannel-ds-s390x created  
  
## After  
## coredns 狀態變成 running, 新增了 kube-flannel-ds  
~$ k get po -o wide  
NAME                                          READY   STATUS    RESTARTS   AGE     IP             NODE                  NOMINATED NODE   READINESS GATES  
coredns-6dcc67dcbc-dfblm                      1/1     Running   0          4m21s   10.244.0.3     k8s-v114-m1-flannel   <none>           <none>  
coredns-6dcc67dcbc-tp9xk                      1/1     Running   0          4m21s   10.244.0.4     k8s-v114-m1-flannel   <none>           <none>  
etcd-k8s-v114-m1-flannel                      1/1     Running   0          3m20s   192.168.2.16   k8s-v114-m1-flannel   <none>           <none>  
kube-apiserver-k8s-v114-m1-flannel            1/1     Running   0          3m35s   192.168.2.16   k8s-v114-m1-flannel   <none>           <none>  
kube-controller-manager-k8s-v114-m1-flannel   1/1     Running   0          3m31s   192.168.2.16   k8s-v114-m1-flannel   <none>           <none>  
kube-flannel-ds-amd64-6d85d                   1/1     Running   0          54s     192.168.2.16   k8s-v114-m1-flannel   <none>           <none>  
kube-proxy-th42c                              1/1     Running   0          4m21s   192.168.2.16   k8s-v114-m1-flannel   <none>           <none>  
kube-scheduler-k8s-v114-m1-flannel            1/1     Running   0          3m19s   192.168.2.16   k8s-v114-m1-flannel   <none>           <none>
```


### 2.2.2 Calico

部署 Calico 要注意預設的 `POD_CIDR` 是 `192.168.0.0/16`，如果 Local LAN 的 CIDR 是一樣的，請先更改 yaml 再 apply，如下：


```
# calico  
## ref: https://docs.projectcalico.org/v3.8/getting-started/kubernetes/installation/calico  
curl https://docs.projectcalico.org/v3.8/manifests/calico.yaml -O  
POD_CIDR="<your-pod-cidr>" \  
sed -i -e "s?192.168.0.0/16?$POD_CIDR?g" calico.yaml  
kubectl apply -f calico.yaml
```


順利後，會看到 coredns 狀態會變成 running，如下：
```
~# kubectl get po -n kube-system  
NAME                                       READY   STATUS    RESTARTS   AGE  
calico-kube-controllers-564b6667d7-jw6vr   1/1     Running   0          4m27s  
calico-node-fjqk6                          1/1     Running   0          4m27s  
coredns-5644d7b6d9-bffcv                   1/1     Running   0          12m  
coredns-5644d7b6d9-dbmsj                   1/1     Running   0          12m  
etcd-u1604-k8s-m1                          1/1     Running   0          11m  
kube-apiserver-u1604-k8s-m1                1/1     Running   0          11m  
kube-controller-manager-u1604-k8s-m1       1/1     Running   0          11m  
kube-proxy-7vbbg                           1/1     Running   0          12m  
kube-scheduler-u1604-k8s-m1                1/1     Running   0          11m  
  
## 檢查 Master Node 狀態，變成了 Ready  
~# kubectl get nodes  
NAME           STATUS   ROLES    AGE   VERSION  
u1604-k8s-m1   Ready    master   13m   v1.16.0
```


### 2.2.3 Weave Net

Weave Net 也是很常見的 CNI，底下是基本的安裝步驟：

```
# 部署網路插件  
kubectl apply -f https://git.io/weave-kube-1.6  
  
# 再次檢查狀態：weave 部署中  
~# kubectl get pods -n kube-system  
NAME                                         READY     STATUS              RESTARTS   AGE  
coredns-78fcdf6894-j88px                     0/1       Pending             0          8m  
coredns-78fcdf6894-lvlf7                     0/1       Pending             0          8m  
etcd-k8s-master01-u1604                      1/1       Running             0          7m  
kube-apiserver-k8s-master01-u1604            1/1       Running             0          7m  
kube-controller-manager-k8s-master01-u1604   1/1       Running             0          8m  
kube-proxy-7cfkg                             1/1       Running             0          8m  
kube-scheduler-k8s-master01-u1604            1/1       Running             0          7m  
weave-net-4gbxq                              0/2       ContainerCreating   0          19s  
  
## 已經完成部署 weave  
~# kubectl get pods -n kube-system  
NAME                                         READY     STATUS    RESTARTS   AGE  
coredns-78fcdf6894-j88px                     1/1       Running   0          9m  
coredns-78fcdf6894-lvlf7                     1/1       Running   0          9m  
etcd-k8s-master01-u1604                      1/1       Running   0          8m  
kube-apiserver-k8s-master01-u1604            1/1       Running   0          8m  
kube-controller-manager-k8s-master01-u1604   1/1       Running   0          8m  
kube-proxy-7cfkg                             1/1       Running   0          9m  
kube-scheduler-k8s-master01-u1604            1/1       Running   0          8m  
weave-net-4gbxq                              2/2       Running   0          39s  
  
## 再次取得 node 狀態  
~# kubectl get nodes  
NAME                 STATUS    ROLES     AGE       VERSION  
k8s-master01-u1604   Ready     master    10m       v1.11.3
```

> - CNI 標準參閱 [Container Network Interface Specification](https://github.com/containernetworking/cni/blob/master/SPEC.md#container-networking-interface-specification)
> - 刪除 weave: `kubectl delete -f https://git.io/weave-kube-1.6`


## 2.3 新增 Worker Node

確認已經安裝 kubeadm, kubectl, kubelet，改機器名字：
1. `/etc/hostname`: k8s-worker01-u1604
2. `/etc/hosts`: k8s-worker01-u1604
3. `reboot`

完成開機後，把這台機器加入 Kubernetes Cluster:
```
~# kubeadm join 192.168.2.16:6443 \  
  --token s8o9wi.dylbvs735sy53mmq \  
  --discovery-token-ca-cert-hash sha256:0c16a05978533ca8f44af6e779162a1c99516fa2a4acd81915f0379755a856bc
```

    如果上述 token 過期了，到 master 執行這段產生： kubeadm token create --print-join-command


到 k8s-master01-u1604 這台機器上，下 kubectl get nodes，就會看到 worker node 已經加入 cluster.
```
~# kubectl get nodes  
NAME                 STATUS    ROLES     AGE       VERSION  
k8s-master01-u1604   Ready     master    19m       v1.11.3  
k8s-worker01-u1604   Ready     <none>    2m        v1.11.3
```

    同樣的，Join 之後可以連到 Worker Node 透過 docker ps 看看實際的運行狀況.















