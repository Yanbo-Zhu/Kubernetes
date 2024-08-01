
如何自己创造一个 kubectl plugin 

https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/


# 1 Krew (kubectl plugin management tool)

https://github.com/kubernetes-sigs/krew?tab=readme-ov-file

## 1.1 安装krew 以及配置环境变量 

https://krew.sigs.k8s.io/docs/user-guide/setup/install/

我在 windos  , bash on wsl, fish on wsl 中都配置好了 

windwos 安装在了 `C:\Users\yzh\.krew\bin`


## 1.2 那些kubectl plugin可以通过 krew安装 

https://krew.sigs.k8s.io/plugins/


- Run `kubectl krew install <PLUGIN_NAME>` to install a plugin via Krew.


# 2 krew 中的可用的plugins


## 2.1 kubectl-ssh-plugin-eks

https://github.com/nithu0115/kubectl-ssh-plugin-eks



This plugin will provide an easy way to SSH into Kubernetes Worker nodes running and have joined the Amazon EKS cluster using kubectl.





# 3 kubectl-plugins von luksa
https://github.com/luksa/kubectl-plugins


## 3.1 安装

我安装在了 /home/yzh/kubectl-plugins
export PATH=$PATH:$HOME/kubectl-plugins


## 3.2 kubectl ssh node

Provider-agnostic way of opening a remote shell to a Kubernetes node.

Enables you to access a node even when it doesn't run an SSH server or when you don't have the required credentials. Also, the way you log in is always the same, regardless of what provides the Kubernetes cluster (e.g. Minikube, Kind, Docker Desktop, GKE, AKS, EKS, ...)

You must have cluster-admin rights to use this plugin.

The primary focus of this plugin is to provide access to nodes, but it also provides a quick way of running a shell inside a pod.

Example usage:

```shell
kubectl ssh node             # access the node in a single-node cluster 
kubectl ssh node my-node     # access a node in a multi-node cluster
```

You can get your node name with the `kubectl get nodes` command.

You can also execute a command inside the node and quit like with command below:

```shell
kubectl ssh node my-node ls   # access a node in a multi-node cluster and execute ls
```

## 3.3 kubectl force delete

Force deletes an object by removing its finalizers and then deleting it.

Example usage:

```shell
kubectl force delete po my-stuck-pod
```

## 3.4 kubectl really get all

Lists absolutely all resources in a namespace, not just the ones returned by `kubectl get all`.

Example usage:

```shell
# List all resources in the current namespace
kubectl really get all

# List all resources in the specified namespace
kubectl really get all -n my-namespace

# List all resources in the whole cluster (all cluster-scoped 
# resources and all namespaced resources in all namespaces)
kubectl really get all --all-namespaces

# List all resources in the whole cluster with the label foo=bar
kubectl really get all --selector foo=bar

# List all resources in the whole cluster in YAML format
kubectl really get all -o yaml
```

## 3.5 kubectl really delete all

Deletes absolutely all resources in a namespace, not just the ones that `kubectl delete all` deletes.

Example usage:

```shell
kubectl really delete all
kubectl really delete all -n some-namespace
```



# 4 kubectl-plugins  von jordanwilson230

https://github.com/jordanwilson230/kubectl-plugins

我在 wsl 中安装了 
/home/yzh/kubectl-plugins

但是 /home/yzh/kubectl-plugins 没有加入环境变量 ， 也没有去使用 


里面有 
kubectl ssh 
kubectl switch
kubectl prompt
kubectl image
kubectl ipcd 
