
# 1 题目要求

更新在namespace **ckad00018** 中的 Pod **ckad00018-newpod**
使其使用一个只允许此 Pod 与 Pod **front** 和 **db** 之间收发流量的 Networkpolicy 。


# 2 参考 
[https://kubernetes.io/zh-cn/docs/concepts/services-networking/network-policies/](https://kubernetes.io/zh-cn/docs/concepts/services-networking/network-policies/)

![](image/image-87.png)


# 3 解答 

1 查看pod 和networkpolicy
kubectl -n ckad00018 get pod --show-labels
![](image/image-90.png)

kubectl -n ckad00018 get networkpolicies --show-labels


2 查看networkpolicy 的 podselector
kubectl -n ckad00018 get networkpolicies access-db -oyaml | grep -i -A3 podselector

![](image/image-89.png)


kubectl -n ckad00018 get networkpolicies access-front -oyaml | grep -i -A3 podselector
-A3: Zeigt die 3 Zeilen nach dem Treffer an (also "3 lines After").

![](image/image-88.png)


通过查看所有networkpolicy 得知，
Pod front和 db 各有一个 networkpolicy
Pad ckad00018-newpod 没networkpolicy


3 所以查看front 和 db 的 networkpolicy 标签 把可以跟他们互通的标签 ，设置 给 ckad00018-newpod
kubectl -n ckad00018 label pod  ckad00018-newpod front-access=true db-access=true
kubectl -n ckad00018 get pod --show-labels  # 如下图
![](image/image-91.png)



