https://rickhw.github.io/2017/07/15/Container/Experience-minikube/
https://zhuanlan.zhihu.com/p/112755080

Minikube是一个可以本地运行的单机版kubernetes,方便我们学习kubernetes和调试程序。

Minikube是由Kubernetes社区维护的单机版的Kubernetes集群，支持macOS, Linux, and Windows等多种操作系统平台，使用最新的官方stable版本，并支持Kubernetes的大部分功能，

# 1 安装 


## 1.1 安裝 kubectl

安裝 kubectl for macOS: https://kubernetes.io/docs/tasks/tools/install-kubectl/ 

```
~  curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl

~  chmod +x ./kubectl
~  sudo mv ./kubectl /usr/local/bin/kubectl
```


## 1.2 安装virtualbox

安裝 VirtualBox: [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)


## 1.3 安装minikube

下载Minikube：[https://storage.googleapis.com/minikube/releases/latest/minikube-installer.exe](https://storage.googleapis.com/minikube/releases/latest/minikube-installer.exe)
默认安装路径：`C:\Program Files\Kubernetes\Minikube`



# 2 相关操作 

安装 minikube

minikube start
执行 minikube dashboard,  会弹出界面 
minkube tunnel  : make pod  von aussen erreichbar 

skafold run ( build image  )
skaffold dev  ( build image + 一些其他的 )


## 2.1 取得版本咨询

```
~  minikube version
minikube version: v1.4.0
commit: 7969c25a98a018b94ea87d949350f3271e9d64b6
```

## 2.2 启动 minikube
`minikube start --image-mirror-country='cn' --container-runtime=containerd`
- `**--image-mirror-country='cn'**`

设置使用国内阿里云镜像源
- `**--container-runtime=containerd**`

Minikube默认安装kubernetes v1.25.0，需要将容器运行时设置为containerd
运行v1.24 .0及之后版本，都需要此设置。如果不设置会出现错误。

如果要运行其他kubernetes版本，使用以下命令
- `minikube start --image-mirror-country='cn' --kubernetes-version=v1.23.0`

---

```
~  minikube start --vm-driver=virtualbox
😄  minikube v1.4.0 on Darwin 10.14.2
💿  Downloading VM boot image ...
    > minikube-v1.4.0.iso.sha256: 65 B / 65 B [--------------] 100.00% ? p/s 0s
    > minikube-v1.4.0.iso: 135.73 MiB / 135.73 MiB [-] 100.00% 6.03 MiB p/s 23s
🔥  Creating virtualbox VM (CPUs=2, Memory=2000MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.16.0 on Docker 18.09.9 ...
💾  Downloading kubelet v1.16.0
💾  Downloading kubeadm v1.16.0
🚜  Pulling images ...
🚀  Launching Kubernetes ...
⌛  Waiting for: apiserver proxy etcd scheduler controller dns
🏄  Done! kubectl is now configured to use "minikube"
```

> - 指定 VM driver: `minikube start --vm-driver=virtualbox`
> - 如果無法正常啟動，先清除: `minikube delete`
> - 設定記憶體與 cpu: `minikube start --memory=16384 --cpus=4 --kubernetes-version=v1.14.2`
> - 指定 vm-driver:
>     - brew install docker-machine-driver-vmware
>     - minikube start –driver=vmware –memory=16384 –cpus=4
>     - minikube config set driver vmware

## 2.3 停止、启动、删除minikube
minikube stop
minikube start
minikube delete --all


## 2.4 minikube的一些其他的命令 

然后查看状态 minikube status

```
~   minikube status  
host: Running  
kubelet: Running  
apiserver: Running  
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100  
  
~  kubectl cluster-info  
Kubernetes master is running at https://192.168.99.100:8443  
KubeDNS is running at https://192.168.99.100:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy  
  
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.  
  
~  kubectl version  
Client Version: version.Info{Major:"1", Minor:"14+", GitVersion:"v1.14.6-eks-5047ed", GitCommit:"5047edce664593832e9b889e447ac75ab104f527", GitTreeState:"clean", BuildDate:"2019-08-22T06:44:59Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"darwin/amd64"}  
Server Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.0", GitCommit:"2bd9643cee5b3b3a5ecbd3af49d09018f0773c77", GitTreeState:"clean", BuildDate:"2019-09-18T14:27:17Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"linux/amd64"}  
  
  
~  kubectl  get no -o wide  
NAME       STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE              KERNEL-VERSION   CONTAINER-RUNTIME  
minikube   Ready    master   31m   v1.16.0   10.0.2.15     <none>        Buildroot 2018.05.3   4.15.0           docker://18.9.9  
  
  
~  kubectl get po --all-namespaces  
NAMESPACE              NAME                                         READY   STATUS    RESTARTS   AGE  
kube-system            coredns-5644d7b6d9-m25bv                     1/1     Running   1          30m  
kube-system            coredns-5644d7b6d9-s7jmt                     1/1     Running   1          30m  
kube-system            etcd-minikube                                1/1     Running   1          28m  
kube-system            kube-addon-manager-minikube                  1/1     Running   1          28m  
kube-system            kube-apiserver-minikube                      1/1     Running   1          29m  
kube-system            kube-controller-manager-minikube             1/1     Running   1          28m  
kube-system            kube-proxy-nms9l                             1/1     Running   1          30m  
kube-system            kube-scheduler-minikube                      1/1     Running   1          28m  
kube-system            storage-provisioner                          1/1     Running   2          29m  
kubernetes-dashboard   dashboard-metrics-scraper-76585494d8-wdrdx   1/1     Running   1          29m  
kubernetes-dashboard   kubernetes-dashboard-57f4cb4545-w298w        1/1     Running   2          29m
```


# 3 minikube tunnel 

https://minikube.sigs.k8s.io/docs/commands/tunnel/

Connect to LoadBalancer services

tunnel creates a route to services deployed with type LoadBalancer and sets their Ingress to their ClusterIP. for a detailed example see [https://minikube.sigs.k8s.io/docs/tasks/loadbalancer](https://minikube.sigs.k8s.io/docs/tasks/loadbalancer)


The network is limited if using the Docker driver on Darwin, Windows, or WSL, and the Node IP is not reachable directly.
Running minikube on Linux with the Docker driver will result in no tunnel being created.
Services of type `NodePort` can be exposed via the `minikube service <service-name> --url` command. It must be run in a separate terminal window to keep the [tunnel](https://en.wikipedia.org/wiki/Port_forwarding#Local_port_forwarding) open. Ctrl-C in the terminal can be used to terminate the process at which time the network routes will be cleaned up.


## 3.1 安装Dashboard


`minikube dashboard --url --port=63373`

不设置`--port`端口是随机的，每次启动可能会变化。

![](https://cdn.nlark.com/yuque/0/2022/png/28915315/1663749917089-fb670ab6-85a3-4180-9190-de757ad7596f.png)

在浏览器访问（命令行退出后无法访问）：

`[http://127.0.0.1:63373/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/](http://127.0.0.1:63373/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/)`

![](https://cdn.nlark.com/yuque/0/2022/png/28915315/1663750453195-0ea2d075-755b-4fbe-925b-5fe53869741b.png)

可以看到，启动了2个dashboard容器。

![](https://cdn.nlark.com/yuque/0/2022/png/28915315/1663750997376-411cbcc2-46a9-440a-b690-3cf4aa1b8780.png)

## 3.2 其他

kubectl get nodes -o wide ##可以查看到我们有了一个单节点的集群，IP地址
kubectl get pods -o wide ##可以看到我在这里创建了两个NGINX pods和IP地址
ssh docker@192.168.99.101 ##现在我们ssh到我们的master节点，默认用户名：docker 密码：tcuser
curl 172.17.0.7    ##在集群内部尝试访问NGINX成功输出界面


# 4 Run “Hello” App

啟動 echoserver

```
## 建立 pod  
~  kubectl run hello-minikube \  
	--image=gcr.io/google_containers/echoserver:1.4 \  
	--port=8080  
deployment "hello-minikube" created  
  
  
~  kubectl get pod  
NAME                             READY     STATUS    RESTARTS   AGE  
hello-minikube-938614450-50h7g   1/1       Running   0          3m  
 11:15:05  gtcafe-dev.rickhwang-dev  minikube    
  
## Expose as NodePort  
~  kubectl expose deployment \  
	hello-minikube --type=NodePort  
service "hello-minikube" exposed  
  
  
~  curl $(minikube service hello-minikube --url)  
CLIENT VALUES:  
client_address=172.17.0.1  
command=GET  
real path=/  
query=nil  
request_version=1.1  
request_uri=http://192.168.99.100:8080/  
  
SERVER VALUES:  
server_version=nginx: 1.10.0 - lua: 10001  
  
HEADERS RECEIVED:  
accept=*/*  
host=192.168.99.100:30459  
user-agent=curl/7.51.0  
BODY:  
-no body in request-
```



# 5 Uninstall minikube or upgrade K8s version

如果想要升級 k8s 版本、或者升級 minikube，最簡單的方法就是直接重新安裝。底下是在 macOS 從重新安裝 minikube 開始：

```
# 刪掉 k8s
~$ minikube delete

🔥  Deleting "minikube" in vmware ...
💀  Removed all traces of the "minikube" cluster.

# unlink existing minikube
~$ rm -rf /usr/local/bin/minikube

# 升級 brew
brew update

# 重新安裝 minikube
brew reinstall minikube
```












