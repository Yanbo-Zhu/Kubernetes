

https://zhuanlan.zhihu.com/p/112755080

Minikube是一个可以本地运行的单机版kubernetes,方便我们学习kubernetes和调试程序。

Minikube是由Kubernetes社区维护的单机版的Kubernetes集群，支持macOS, Linux, and Windows等多种操作系统平台，使用最新的官方stable版本，并支持Kubernetes的大部分功能，


# 1 安装 

下载Minikube：[https://storage.googleapis.com/minikube/releases/latest/minikube-installer.exe](https://storage.googleapis.com/minikube/releases/latest/minikube-installer.exe)
默认安装路径：`C:\Program Files\Kubernetes\Minikube`



# 2 相关操作 

## 2.1 启动 minikube
`minikube start --image-mirror-country='cn' --container-runtime=containerd`
- `**--image-mirror-country='cn'**`

设置使用国内阿里云镜像源
- `**--container-runtime=containerd**`

Minikube默认安装kubernetes v1.25.0，需要将容器运行时设置为containerd
运行v1.24 .0及之后版本，都需要此设置。如果不设置会出现错误。


如果要运行其他kubernetes版本，使用以下命令
- `minikube start --image-mirror-country='cn' --kubernetes-version=v1.23.0`
## 2.2 停止、启动、删除minikube
minikube stop
minikube start
minikube delete --all


## 2.3 

然后查看状态 minikube status

## 2.4 安装Dashboard


`minikube dashboard --url --port=63373`

不设置`--port`端口是随机的，每次启动可能会变化。

![](https://cdn.nlark.com/yuque/0/2022/png/28915315/1663749917089-fb670ab6-85a3-4180-9190-de757ad7596f.png)

在浏览器访问（命令行退出后无法访问）：

`[http://127.0.0.1:63373/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/](http://127.0.0.1:63373/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/)`

![](https://cdn.nlark.com/yuque/0/2022/png/28915315/1663750453195-0ea2d075-755b-4fbe-925b-5fe53869741b.png)

可以看到，启动了2个dashboard容器。

![](https://cdn.nlark.com/yuque/0/2022/png/28915315/1663750997376-411cbcc2-46a9-440a-b690-3cf4aa1b8780.png)

## 2.5 其他

kubectl get nodes -o wide ##可以查看到我们有了一个单节点的集群，IP地址
kubectl get pods -o wide ##可以看到我在这里创建了两个NGINX pods和IP地址
ssh docker@192.168.99.101 ##现在我们ssh到我们的master节点，默认用户名：docker 密码：tcuser
curl 172.17.0.7    ##在集群内部尝试访问NGINX成功输出界面
