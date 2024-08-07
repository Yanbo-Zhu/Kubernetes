
https://medium.com/@muppedaanvesh/a-hands-on-guide-to-kubernetes-endpoints-endpointslices-%EF%B8%8F-1375dfc9075c

# 1 Endpoint 

Endpoint是可被访问的服务端点，即一个状态为running的pod，它是service访问的落点，只有service关联的pod才可能成为endpoint。

endpoint是k8s集群中的一个资源对象，存储在etcd中，用来记录一个service对应的所有pod的访问地址。service配置selector，endpoint controller才会自动创建对应的endpoint对象；否则，不会生成endpoint对象.




例如，k8s集群中创建一个名为hello的service，就会生成一个同名的endpoint对象，ENDPOINTS就是service关联的pod的ip地址和端口。

一个 Service 由一组 backend Pod 组成。这些 Pod 通过 endpoints 暴露出来。 Service Selector 将持续评估，结果被 POST 到一个名称为 Service-hello 的 Endpoint 对象上。 当 Pod 终止后，它会自动从 Endpoint 中移除，新的能够匹配上 Service Selector 的 Pod 将自动地被添加到 Endpoint 中。 检查该 Endpoint，注意到 IP 地址与创建的 Pod 是相同的。现在，能够从集群中任意节点上使用 curl 命令请求 hello Service : 。 注意 Service IP 完全是虚拟的，它从来没有走过网络，如果对它如何工作的原理感到好奇，可以阅读更多关于 服务代理 的内容。

Endpoints是实现实际服务的端点集合。

Kubernetes在创建Service时，根据Service的标签选择器（Label Selector）来查找Pod，据此创建与Service同名的EndPoints对象。当Pod的地址发生变化时，EndPoints也随之变化。Service接收到请求时，就能通过EndPoints找到请求转发的目标地址。

Service不仅可以代理Pod，还可以代理任意其他后端，比如运行在Kubernetes外部Mysql、Oracle等。这是通过定义两个同名的service和endPoints来实现的。

在实际的生产环境使用中，通过分布式存储来实现的磁盘在mysql这种IO密集性应用中，性能问题会显得非常突出。所以在实际应用中，一般不会把mysql这种应用直接放入kubernetes中管理，而是使用专用的服务器来独立部署。而像web这种无状态应用依然会运行在kubernetes当中，这个时候web服务器要连接kubernetes管理之外的数据库，有两种方式：一是直接连接数据库所在物理服务器IP，另一种方式就是借助kubernetes的Endpoints直接将外部服务器映射为kubernetes内部的一个服务。

简单认为：动态存储pod名字与pod ip对应关系的list，并提供将请求转发到实际pod上的能力


Endpoints
Endpoints表示一个Service对应的所有Pod副本的访问地址。
Node上的Kube-proxy进程获取每个Service的Endpoints，实现Service的负载均衡功能。

![](image/e80693961bacc62355b73be3a5d27b17.png)

---


- 为什么我们需要Service 关联endpoint 因为 我们需要 endpoint 这个 resource中定义的 ipadrrese
    - Service reosurce只定义了那些port 被暴露了出来， 却无法定义那些 ip addresse 中的 这个port 要被暴露出来 . 这时候就需要 endpoint 去 define a list of IP addresses and ports.

Endpoint是kubernetes中的一个资源对象，存储在etcd中，用来记录一个service对应的所有pod的访问地址，它是根据service配置文件中selector描述产生的。
一个Service由一组Pod组成，这些Pod通过Endpoints暴露出来，**Endpoints是实现实际服务的端点集合**。换句话说，service和pod之间的联系是通过endpoints实现的。

The Role of Endpoints in Service Discovery: 
Service discovery in Kubernetes relies heavily on Endpoints. When a service receives a request, it uses the information in the associated Endpoint object to route the request to one of the available pods. This mechanism ensures that traffic is evenly distributed among healthy pods.


https://medium.com/@muppedaanvesh/a-hands-on-guide-to-kubernetes-endpoints-endpointslices-%EF%B8%8F-1375dfc9075c

