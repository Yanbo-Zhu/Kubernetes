

# 1 题设 

```
Quick Reference
Cluster/配置环境 k8s
Namespace ckad00017

Contest
您需要扩展一个现有的应用程序，并将其公开在基础设施内。

您必须切换到正确的 Cluster/配置环境。不这样做可能导致零分。
[candidate@node-1] $ kubectl config use-context k8s

Task
1 首先，更新在 namespace ckad00017 中的 Deployment ckad00017-deployment ：
⚫ 以使其运行 5 个 Pod 的副本
⚫ 将以下标签添加到 Pod
tier: dmz

2 然后，在 namespace ckad00017 中创建一个名为 rover 的 NodePort Service 以在 TCP 端口 81 上公开 Deployment ckad00017-deployment
```


[https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#expose](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#expose)



# 2 解题 

1 扩容副本数到5
kubectl -n **ckad00017 scale deployment ckad00017-deployment --replicas=5

2 添加标签
kubectl -n ckad00017 get deployments.apps -o yaml > 17.yaml
kubectl delete -f 17.yaml
vim 17.yaml
```
...
spec:
	selector:
	  matchLabels
	    app:ckad00017-deployment
	    tier:dmz  # 题目需求
template:
	metadata:
		labels:
		  app:ckad00017-deployment
		  tier:dmz # 题目需求
```


1+2 可以连起来一起做
kubectl get deployment ckad00017-deployment -n ckad00017 -o yaml > ckad00017.yaml
cp ckad00017.yaml bak-ckad00017.yaml
kubectl delete -f ckad00017.yaml

vi ckad00017.yaml
修改 replicas: 1 为 replicas: 5；标签 app: ckad00017-deployment

kubectl apply -f ckad00017.yaml
[![CKAD考试题解析](https://www.ljh.cool/wp-content/uploads/2023/02/image-85.png)](https://www.ljh.cool/wp-content/uploads/2023/02/image-85.png)

检查
kubectl get pod -n ckad00017 --show-labels


3 expose 暴露端口, port为service的端口, target-port为容器端口, 默认为80,以检查为准
kubectl -n ckad00017 expose deployment ckad00017-deployment --name rover --protocol TCP --port 81 --target-port 81 --type NodePort 

```
注意：
--port=81 是暴露的 service 的端口为 81。
--target-port=81 是 Deployment ckad00017-deployment 内 Pod 容器使用的端口，这个需要检查 yaml 文件。如果检查完，发现容器端口为 containerPort: 80，
或者没有 containerPort，则也是用默认的 80。则暴露服务的命令需要写成。
kubectl -n ckad00017 expose deployment ckad00017-deployment --name=rover --protocol=TCP --port=81 --target-port=80 --type=NodePort
```

4 测试验证
kubectl -n ckad00017 get svc


curl 10.103.95.21:81
[![CKAD考试题解析](https://www.ljh.cool/wp-content/uploads/2023/02/image-86.png)](https://www.ljh.cool/wp-content/uploads/2023/02/image-86.png)






