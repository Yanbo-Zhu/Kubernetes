
# 1 kubernetes abbreviation

certificates = cert, certs
certificiaterequests = cr, crs
certificatesigningrequests = csr
componentstatuses = cs
configmaps = cm
cronjobs = cj
customresourcedefinitions = crd, crds
daemonsets = ds
deployments = deploy
endpoints = ep
events = ev
horizontalpodautoscalers = hpa
ingresses = ing
limitranges = limits
namespaces = ns
networkpolicies = netpol
nodes = no
persistentvolumes = pv
persistentvolumeclaims = pvc
pods = po
podsecuritypolicies = psp
priorityclasses = pc
replicationcontrollers = rc
replicasets = rs
resourcequotas = quota
scheduledscalers = ss
services = svc
serviceaccounts = sa
statefulsets = sts
storageclasses = sc


# 2 kubectl port-forward

https://kubernetes.io/docs/reference/kubectl/generated/kubectl_port-forward/

$ kubectl port-forward deployments/example-app 8080:8080
```
  # Listen on port 8888 locally, forwarding to 5000 in the pod
  kubectl port-forward pod/mypod 8888:5000
```