Kubernetes Endpoints are API objects that define a list of IP addresses and ports. These addresses correspond to the pods that are dynamically assigned to a service. Essentially, an Endpoint in Kubernetes is a bridge connecting a service to the pods that fulfill the service’s requests.

When a service is created, Kubernetes automatically creates an associated Endpoint object. The Endpoint object maintains the IP addresses and port numbers of the pods that match the service’s selector criteria.

To understand how Kubernetes Endpoints work, let’s break down the process:
1. **Service Creation:** When you create a service in Kubernetes, you define a selector that matches a set of pods. This selector determines which pods the service will route traffic to.
2. **Endpoint Creation**: Kubernetes `automatically` creates an Endpoint object associated with the service. This object contains the IP addresses and ports of the pods that match the selector.
3. **Updating Endpoints**: As pods are added or removed, or their statuses change, Kubernetes `continuously updates` the Endpoint object to reflect the current set of matching pods. This ensures that the service always routes traffic to the appropriate pods.

![](https://miro.medium.com/v2/resize:fit:700/1*kAMuf7ryG81KEFWzQFwVDw.gif)


![image-20200509191917069](https://gitee.com/yooome/golang/raw/main/22-k8s%E8%AF%A6%E7%BB%86%E6%95%99%E7%A8%8B-%E8%B0%83%E6%95%B4%E7%89%88/Kubenetes.assets/image-20200509191917069.png)


![](image/9.webp)


## 1.1 例子

示例：查询端点信息
```
# endpoint 的别名是 ep
kubectl get endpoint

```

示例：删除一个 Pod ，Deployment 会将 Pod 重建，Endpoints 的内容随着 Pod 发生变化。

```
kubectl delete pod xxx

```


示例: 创建自定义的 EndPoint
vi k8s-ep.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: cluster-svc-no-selector
  namespace: default
spec:
  ports:
    - name: http
      port: 80
      targetPort: 80 # 此时的 targetPort 需要和 Endpoints 的 subsets.addresses.ports.port 相同  
---
apiVersion: v1
kind: Endpoints
metadata:
  name: cluster-svc-no-selector # 此处的 name 需要和 Service 的 metadata 的 name 相同
  namespace: default
subsets:
- addresses:
  - ip: 220.181.38.251 # 百度
  - ip: 221.122.82.30 # 搜狗 
  - ip: 61.129.7.47 # QQ
  ports:
  - name: http # 此处的 name 需要和 Service 的 spec.ports.name 相同
    port: 80
    protocol: TCP
```

- The `addresses` field lists the IP addresses of the pods that match the service selector.
- The `ports` field lists the ports on which the pods are listening.


kubectl apply -f k8s-ep.yaml


## 1.2 service如何知道使用那些endpoint 
https://stackoverflow.com/questions/52857825/what-is-an-endpoint-in-kubernetes

### 1.2.1 如果有service label selector

如果在定义一个 service 的时候 有使用 label selector
那么 depoly 这个 service 的时候 ， 一个新的 endpoint ressource 就被自动生成了， 里面的内容 就是 被自动匹配的pod 的 ip addresse ，以及在 service resource 中定义的 port 的值 

To understand how Kubernetes Endpoints work, let’s break down the process:
1. **Service Creation:** When you create a service in Kubernetes, you define a selector that matches a set of pods. This selector determines which pods the service will route traffic to.
2. **Endpoint Creation**: Kubernetes `automatically` creates an Endpoint object associated with the service. This object contains the IP addresses and ports of the pods that match the selector.
3. **Updating Endpoints**: As pods are added or removed, or their statuses change, Kubernetes `continuously updates` the Endpoint object to reflect the current set of matching pods. This ensures that the service always routes traffic to the appropriate pods.


```
apiVersion: v1
kind: Service
metadata:
  name: service-clusterip
  namespace: dev
spec:
  selector:      # 注意看这里 用来关联对应的Pod
    app: nginx-pod
  clusterIP: 10.97.97.97 # service的ip地址，如果不写，默认会生成一个
  type: ClusterIP
  ports:
  - name: nginx
  - port: 80  # Service端口       
    targetPort: 80 # pod端口
```


### 1.2.2 create a service without a label selector:


Regarding _external_ Enpoints:

You create a service without a label selector:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service #<------ Should match the name of Endpoints object
spec:
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 9376
```

So the corresponding Endpoint object will not be created automatically and you manually add the `Endpoints` object and map the Service to the desired network address and port where the external resource is running:

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service #<------ Should match the name of Service
subsets:
  - addresses:
      - ip: 192.0.2.45
    ports:
      - port: 9376
```


## 1.3 DNS and Endpoints

Kubernetes services are accessible via DNS. For example, if you have a service named `my-service` in the `default` namespace, it can be resolved with the DNS name `my-service.default.svc.cluster.local`. When this DNS name is resolved, it points to the IP addresses listed in the corresponding Endpoint object.


# 2 Endpoint Silces 

就是一个 endpoint group 

Endpoint Slices are a more scalable and efficient way to manage endpoints in Kubernetes. Introduced in Kubernetes 1.16, Endpoint Slices provide a way to distribute the network endpoints across multiple resources, reducing the load on the Kubernetes API server and improving the performance of large clusters.

An Endpoint Slice represents a subset of the endpoints that make up a service. Instead of having a single large Endpoint object for a service, multiple smaller Endpoint Slice objects are created, each representing a portion of the endpoints.

By default, the control plane creates and manages EndpointSlices to have no more than 100 endpoints each. You can configure this with the `--max-endpoints-per-slice` [kube-controller-manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/) flag, up to a maximum of **1000**.

EndpointSlices can act as the source of truth for [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/) when it comes to how to route internal traffic.



https://blog.csdn.net/qq_33745102/article/details/127966484

EndpointSlice是Endpoint对象的集合。kubernetes会给任何带选择器的Service对象创建EndpointSlice. EndpintSlice对象包含Service 选择器匹配到的所有Pod 的网络地址。EndpointSlice通过<协议，端口，Service名字> 对Endpoint进行分组。

![](image/a5f29abf7fe1ed962a37f42801e36abb.png)



## 2.1 Address types

EndpointSlices support three address types:

- IPv4
- IPv6
- FQDN (Fully Qualified Domain Name)

Each `EndpointSlice` object represents a specific IP address type. If you have a Service that is available via IPv4 and IPv6, there will be at least two `EndpointSlice` objects (one for IPv4, and one for IPv6).

## 2.2 How Endpoint Slices Work

Endpoint Slices work similarly to Endpoints but with added benefits for scalability and performance:

1. **Service Creation:** When a service is created, Kubernetes creates Endpoint Slices instead of a single Endpoint object.
2. **Endpoint Slice Creation**: Each Endpoint Slice contains a subset of the total endpoints for the service. This distribution helps in managing large numbers of endpoints more efficiently.
3. **Updating Endpoint Slices**: Like Endpoints, Endpoint Slices are continuously updated as pods are added, removed, or their statuses change. Kubernetes ensures that the Endpoint Slices reflect the current set of matching pods.

## 2.3 Anatomy of an Endpoint Slice Object

Let’s take a look at a sample Endpoint Slice object:

```
apiVersion: discovery.k8s.io/v1  
kind: EndpointSlice  
metadata:  
  name: my-service-abc123  
addressType: IPv4  
endpoints:  
  - addresses:  
      - 10.0.0.1  
      - 10.0.0.2  
ports:  
  - port: 80
```

In this example:

-  ==The `endpoints` field lists the IP addresses of a subset of pods.==
- The `ports` field lists the ports on which these pods are listening.
- The `addressType` indicates the type of address, such as IPv4 or IPv6.

## 2.4 The Role of Endpoint Slices in Service Discovery

Endpoint Slices improve upon the traditional Endpoints by offering better scalability and performance. They distribute the network endpoints across multiple objects, reducing the load on the Kubernetes API server and improving service discovery in large clusters.

### 2.4.1 DNS and Endpoint Slices

Just like with Endpoints, Kubernetes services using Endpoint Slices are accessible via DNS. The DNS resolution process remains the same, but the underlying management of endpoints is more efficient.


## 2.5 例子 


Let’s go through a hands-on example to see how Endpoints and Endpoint Slices work in practice.

### 2.5.1 Step 1: Create a Deployment

First, create a simple Nginx deployment:

```
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  name: ep-test  
  labels:  
    app: nginx  
spec:  
  replicas: 3  
  selector:  
    matchLabels:  
      app: nginx  
  template:  
    metadata:  
      labels:  
        app: nginx  
    spec:  
      containers:  
      - name: nginx-container  
        image: nginx:latest

```

Save this YAML as `nginx-deployment.yaml` and apply it:

kubectl apply -f nginx-deployment.yaml

### 2.5.2 Step 2: Create a Service

Next, create a service to expose the Nginx deployment:

```
apiVersion: v1  
kind: Service  
metadata:  
  name: nginx-svc  
spec:  
  type: NodePort  
  selector:  
    app: nginx  
  ports:  
    - protocol: TCP  
      port: 80  
      targetPort: 80
```

Save this YAML as `nginx-service.yaml` and apply it:

kubectl apply -f nginx-service.yaml

Output:
```

$ kubectl get po,svc  
NAME                           READY   STATUS    RESTARTS   AGE  
pod/ep-test-5bb85d69d8-8p2v5   1/1     Running   0          63s  
pod/ep-test-5bb85d69d8-cf7c4   1/1     Running   0          62s  
pod/ep-test-5bb85d69d8-vfkl5   1/1     Running   0          62s  

NAME                TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE  
service/nginx-svc   NodePort   10.245.58.203   <none>        80:31907/TCP   63s
  
```

### 2.5.3 Step 3: Verify Endpoints and Endpoint Slices

After creating the service, Kubernetes automatically creates the associated Endpoints and Endpoint Slices.

Check the Endpoints:

kubectl get endpoints nginx-svc -o yaml

You should see output similar to this:

```

apiVersion: v1  
kind: Endpoints  
metadata:  
  annotations:  
    endpoints.kubernetes.io/last-change-trigger-time: "2024-06-09T18:48:32Z"  
  creationTimestamp: "2024-06-09T18:48:30Z"  
  name: nginx-svc  
  namespace: endpoints  
  resourceVersion: "15095762"  
  uid: 8ecd5c61-bf96-41a2-b961-681b13f70b0b  
subsets:  
- addresses:  
  - ip: 10.244.0.227  
    nodeName: pool-t5ss0fagn-rxkuu  
    targetRef:  
      kind: Pod  
      name: ep-test-5bb85d69d8-8p2v5  
      namespace: endpoints  
      uid: 4e8f2873-3abe-4f11-adfc-8a7b579db114  
  - ip: 10.244.0.250  
    nodeName: pool-t5ss0fagn-rxkuu  
    targetRef:  
      kind: Pod  
      name: ep-test-5bb85d69d8-vfkl5  
      namespace: endpoints  
      uid: 30fea27d-bc8f-4dd7-9daa-9e03bb3d82a2  
  - ip: 10.244.0.88  
    nodeName: pool-t5ss0fagn-jeb47  
    targetRef:  
      kind: Pod  
      name: ep-test-5bb85d69d8-cf7c4  
      namespace: endpoints  
      uid: 2ac548ea-4b44-4c51-9c79-f337441439b4  
  ports:  
  - port: 80  
    protocol: TCP
```

### 2.5.4 Step 4: Verify Endpoint Slices

Check the Endpoint Slices:

kubectl get endpointslices -l kubernetes.io/service-name=nginx-svc -o yaml

You should see output similar to this:
```

$ kubectl get endpointslices nginx-svc-t9zx4 -o yaml   
  
addressType: IPv4  
apiVersion: discovery.k8s.io/v1  
endpoints:  
- addresses:  
  - 10.244.0.88  
  conditions:  
    ready: true  
    serving: true  
    terminating: false  
  nodeName: pool-t5ss0fagn-jeb47  
  targetRef:  
    kind: Pod  
    name: ep-test-5bb85d69d8-cf7c4  
    namespace: endpoints  
    uid: 2ac548ea-4b44-4c51-9c79-f337441439b4  
- addresses:  
  - 10.244.0.227  
  conditions:  
    ready: true  
    serving: true  
    terminating: false  
  nodeName: pool-t5ss0fagn-rxkuu  
  targetRef:  
    kind: Pod  
    name: ep-test-5bb85d69d8-8p2v5  
    namespace: endpoints  
    uid: 4e8f2873-3abe-4f11-adfc-8a7b579db114  
- addresses:  
  - 10.244.0.250  
  conditions:  
    ready: true  
    serving: true  
    terminating: false  
  nodeName: pool-t5ss0fagn-rxkuu  
  targetRef:  
    kind: Pod  
    name: ep-test-5bb85d69d8-vfkl5  
    namespace: endpoints  
    uid: 30fea27d-bc8f-4dd7-9daa-9e03bb3d82a2  

kind: EndpointSlice  
metadata:  
  annotations:  
    endpoints.kubernetes.io/last-change-trigger-time: "2024-06-09T18:48:32Z"  
  creationTimestamp: "2024-06-09T18:48:30Z"  
  generateName: nginx-svc-  
  generation: 4  
  labels:  
    endpointslice.kubernetes.io/managed-by: endpointslice-controller.k8s.io  
    kubernetes.io/service-name: nginx-svc  
  name: nginx-svc-t9zx4  
  namespace: endpoints  
  ownerReferences:  
  - apiVersion: v1  
    blockOwnerDeletion: true  
    controller: true  
    kind: Service  
    name: nginx-svc  
    uid: b205cb1b-55bd-4d07-82b3-c60dff0b83f0  
  resourceVersion: "15095763"  
  uid: 12de64ee-6f80-4adc-aaac-bf2aa8156e1c  
ports:  
- name: ""  
  port: 80  
  protocol: TCP
```

In this output, you can see that the Endpoint addresses are distributed across multiple Endpoint Slices.


# 3 Endpoint Slices 的优势 相比于 Endpoints

https://blog.csdn.net/qq_33745102/article/details/127966484

EndpointSlice是Endpoint对象的集合。kubernetes会给任何带选择器的Service对象创建EndpointSlice. EndpintSlice对象包含Service 选择器匹配到的所有Pod 的网络地址。EndpointSlice通过<协议，端口，Service名字> 对Endpoint进行分组。

Endpoint 包含Service所有的Pod IP, 因此当一个Pod发生重启时，Pod IP发生改变，需要对整个Endpoint对象重新计算并存储。
当Pod数量较少的情况下，这个不是大问题。但是当数量很多时，需要的大量的网络IO.

当一个Pod发生重启时，EndpointSlice引入的话，只需要更新发生改变的数组元素就可以了。核心点在于在etcd里存储过程中不把整块数据一块存放（像原来Endpoint那样），而是分成几块分开存储。


## 3.1 例子： 20,000 endpoints, 5,000 nodes

可以发现当副本数量增多，网络IO是相当多的。尤其是一个Pod重启这样非常简单频繁出现的场景竟然要传输10TB的数据，而EndpointSlice方式只需要50MB.

![](image/Pasted%20image%2020240807105415.png)

![](image/Pasted%20image%2020240807105420.png)

![](image/Pasted%20image%2020240807105426.png)




# 4 Common Use Cases for Endpoints and Endpoint Slices

1. **Load Balancing**: Both Endpoints and Endpoint Slices enable Kubernetes services to distribute traffic evenly across multiple pods, ensuring high availability and efficient resource utilization.
2. **Service Discovery**: By maintaining a dynamic list of IP addresses for matching pods, Endpoints and Endpoint Slices facilitate seamless service discovery within a cluster.
3. **Health Monitoring**: Continuous updates to Endpoints and Endpoint Slices ensure that only healthy pods receive traffic, improving the reliability of your applications.

# 5 Best Practices for Working with Endpoints and Endpoint Slices

1. **Use Readiness Probes**: Ensure that your pods are ready to receive traffic before they are added to an Endpoint or Endpoint Slice. Kubernetes provides readiness probes to help with this.
2. **Avoid Manual Management**: Let Kubernetes manage Endpoints and Endpoint Slices `automatically`. Manual management can lead to inconsistencies and increased operational overhead.
3. **Monitor Health**: Use monitoring tools to keep an eye on the health and status of Endpoints and Endpoint Slices. This can help you quickly identify and resolve issues.
