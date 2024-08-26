
# 1 题设


```
Quick Reference
Cluster/配置环境 k8s
Namespace ingress-kk

您必须切换到正确的 Cluster/配置环境。不这样做可能导致零分。
[candidate@node-1] $ kubectl config use-context k8s

Task
在 namespace ingress-kk 下有一个 ingress ，但是它貌似不能被正常访问，
请排除出原因，并修复。

请注意，这道题的 deployment 是正确的，请不要修改 deployment。
```


# 2 参考 

[https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/)

[https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#expose](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#expose)


```
# 参考官方示例
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376

# 或者
kubectl expose -h 

```


# 3 解答 


1 先检查这道题里的 ingress 和 deployment，以及 service，这三项哪里有问题。
kubectl -n ingress-kk get all

[![CKAD考试题解析](https://www.ljh.cool/wp-content/uploads/2023/02/image-96.png)](https://www.ljh.cool/wp-content/uploads/2023/02/image-96.png)

发现没有service , 查看ingress 信息
kubectl -n ingress-kk


经过分析检查，会发现没有service，所以需创建service，而service需要的内容部分来自ingress和deployment里的字段。

2  查看ingress，记下service的name和port的number。ingress中找svc所需的svc名和port
kubectl -n ingress-kk get ingress -o yaml
[![CKAD考试题解析](https://www.ljh.cool/wp-content/uploads/2023/02/image-97.png)](https://www.ljh.cool/wp-content/uploads/2023/02/image-97.png)

3 查看deployment，记下ports的containerPort, 通过 deployment中找 标签,targetport

kubectl -n ingress-kk get deployments -o yaml

[![CKAD考试题解析](https://www.ljh.cool/wp-content/uploads/2023/02/image-98.png)](https://www.ljh.cool/wp-content/uploads/2023/02/image-98.png)

4 创建service
vi kk-svc.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: nginxsvc-kk #这里对应 ingress 里的 service: name:
  namespace: ingress-kk
spec:
  ports:
  - port: 80 #port: 80 是 service 的端口，它对应的是 ingress 里的 port: number
    targetPort: 80 # targetPort: 80 是 deployment 里的容器的端口
    protocol: TCP
  selector:
    name: nginx-lab #这里对应 deployment 的 labels 标签
```

kubectl apply -f kk-svc.yaml


或者 
参考： $ `kubectl expose (-f FILENAME | TYPE NAME) [--port=port] [--protocol=TCP|UDP|SCTP] [--target-port=number-or-name] [--name=name] [--external-ip=external-ip-of-service] [--type=type]  `
kubectl expose deployment nginxdep --port=80 --protocol=TCP --target-port=80 --name=nginxsvc-kk --selector name=nginx-lab



5 检查
kubectl -n ingress-kk get all

curl <service 的 ip 地址>

[![CKAD考试题解析](https://www.ljh.cool/wp-content/uploads/2023/02/image-99.png)](https://www.ljh.cool/wp-content/uploads/2023/02/image-99.png)