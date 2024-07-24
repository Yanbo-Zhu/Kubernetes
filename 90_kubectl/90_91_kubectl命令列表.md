


# 1 kubectl port-forward

https://kubernetes.io/docs/reference/kubectl/generated/kubectl_port-forward/

$ kubectl port-forward deployments/example-app 8080:8080
```
  # Listen on port 8888 locally, forwarding to 5000 in the pod
  kubectl port-forward pod/mypod 8888:5000
```

