

# 1 kubectl-plugins
https://github.com/luksa/kubectl-plugins

1
其中 [kubectl-ssh](https://github.com/luksa/kubectl-plugins/blob/master/kubectl-ssh "kubectl-ssh")

kubectl ssh node             # access the node in a single-node cluster 
kubectl ssh node my-node     # access a node in a multi-node cluster

2 
kubectl really get all

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

