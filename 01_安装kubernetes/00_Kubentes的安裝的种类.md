

K8s Cluster 安裝分幾種選擇：

1. `全自動`: Master / Worker Nodes 安裝都不用管，連升級 K8s 版本都不用管，像 GCP 的 GKE
2. `半自動`: Cluster 的建置與管理是半自動，需要自己處理 K8s 升級。類似產品如下：
    - [kops](https://github.com/kubernetes/kops) : Master / Worker Nodes 都自己裝，除了這些，也包含網路規劃、權限等
    - [EKS](https://rickhw.github.io/2019/03/17/Container/Install-K8s-with-Kubeadm/): Master Node 由 AWS 管理，使用者管理 Worker Nodes，詳細筆記參閱：[K8s 安裝筆記 - AWS EKS](https://rickhw.github.io/2019/03/17/Container/Install-K8s-with-Kubeadm/)
    - [Experience EKS Anywhere](https://rickhw.github.io/2021/09/23/AWS/Experience-EKS-Anywhere/): AWS OpenSource 的 EKS 版本，可以安裝在自己的環境。
3. `半手動`: 從 VM / Machine 開始就要自己來，也就是本文，主要是使用 K8s 官方工具 `kubeadm`。
4. `全手動`: 全都自己來，每個 k8s 的角色都自己安裝，從 kube-apiserver、etcd、kube-proxy … 等。






