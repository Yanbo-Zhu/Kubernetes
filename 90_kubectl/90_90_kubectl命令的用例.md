

kubectl 是一个命令行工具，用于与Kubernetes集群和其中的 pod 通信。使用它你可以查看集群的状态，列出集群中的所有 pod，进入 pod 中执行命令等。你还可以使用 YAML 文件定义资源对象，然后使用kubectl 将其应用到集群中。

查看节点状态：

`kubectl get node`

![](https://cdn.nlark.com/yuque/0/2022/png/28915315/1663748785931-438ecd5b-bee8-49c9-997c-9ffc5d1f7b1f.png)

`kubectl get pod -A`

![](https://cdn.nlark.com/yuque/0/2022/png/28915315/1663748833731-5fc73596-d048-4f97-9ce1-bf4d95fb388a.png)

`minikube ssh`进入容器，查看kubelet状态`systemctl status kubelet`

![](https://cdn.nlark.com/yuque/0/2022/png/28915315/1663748921853-09286afb-7cbc-4f52-9f97-4d86f9a66c0d.png)


```powershell
# 1、查看pod 状态的的命令
kubectl get pod -n kube-system -o wide
# 2、删除pod
kubectl delete pod/kub-flannel-ds-xxxx -n kube-system --grace-period=0 --force
# 3、生成 新的token
[root@master ~]# kubeadm token create --print-join-command
```


# 1 Case

## 1.1 Explore Kubernetes resources

```
# List all namespaces
kubectl get namespace
 
# List everything in namespace kubernetes-dashboard (-n kubernetes-dashboard)
kubectl get all -n kubernetes-dashboard
 
# List all pods in namespace kubernetes-dashboard (-n ...)
kubectl get pods -n kubernetes-dashboard
 
# List all pods in all namespaces (-A)
kubectl get pods -A
 
# Get further information on a particular Pod
kubectl describe pod kubernetes-dashboard-78f87ddfc-nbw49 -n kubernetes-dashboard
 
# List all deployments in default namespace
kubectl get deployment
 
# Get further information on a particular Deployment
kubectl describe deployment test-rail-rail-foreground
 
# ... in a similar way with the combination of get and describe, further resources can be explored, like:
# service, configmap, secret, replicaset, node, daemonset, ...
 
# Get events
kubectl events -A
```

Alternatively, you can click through the Kubernetes Dashboard and view Kubernetes resources. Remember to select the right namespace on the upper left, or "All namespaces".


## 1.2 Check the status of Pods

Use the following commands

```
# List all pods in all namespaces (-A)
kubectl get pods -A
 
# List pods in default namespace (Note: '-n default' can be omitted since it is the default)
kubectl get pods -n default
 
# Example output:
NAME                                         READY   STATUS    RESTARTS       AGE
test-rail-rail-foreground-57bc46df68-whb69   1/1     Running   1 (6d1h ago)   22d
```

If READY indicates all pods are ready and STATUS indicates they are Running, everything should be fine. 


## 1.3 Analyse failing Pods

If a Pod fails or causes issues, try to get more information to idenfity the root cause: 

```
# Show pod details
kubectl describe pod test-rail-rail-foreground-57bc46df68-whb69 -n default
# Here you can see volumes, secrets, configmaps and configuration used by the pod,
# as well as associated events, which allow further inspection. Often this helps to
# understand what could be wrong.
 
# Show logs of pod
kubectl logs test-rail-rail-foreground-57bc46df68-whb69 -n default
```

## 1.4 Restart a Pod
In some cases you may want to restart a Pod. Refer to the following page to find various ways how to do that: [https://spacelift.io/blog/restart-kubernetes-pods-with-kubectl](https://spacelift.io/blog/restart-kubernetes-pods-with-kubectl)


## 1.5 Rebuild the whole cluster

This will remove the Kubernetes cluster and all configuration and resources in it completely. The next Puppet run on the Kubernetes host will restore its initial state, undoing all manual modifications. 

This instruction is specific to the Kubernetes distro 'k0s'.

Execute the following commands on the Kubernetes Single Node Cluster host. Ignore potential error messages. 
```
puppet agent --disable
systemctl stop k0scontroller.service
k0s stop
k0s reset
rm -rf /etc/k0s
rm -rf /etc/k8s
rm -rf /root/.kube
puppet agent --enable
```

After the next puppet run, the Cluster and the IVU.plan deployment inside will be rebuilt with its original configuration. 


# 2 kubectl events

kubectl events --namespace prom-lzm -o yaml --for pod/prometheus-lzm-prometheus-0

# 3 kubectl logs
https://stackoverflow.com/questions/57007134/how-to-see-logs-of-terminated-pods
> kubectl logs 加-p ， 后边不是让你加pod name 的

kubectl logs prometheus-lzm-prometheus-0 -c prometheus -n prom-lzm -f 

```
-c, --container="": Print the logs of this container
  -f, --follow[=false]: Specify if the logs should be streamed.
      --limit-bytes=0: Maximum bytes of logs to return. Defaults to no limit.
  -p, --previous[=false]: If true, print the logs for the previous instance of the container in a pod if it exists.
      --since=0: Only return logs newer than a relative duration like 5s, 2m, or 3h. Defaults to all logs. Only one of since-time / since may be used.
      --since-time="": Only return logs after a specific date (RFC3339). Defaults to all logs. Only one of since-time / since may be used.
      --tail=-1: Lines of recent log file to display. Defaults to -1, showing all log lines.
      --timestamps[=false]: Include timestamps on each line in the log output

```


# 4 切换为 root user 

```
su - root
```


# 5 kubectl ssh:  

This kubectl addon is from here. [https://github.com/luksa/kubectl-plugins](https://github.com/luksa/kubectl-plugins). And I have verified that. T
He provides a provider-agnostic way to get a shell into a worker node if you have cluster-admin privileges.
There’s no need to manage any ssh keys either.
kubectl ssh node my-node     # access a node in a multi-node cluster


P.S. This is connecting inside of a freshly created pod on the specified node. In that sense, you do not get access to node itself (as you wanted) but (just) a `privileged` pod


You can get your node name with the command: kubectl get nodes 

## 5.1 into node

Example usage:
```
kubectl ssh node             # access the node in a single-node cluster 
kubectl ssh node my-node     # access a node in a multi-node cluster
kubectl ssh node my-node ls   # access a node in a multi-node cluster and execute ls

```

## 5.2 into pod 

```
kubectl ssh pod prometheus-lzm-prometheus-0 -c prometheus -n prom-lzm                                                                    2484ms  Sat Oct 12 13:21:08 2024
/prometheus $
/prometheus $ sudo su
sh: sudo: not found
/prometheus $ su - root
su: must be suid to work properly
/prometheus $

```



# 6 kubectl exec 
## 6.1 ssh into the pod
https://kubernetes.io/docs/tasks/debug/debug-application/get-shell-running-container/
https://www.valewood.org/topics/devops/learn/technology/kubernetes/how-to-ssh-into-a-k8s-pod/
\
> The double dash (--) separates the arguments you want to pass to the command from the kubectl arguments.
> The short options -i and -t are the same as the long options --stdin and --tty

### 6.1.1 得到所有的 pods

kubectl get pod -n default 

```
NAME                                       READY   STATUS    RESTARTS   AGE
alertmanager-app-alertmanager-0            2/2     Running   0          39d
app-operator-c489b4bc-dwlj4                1/1     Running   0          39d
prom-grafana-69d87c84f9-d9pxj              3/3     Running   0          6d16h
prom-kube-state-metrics-5cb887bff6-82kzr   1/1     Running   0          39d
prom-prometheus-node-exporter-24vnb        1/1     Running   0          3d
prom-prometheus-node-exporter-hdn8x        1/1     Running   0          2d23h
prom-prometheus-node-exporter-ql8cz        1/1     Running   0          39d
prom-prometheus-node-exporter-vlxrj        1/1     Running   0          6d23h
prometheus-app-prometheus-0                2/2     Running   0          6d16h
promtail-6lndq                             1/1     Running   0          2d23h
promtail-8hmqm                             1/1     Running   0          2d23h
promtail-rrfdx                             1/1     Running   0          2d23h
promtail-vfjgp                             1/1     Running   0          2d23h
```
### 6.1.2 Running individual commands in a container 

In an ordinary command window, not your shell, list the environment variables in the running container:

```shell
kubectl exec shell-demo -- env
```

Experiment with running other commands. Here are some examples:

```shell
kubectl exec shell-demo -- ps aux
kubectl exec shell-demo -- ls /
kubectl exec shell-demo -- cat /proc/1/mounts
```

### 6.1.3 实操

First, ensure you know the name of the pod you want to access. You can list all pods in a namespace with the command: `kubectl get pods -n <namespace>.`


To get a shell access to the container running inside the application pod, all you have to do is:


```
kubectl exec -ti --namespace <your namespace> <your pod name> -- /bin/sh
kubectl exec -it <pod-name> -n <namespace> -- /bin/bash
kubectl exec -it <pod-name> -n <namespace> -- /bin/sh
kubectl exec -it <pod-name> -n <namespace> -- /bin/zsh


# e2x dev
kubectl exec -it prom-grafana-69d87c84f9-d9pxj -n default -- /bin/bash

nslookup e2x-d3042e-a01.ivu-cloud.local   ->  172.18.92.162
curl e2x-d3042e-a01.ivu-cloud.local:32000/metrics
```

then you can try something like /bin/sh or /bin/zsh

This will open a shell inside of your application container. You can now execute any command you need.

## 6.2 ssh into a container in a poad

### 6.2.1 得到 container name

kubectl describe pod prom-grafana-69d87c84f9-d9pxj
选中 你你想要进入的 container 的name 和 id 

```
Name:             prom-grafana-69d87c84f9-d9pxj
Namespace:        default
Priority:         0
Service Account:  prom-grafana
Node:             ip-172-18-88-144.eu-central-1.compute.internal/172.18.88.144
Start Time:       Thu, 25 Jul 2024 22:35:19 +0200
Labels:           app.kubernetes.io/instance=prom
                  app.kubernetes.io/name=grafana
                  pod-template-hash=69d87c84f9
Annotations:      checksum/config: ab75e529bf07ff6b06c605c3d024f70359ca299af13c7a32cd9e662eefb8c065
                  checksum/sc-dashboard-provider-config: e70bf6a851099d385178a76de9757bb0bef8299da6d8443602590e44f05fdf24
                  checksum/secret: 032056e9c62bbe9d1daa41ee49cd3d9524c076f51ca4c65adadf4ef08ef28712
                  kubectl.kubernetes.io/default-container: grafana
Status:           Running
IP:               100.64.63.90
IPs:
  IP:           100.64.63.90
Controlled By:  ReplicaSet/prom-grafana-69d87c84f9
Containers:
  grafana-sc-dashboard:
    Container ID:    containerd://e81734692bc528a4c7281ad6e005234d219415aabc1cd2a50b0aa7df702171c9
    Image:           quay.io/kiwigrid/k8s-sidecar:1.26.1
    Image ID:        quay.io/kiwigrid/k8s-sidecar@sha256:b8d5067137fec093cf48670dc3a1dbb38f9e734f3a6683015c2e89a45db5fd16
    Port:            <none>
    Host Port:       <none>
    SeccompProfile:  RuntimeDefault
    State:           Running
      Started:       Thu, 25 Jul 2024 22:35:20 +0200
    Ready:           True
    Restart Count:   0
    Environment:
      METHOD:        WATCH
      LABEL:         grafana_dashboard
      LABEL_VALUE:   1
      FOLDER:        /tmp/dashboards
      RESOURCE:      both
      NAMESPACE:     ALL
      REQ_USERNAME:  <set to the key 'admin-user' in secret 'prom-grafana'>      Optional: false
      REQ_PASSWORD:  <set to the key 'admin-password' in secret 'prom-grafana'>  Optional: false
      REQ_URL:       http://localhost:3000/api/admin/provisioning/dashboards/reload
      REQ_METHOD:    POST
    Mounts:
      /tmp/dashboards from sc-dashboard-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-dxqv9 (ro)
  grafana-sc-datasources:
    Container ID:    containerd://7aeabf0f79c3c986026f04014fbadb338525bda7bc1692982fd29b62589a47b2
    Image:           quay.io/kiwigrid/k8s-sidecar:1.26.1
    Image ID:        quay.io/kiwigrid/k8s-sidecar@sha256:b8d5067137fec093cf48670dc3a1dbb38f9e734f3a6683015c2e89a45db5fd16
    Port:            <none>
    Host Port:       <none>
    SeccompProfile:  RuntimeDefault
    State:           Running
      Started:       Thu, 25 Jul 2024 22:35:20 +0200
    Ready:           True
    Restart Count:   0
    Environment:
      METHOD:        WATCH
      LABEL:         grafana_datasource
      LABEL_VALUE:   1
      FOLDER:        /etc/grafana/provisioning/datasources
      RESOURCE:      both
      REQ_USERNAME:  <set to the key 'admin-user' in secret 'prom-grafana'>      Optional: false
      REQ_PASSWORD:  <set to the key 'admin-password' in secret 'prom-grafana'>  Optional: false
      REQ_URL:       http://localhost:3000/api/admin/provisioning/datasources/reload
      REQ_METHOD:    POST
    Mounts:
      /etc/grafana/provisioning/datasources from sc-datasources-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-dxqv9 (ro)
  grafana:
    Container ID:    containerd://872147dd9c981e812683fd46d3a08d6b0afc7dd116786813d0ad464498993048
    Image:           docker.io/grafana/grafana:11.0.0
    Image ID:        docker.io/grafana/grafana@sha256:0dc5a246ab16bb2c38a349fb588174e832b4c6c2db0981d0c3e6cd774ba66a54
    Ports:           3000/TCP, 9094/TCP, 9094/UDP
    Host Ports:      0/TCP, 0/TCP, 0/UDP
    SeccompProfile:  RuntimeDefault
    State:           Running
      Started:       Thu, 25 Jul 2024 22:35:20 +0200
    Ready:           True
    Restart Count:   0
    Liveness:        http-get http://:3000/api/health delay=60s timeout=30s period=10s #success=1 #failure=10
    Readiness:       http-get http://:3000/api/health delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:
      POD_IP:                       (v1:status.podIP)
      GF_SECURITY_ADMIN_USER:      <set to the key 'admin-user' in secret 'prom-grafana'>      Optional: false
      GF_SECURITY_ADMIN_PASSWORD:  <set to the key 'admin-password' in secret 'prom-grafana'>  Optional: false
      GF_PATHS_DATA:               /var/lib/grafana/
      GF_PATHS_LOGS:               /var/log/grafana
      GF_PATHS_PLUGINS:            /var/lib/grafana/plugins
      GF_PATHS_PROVISIONING:       /etc/grafana/provisioning
    Mounts:
      /etc/grafana/grafana.ini from config (rw,path="grafana.ini")
      /etc/grafana/provisioning/dashboards/sc-dashboardproviders.yaml from sc-dashboard-provider (rw,path="provider.yaml")
      /etc/grafana/provisioning/datasources from sc-datasources-volume (rw)
      /tmp/dashboards from sc-dashboard-volume (rw)
      /var/lib/grafana from storage (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-dxqv9 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  config:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      prom-grafana
    Optional:  false
  storage:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  sc-dashboard-volume:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  sc-dashboard-provider:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      prom-grafana-config-dashboards
    Optional:  false
  sc-datasources-volume:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  kube-api-access-dxqv9:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
```

### 6.2.2 使用 kubectl exec 进入

If a Pod has more than one container, use `--container` or `-c` to specify a container in the `kubectl exec` command. For example, suppose you have a Pod named my-pod, and the Pod has two containers named _main-app_ and _helper-app_. The following command would open a shell to the _main-app_ container.

```shell
kubectl exec -i -t my-pod --container main-app -- /bin/bash
```

```
kubectl exec --stdin --tty <pod-name> --container prometheus -n <namespace> -- /bin/bash
kubectl exec --stdin --tty app-operator-c489b4bc-sqpmf --container kube-prometheus-stack -n default -- /bin/bash


kubectl exec --stdin --tty prometheus-app-prometheus-0 -n default -- sh
```



### 6.2.3 使用  kubectl ssh plugin


https://github.com/jordanwilson230/kubectl-plugins

`Usage: kubectl ssh [OPTIONAL: -n <namespace>] [OPTIONAL: -u <user>] [OPTIONAL: -c <Container Name>] [REQUIRED: <PodName> ] -- [command]`

Example: 
kubectl ssh -n default -u root -c prometheus prometheus-282sd0s2 -- bash
kubectl ssh pod prometheus-lzm-prometheus-0 -c prometheus -n prom-lzm -u root -- bash
