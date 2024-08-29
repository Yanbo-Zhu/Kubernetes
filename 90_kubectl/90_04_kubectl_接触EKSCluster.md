

# 1 使用aws-adfs login 登录一个aws account 

➜  ~ aws-adfs login --region eu-central-1 --adfs-host adfs02.ivu-cloud.com

➜  ~ aws-adfs login --profile ivu-cloud-e2x \
      --role-arn arn:aws:iam::681290371536:role/Admin_ADFS \
      --region eu-central-1 \
      --adfs-host adfs02.ivu-cloud.com




# 2 Check that kubectl is properly configured by getting the cluster state


https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/

In order for kubectl to find and access a Kubernetes cluster, it needs a [kubeconfig file](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/), which is created automatically when you create a cluster using [kube-up.sh](https://github.com/kubernetes/kubernetes/blob/master/cluster/kube-up.sh) or successfully deploy a Minikube cluster. By default, kubectl configuration is located at `~/.kube/config`.


1 
Check that kubectl is properly configured by getting the cluster state:

```shell
kubectl cluster-info
```
If you see a URL response, kubectl is correctly configured to access your cluster.


2 
If you see a message similar to the following, kubectl is not configured correctly or is not able to connect to a Kubernetes cluster.
```
The connection to the server <server-name:port> was refused - did you specify the right host or port?
```
For example, if you are intending to run a Kubernetes cluster on your laptop (locally), you will need a tool like [Minikube](https://minikube.sigs.k8s.io/docs/start/) to be installed first and then re-run the commands stated above.


3 
If `kubectl cluster-info` returns the url response but you can't access your cluster, to check whether it is configured properly, use:
```shell
kubectl cluster-info dump
```


# 3 查看 kubectl config file 


Kubectl config get-contexts
```
CURRENT   NAME                                                 CLUSTER                                                    AUTHINFO                                                   NAMESPACE
          arn:aws:eks:eu-central-1:618143434103:cluster/main   arn:aws:eks:eu-central-1:618143434103:cluster/main         arn:aws:eks:eu-central-1:618143434103:cluster/main
*         arn:aws:eks:eu-central-1:681290371536:cluster/dev    arn:aws:eks:eu-central-1:681290371536:cluster/dev          arn:aws:eks:eu-central-1:681290371536:cluster/dev
          docker-desktop                                       docker-desktop                                             docker-desktop
          e20-d3042                                            local-e20-d3042                                            user-e20-d3042
          e20-d3042b                                           cluster-local-e20-d3042b                                   user-e20-d3042b
          e20-eks-cluster-main                                 arn:aws:eks:eu-central-1:618143434103:cluster/main         arn:aws:eks:eu-central-1:618143434103:cluster/main
          e2x-eks-cluster-dev                                  arn:aws:eks:eu-central-1:681290371536:cluster/dev          arn:aws:eks:eu-central-1:681290371536:cluster/dev
          e2x-eks-cluster-eks-dev-ex                           arn:aws:eks:eu-central-1:681290371536:cluster/eks-dev-ex   arn:aws:eks:eu-central-1:681290371536:cluster/eks-dev-ex
```


kubectl config view, --cluster string, --context string

kubectl config view
```
B2DE1F9328FC52F6DC1323E665CD8.gr7.eu-central-1.eks.amazonaws.com
  name: arn:aws:eks:eu-central-1:618143434103:cluster/main
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://08A0408DF33822C2DE2D141CFECDB47F.gr7.eu-central-1.eks.amazonaws.com
  name: arn:aws:eks:eu-central-1:681290371536:cluster/dev
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://EFAA54F84DDE50C5290994816323A66C.gr7.eu-central-1.eks.amazonaws.com
  name: arn:aws:eks:eu-central-1:681290371536:cluster/eks-dev-ex
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://172.18.80.141:6443
  name: cluster-local-e20-d3042b
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://kubernetes.docker.internal:6443
  name: docker-desktop
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://172.18.80.55:6443
  name: local-e20-d3042
contexts:
- context:
    cluster: arn:aws:eks:eu-central-1:618143434103:cluster/main
    user: arn:aws:eks:eu-central-1:618143434103:cluster/main
  name: arn:aws:eks:eu-central-1:618143434103:cluster/main
- context:
    cluster: arn:aws:eks:eu-central-1:681290371536:cluster/dev
    user: arn:aws:eks:eu-central-1:681290371536:cluster/dev
  name: arn:aws:eks:eu-central-1:681290371536:cluster/dev
- context:
    cluster: docker-desktop
    user: docker-desktop
  name: docker-desktop
- context:
    cluster: local-e20-d3042
    user: user-e20-d3042
  name: e20-d3042
- context:
    cluster: cluster-local-e20-d3042b
    user: user-e20-d3042b
  name: e20-d3042b
- context:
    cluster: arn:aws:eks:eu-central-1:618143434103:cluster/main
    user: arn:aws:eks:eu-central-1:618143434103:cluster/main
  name: e20-eks-cluster-main
- context:
    cluster: arn:aws:eks:eu-central-1:681290371536:cluster/dev
    user: arn:aws:eks:eu-central-1:681290371536:cluster/dev
  name: e2x-eks-cluster-dev
- context:
    cluster: arn:aws:eks:eu-central-1:681290371536:cluster/eks-dev-ex
    user: arn:aws:eks:eu-central-1:681290371536:cluster/eks-dev-ex
  name: e2x-eks-cluster-eks-dev-ex
current-context: arn:aws:eks:eu-central-1:681290371536:cluster/dev
kind: Config
preferences: {}
users:
- name: arn:aws:eks:eu-central-1:618143434103:cluster/main
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      args:
      - --region
      - eu-central-1
      - eks
      - get-token
      - --cluster-name
      - main
      - --output
      - json
      command: aws
      env:
      - name: AWS_PROFILE
        value: ivu-cloud-e20
      interactiveMode: IfAvailable
      provideClusterInfo: false
- name: arn:aws:eks:eu-central-1:681290371536:cluster/dev
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      args:
      - --region
      - eu-central-1
      - eks
      - get-token
      - --cluster-name
      - dev
      - --output
      - json
      command: aws
      env:
      - name: AWS_PROFILE
        value: ivu-cloud-e2x
      interactiveMode: IfAvailable
      provideClusterInfo: false
- name: arn:aws:eks:eu-central-1:681290371536:cluster/eks-dev-ex
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      args:
      - --region
      - eu-central-1
      - eks
      - get-token
      - --cluster-name
      - eks-dev-ex
      - --output
      - json
      command: aws
      env:
      - name: AWS_PROFILE
        value: ivu-cloud-e2x
      interactiveMode: IfAvailable
      provideClusterInfo: false
- name: docker-desktop
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
- name: user-e20-d3042
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
- name: user-e20-d3042b
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
```
# 4 连接远程的cluster


## 4.1 直接登录Cluster所在的host_然后使用kubectl to connect a cluster 

https://confluence.ivu.de/display/SYS/k8s+Single+Node+Clusters+by+Puppet+for+QS24#k8sSingleNodeClustersbyPuppetforQS24-Gettingstartedwiththek8sSingleNodeCluster

1. Log on to the Kubernetes host using SSH. 
2. Become root. 
3. Use command 'kubectl'
4. Alternatively, use 'k0s kc' to make use of the command completion.

## 4.2 本地安装kubectl_连接远程的cluster

先 使用aws-adfs login 登录一个aws account 

### 4.2.1 setting up a kubeconfig

To control the cluster with kubectl remotely from another host, perform the following steps.

#### 4.2.1.1 windows 中
Install the configuration file for access to your cluster, either
1. Copy the file `/root/.kube/config `from the Kubernetes Host in aws umgebung into the directory created in the previous step, `c:\Users\yzh\.kube\config-e20-d2034`.  or 
2. On a single-node AWS system download [`http://e20-<environment>-a01.ivu-cloud.local/kubeconfig.txt`] (`http://e20-%3cenvironment%3e-a01.ivu-cloud.local/kubeconfig.txt` ), rename the resulting file to config and copy it into `c:\Users\yzh\.kube\config-e20-d2034`,  the directory created in the previous step, or



某个 single node cluster 中的内容 
将   name: Default, current-context: Default 改为   name: e20-d2034 ,   current-context: e20-d2034   , 然后将 整个文件存到 `c:\Users\yzh\.kube\config-e20-d2034` 中 

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURBRENDQWVpZ0F3SUJBZ0lVZFNOMmtlZ0JSN2pON056c3ZFTlh5TE1VcXBrd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0dERVdNQlFHQTFVRUF4TU5hM1ZpWlhKdVpYUmxjeTFqWVRBZUZ3MHlOREEzTWpJeE9USTNNREJhRncwegpOREEzTWpBeE9USTNNREJhTUJneEZqQVVCZ05WQkFNVERXdDFZbVZ5Ym1WMFpYTXRZMkV3Z2dFaU1BMEdDU3FHClNJYjNEUUVCQVFVQUE0SUJEd0F3Z2dFS0FvSUJBUURjSFZ1RzhSeVp1WlJhM255d1NSWlhubVIxaEIyTER2U0cKUkNybkwwSWY4bFd6elhLQnR3M1pmQ2lTbUlmVGtVdEwvajZKYWxDdlZhYXhJaXd0ZHYxR3ZoUTdjd3gzU2VDbwpjVjNhWWQ3ckNva2FEQk9FWDgyd3JjTVRvTmZPdEZGcWVOVU9HQTdEbHlhNkczSUNvWDZ6cjF5bEdqdmlDcWZoCkVHUkUweE1JNUdGQU9rczJqUWVDclB2aHBuM0VvQUtmQmYxM3R4REdsYlVMR0ZsV0hFS2ZPS2x2NXE1M1Rsbk4KL3dlc1R5L09qamtwdXk0c1NpN1ozM3FjL25GTzBOcVRTZnVrbU9UVExveFhoUzdibGhHOHBpblpIaWhPeXk2Qwo2b0ZVSGNtd0xyT3ZmSDJXNjc3bHl0MEhRYUFCdU1UMWFEblRxTktnRnZnZCs1ekJuSSs3QWdNQkFBR2pRakJBCk1BNEdBMVVkRHdFQi93UUVBd0lCQmpBUEJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJUUG9lT0oKeXE0QWl3Z0ZIQlRuczhXMjV2Z0d1REFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBQXIvb0hYTUxNSGx0Ulp0UQpqeXNqVGRiSmdBZlVwUnpoODZOR1hOMmJJdkFxUmNJbmZCd3l3WUY3NFNoZ0JzWGdhYXp4WWF0ZEdiZDlQb1NOCllqaHR2VnZuczMrVEQyOENhaEU2SGRNeVNycTN3SnlMaHR1SE1xZ0JpdnlIQmNVMWlwMG42RjhUQXE0dTF6eTgKTmtJS1JFbEVFSEpmaUI4SVJHbTV1cXB0Y3lOMURiWFptZ3BmdVhSMW42cHFOVHU2VCsrYnRKV0pIV0JqbzdENApJL0hUdHdXUjJtWjBSTlkweDBUaERwOHVhclpHWVR6dUNTeU5mdmw1RFBlQ0VtcWZ0aERpdlFEMkw0MDVOaWF6CkhiSWZhOFRvMlBoRllUQlQwTUZENUhjRFNad2UvTVEzQXhybk1nRTdnUWsvUXBkbmR3bHFYaTBZcXZkdG9neEYKVkMySnBnPT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    server: https://172.18.80.55:6443
  name: local
contexts:
- context:
    cluster: local
    user: user
  name: Default
current-context: Default
kind: Config
preferences: {}
users:
- name: user
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURXVENDQWtHZ0F3SUJBZ0lVUWY4dzFmT01XOEhBK2FUbUtEbXp0MFJ5dUo0d0RRWUpLb1pJaHZjTkFRRUwKQlFBd0dERVdNQlFHQTFVRUF4TU5hM1ZpWlhKdVpYUmxjeTFqWVRBZUZ3MHlOREEzTWpJeE9USTNNREJhRncweQpOVEEzTWpJeE9USTNNREJhTURReEZ6QVZCZ05WQkFvVERuTjVjM1JsYlRwdFlYTjBaWEp6TVJrd0Z3WURWUVFECkV4QnJkV0psY201bGRHVnpMV0ZrYldsdU1JSUJJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBTUlJQkNnS0MKQVFFQXpDcEZOVjFacEliNURvMVpOVDBHc2gxVXZjVGxjZjFCVUd6eHBoUnlHaUhibXB6RGpCZnF5Z0FlOHFGcgpISGNBbnJ4T3JaMnJOM2FsbG5uOWdZRDFhQkRPOUtWNXlFOVBmdWFweWxMVnZoV2JvZGNXUE1hTkZXakZ6SEpUCnJ2bSs0Z3VjMUF6Z2RERlo0V1VpdGJ6SkxRczJUR1VzSzVFazZHeTRxVnpJZHdTaEtWVDU5R1JpNXA3V2NDVzAKQjJSQVc4T3NFd1Y2N2ZQQ3pPU0Yzb2VWMXBwSmJ1TmlqdzlxM1NHR0JhWFhKVm9UelpXbGthd3o0MG4wSXpxSQpRWXB1ZGN6Vy9QZmlpdGtDZFJDSkVYVlQzZndMNlNpTDgrMlJJSkFGSWFrUzhtS3VHbzU5NlNzWGhLV3Q4OEpuCmJpMkEwTUh1QVhmQW9mWVFPN3I0VmJmb1N3SURBUUFCbzM4d2ZUQU9CZ05WSFE4QkFmOEVCQU1DQmFBd0hRWUQKVlIwbEJCWXdGQVlJS3dZQkJRVUhBd0VHQ0NzR0FRVUZCd01DTUF3R0ExVWRFd0VCL3dRQ01BQXdIUVlEVlIwTwpCQllFRkhzWSs5U1oveW5Ca3NDVUFYUC9IVnQraytYV01COEdBMVVkSXdRWU1CYUFGTStoNDRuS3JnQ0xDQVVjCkZPZXp4YmJtK0FhNE1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQ3REM1YzSXo3YVVYbXo5aUswakJqaVBCNTYKZXg5OE1ycnVUUGVHdHE0YnVVdkxpa0d0MzNVeTZPQTNlNE5JdGs3bnpwamU2L1dtTEtrU0JMV1F6WlBuZ25xQwprZUxjVXlpZklFNjRHak55Wi83V1FPaTdNWW5HQ1l4UU9OUnBqQnl2SU85YUFaQW1acGRpVm9QNGQ4VW5nbEZDCnA5aUZYeVgxVHkvS2JjMU83RE42Qmxlc25sYW8vOGJHOG9RTnpNZlNaYS9pSUpxellEbTJHSlBEQUFGMkd5ME0KWTlIRjUwd2dRN0xXM2kyQkFyQkYyTnkvNTdDR0Zta3NROWlmeDhGTlJmb0NkVHo4ZXhIZjczeUtCWE56SmV5eQpvQysrVVR0RVhIcjBlMHlhYkdkVHN5Y2hvY0x2T1BWRC9nZkpSMUtoQkY0dzNSWUlmWU9xU0Q5SWc0bFMKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBekNwRk5WMVpwSWI1RG8xWk5UMEdzaDFVdmNUbGNmMUJVR3p4cGhSeUdpSGJtcHpECmpCZnF5Z0FlOHFGckhIY0FucnhPcloyck4zYWxsbm45Z1lEMWFCRE85S1Y1eUU5UGZ1YXB5bExWdmhXYm9kY1cKUE1hTkZXakZ6SEpUcnZtKzRndWMxQXpnZERGWjRXVWl0YnpKTFFzMlRHVXNLNUVrNkd5NHFWeklkd1NoS1ZUNQo5R1JpNXA3V2NDVzBCMlJBVzhPc0V3VjY3ZlBDek9TRjNvZVYxcHBKYnVOaWp3OXEzU0dHQmFYWEpWb1R6WldsCmthd3o0MG4wSXpxSVFZcHVkY3pXL1BmaWl0a0NkUkNKRVhWVDNmd0w2U2lMOCsyUklKQUZJYWtTOG1LdUdvNTkKNlNzWGhLV3Q4OEpuYmkyQTBNSHVBWGZBb2ZZUU83cjRWYmZvU3dJREFRQUJBb0lCQUQvMmJrQk53cXZjN1dJMQp2bUVEZ1htRDN1eWxKdzBUUHNjbm1uMWhvbTIxZGN0Mm1Yem1jVlJmdlZKZVorUS9GQjZtK0M5RVdrUENGQmFVCm1XdGRMNFV1M3VlZWdBelZiQjVodllmNWM2VmR0Nmg0YmlzVU9WY2Z2L2hSU1E1a2gvemhqbnlRbkRGSzFOSGMKa1lkR1lmQ090ckF3Y2lva0N1QVdUN1Fmc1Y0M3NkRTRYOVZCS2tFYUhCS0w3cTdpb0ZRVkkrTmNGcStHUVJSTwpkdmZicHg4MjJZWS81bE1paFYzN0dGN0lReUFlaVRoZFY2dFZaUktNU2tLUkJUK1phcHVUL2NZWnVITnozUlJLClVuS2ZsVERlZEUxSFBaS29ramFKaXBqRkJVczNiV1JLWnRkd2hKa1p5NFlEVnBEeW8yU1JpTWx0S1FNalR4Rk8KVnY5ZXBPRUNnWUVBNE9RMTNhcG5YUTNzV29ObUh5YmpRdCs0NjVKWEVFemxsN3NlK2lhOEVrcWc5eXowSnQvdgo3RWpKZGFESWNiMUYreTJkam9jdldtejVQYi9OZ0FHWGNGUk1QZE03djRPK09WQkxIajZGWmlxa01UWmpoRWxDCmh4SlREQ3QyalkvSUExMVljc0tzbTMvZmQxUkpYZTZydmxMK3NjQVBBekNkZDRna0w0cFBKM1VDZ1lFQTZHZ2EKOVRmRnEyWFZsSTNXMjJtanFYYkpkOVMzM2ZEdzFjM015VlVjM3JxN0JUQ0N6U0I1Z25hZEcvMjRCR1hweVhDTwp0UWdIMTJzRzQ4bVRWbGdCT3AxbnZwZURNeEtGanRiNTVsazRpK1NzSUN6ZGlEZHZGS1BQdkZZMVZvMFdXR2JWClFBc0ZsMGQ1bVorOGdqQWtzUmhvakhiL3JTRmxJeGVxMUdUTW1MOENnWUVBdi9JSUI2bnBqd0xUT0czdU82aDEKVUI2ak5tMHc0amkvdlVGNHJ3bGdmRHcySnNHM29YYUEwS3RQUjVaemZxQ05tbFRVcFZHOG1QRnB5Y3ByRzFaQQpheCtIOHp6WFFoNnZ2VHRLNGdWNjFqaU82M0lBZm1nSTRFQkRvWjkzRUZ3bjJyZFJScUhoc05ielpHWU1PSi84CjlmVGdiZFgrK2pvRUxJR0dZUTByZ2dFQ2dZQXQ4NWVROEt3V3paWERQNkJCMWN0VVVIWlpTU2ZwckNBU1JoUzkKb2lTSkxXYVpDaFJ5dG44UCtwL3B6dXE1ZytVTHZVT2FLN0pSTjRvdk04WDhCbjZIdG1PeTNZWkZiTjRYZGc0SApLNUR3cUJBWWRJYUF6bVVhTWFGN1haaENPcmMyVjI3R3NPYzBHQi9FN2o1NWgwZFo5TGVIUG1nak5UMG9DUi92Cnl0WmhSUUtCZ1FER2tQSHRnMzJjOHdodEJnNHNRSjByOU85Y3J2cy93RHlVbzJjaXZtZ1FwbkllRVFBb2RGVVcKcXVpNUQxQ2NGUk5JNWpsMDIrRDV5TjczUWloWjY5WkJuYUludmhoc3BKTUlJVHFJS1dZUyt1WlJHV3pzWVlWZgpuQVk3dTFoLzl0ZGd4dCs3ai9PU3MxY1dZU3ZoMmkzTEJMcGVrU0NWQzZodFJUYjlKWHNjdUE9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=

```



2 
将 `c:\Users\yzh\.kube\config-e20-d2034` 这个路径添加到 env variable Kubeconfig 中 


You are now connected as cluster admin to Kubernetes. Note that in this role, you may not only monitor, but also reconfigure and inherently misconfigure / destroy resources in the Kubernetes cluster. On the other hand, this is a Dedicated Single Node Cluster, hence the only system that may be harmed is this dedicated cluster and the IVU.plan environment within.


### 4.2.2 wsl 中 


在 ~/.bashrc add
将 新的 kubeconfig 文件的路径  添加到 KUBECONFIG 最后 ， 然后 执行 source ~/.bashrc 
```
export KUBECONFIG="${KUBECONFIG}:config-demo:config-demo-2"

export KUBECONFIG="${KUBECONFIG}:${HOME}/.kube/config"

export KUBECONFIG="${KUBECONFIG}:${HOME}/.kube/config:/mnt/c/Users/yzh/.kube/config-e20-d3042:/mnt/c/Users/yzh/.kube/config-e20-d3042b:/mnt/c/Users/yzh/.kube/config-e20-eks-cluster-main:/mnt/c/Users/yzh/.kube/config-e2x-eks-cluster-dev:/mnt/c/Users/yzh/.kube/config-e2x-eks-cluster-eks-dev-ex"


```



### 4.2.3 查到你想访问的 cluster 的  content name 

执行  kubectl config view, 查到你想访问的 cluster 的  content name 


### 4.2.4 验收
1 
不一定在 `c:\Users\yzh\.kube` 路径下 ,  随便哪个路径下

执行  kubectl config use-context e20-d2034, 得到 output 为 Switched to context "e20-d2034"

然后执行 
➜  ~ kubectl get pods -A
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE
kube-system   aws-node-dbz57                       1/1     Running   0          4d20h
kube-system   coredns-7bc655f56f-4th8h             1/1     Running   0          4d20h
kube-system   coredns-7bc655f56f-9l8dn             1/1     Running   0          4d20h
kube-system   ebs-csi-controller-ccd784cbf-ld7p5   6/6     Running   0          4d20h
kube-system   ebs-csi-controller-ccd784cbf-w24x4   6/6     Running   0          4d20h
kube-system   ebs-csi-node-j47lj                   3/3     Running   0          4d20h
kube-system   kube-proxy-8w4g7                     1/1     Running   0          4d20h



2 
在 `c:\Users\yzh\.kube` 路径下 , 
执行  kubectl config view, 查到你想访问的 cluster 的  name 
执行  kubectl config use-context e20-d2034 后, 
在执行 kubectl get pods -A,   这个 kubectl 使用的 仍然是 `c:\Users\yzh\.kube\config` 中的 配置. 不是 `c:\Users\yzh\.kube\config-e20-d2034 ` 中的 配置

所以 此时 执行 kubectl get pods -A 会出现错误 
```
E0723 17:19:40.944381    5928 memcache.go:265] couldn't get current server API group list: Get "https://kubernetes.docker.internal:6443/api?timeout=32s": dial tcp 127.0.0.1:6443: connectex: Es konnte keine Verbindung hergestellt werden, da der Zielcomputer die Verbindung verweigerte.
E0723 17:19:41.005270    5928 memcache.go:265] couldn't get current server API group list: Get "https://kubernetes.docker.internal:6443/api?timeout=32s": dial tcp 127.0.0.1:6443: connectex: Es konnte keine Verbindung hergestellt werden, da der Zielcomputer die Verbindung verweigerte.
E0723 17:19:41.006345    5928 memcache.go:265] couldn't get current server API group list: Get "https://kubernetes.docker.internal:6443/api?timeout=32s": dial tcp 127.0.0.1:6443: connectex: Es konnte keine Verbindung hergestellt werden, da der Zielcomputer die Verbindung verweigerte.
E0723 17:19:41.007986    5928 memcache.go:265] couldn't get current server API group list: Get "https://kubernetes.docker.internal:6443/api?timeout=32s": dial tcp 127.0.0.1:6443: connectex: Es konnte keine Verbindung hergestellt werden, da der Zielcomputer die Verbindung verweigerte.
E0723 17:19:41.009074    5928 memcache.go:265] couldn't get current server API group list: Get "https://kubernetes.docker.internal:6443/api?timeout=32s": dial tcp 127.0.0.1:6443: connectex: Es konnte keine Verbindung hergestellt werden, da der Zielcomputer die Verbindung verweigerte.
Unable to connect to the server: dial tcp 127.0.0.1:6443: connectex: Es konnte keine Verbindung hergestellt werden, da der Zielcomputer die Verbindung verweigerte.
```

所以 此时 需要时 在 `c:\Users\yzh\.kube` 路径下   执行 kubectl get pods -A  --kubeconfig=config-e20-d2034  不会报错了 




# 5 已经连接到cluster_让探后查用那个身份在操作的 


It is not always obvious what attributes (username, groups) you will get after authenticating to the cluster. It can be even more challenging if you are managing more than one cluster at the same time.

There is a `kubectl` subcommand to check subject attributes, such as username, for your selected Kubernetes client context: 
`kubectl auth whoami`.


# 6 EKS Cluster 相关 

## 6.1 查看这 aws account 有几个 eks cluster 

先使用aws-adfs login 登录一个aws account 

查看这 aws account 有几个 eks cluster 
`aws --profile ivu-cloud-e2x eks list-clusters`


```
 ⚡ 🦄  aws --profile ivu-cloud-e20 eks list-clusters
{
    "clusters": [
        "eks-dev-staging-e20",
        "main"
    ]
}

 ⚡ 🦄  aws --profile ivu-cloud-e2x eks list-clusters
{
    "clusters": [
        "dev",
        "eks-dev-ex"
    ]
}

```

## 6.2 Create or update a kubeconfig file for your cluster

https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html

update-kubeconfig 的命令的说明 
https://docs.aws.amazon.com/cli/latest/reference/eks/update-kubeconfig.html

```
aws eks update-kubeconfig --region region-code --name my-cluster

aws --profile ivu-cloud-e2x eks update-kubeconfig --name titanic-e2x
```

By default, the resulting configuration file is created at the default kubeconfig path (.kube) in your home directory or merged with an existing config file at that location. You can specify another path with the `--kubeconfig` option.

--kubeconfig (string) Optionally specify a kubeconfig file to append with your configuration. By default, the configuration is written to the first file path in the KUBECONFIG environment variable (if it is set) or the default kubeconfig path (.kube/config) in your home directory.


```
我用的是 
aws --profile ivu-cloud-e20 eks update-kubeconfig --name main --kubeconfig c:\Users\yzh\.kube\config-e20-eks-cluster-main
```
这个命令会将 elk cluster main 的信息 写入到  `c:\Users\yzh\.kube\config-e20-eks-cluster-main` 中


After setting up a kubeconfig, 
```

➜  ~ kubectl get pods -A
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE
kube-system   aws-node-dbz57                       1/1     Running   0          4d20h
kube-system   coredns-7bc655f56f-4th8h             1/1     Running   0          4d20h
kube-system   coredns-7bc655f56f-9l8dn             1/1     Running   0          4d20h
kube-system   ebs-csi-controller-ccd784cbf-ld7p5   6/6     Running   0          4d20h
kube-system   ebs-csi-controller-ccd784cbf-w24x4   6/6     Running   0          4d20h
kube-system   ebs-csi-node-j47lj                   3/3     Running   0          4d20h
kube-system   kube-proxy-8w4g7                     1/1     Running   0          4d20h
```


## 6.3 Continuous usage

Assumed roles usually expire after a certain time (e.g. 1h). The following bash function may be of help to simplify the process of renewing the session and updating kubeconfig. Add these lines to your shell's rc file.


bash shell 
```
# call example: refresh-eks-access ivu-cloud-e2x titanic-e2x
function refresh-eks-access() {
  aws-adfs login --profile $1
  aws --profile $1 eks update-kubeconfig --name $2
}
```



fish shell
```
function refresh-eks-access
  aws-adfs login --profile $argv[1]
  aws --profile $argv[1] eks update-kubeconfig --name $argv[2]
end

```

in powershell 
```
# call refresh-eks-access ivu-cloud-e20 main
function refresh-eks-access {
    param (
        [string]$Profile,
        [string]$ClusterName
    )     
	aws-adfs login --profile $Profile --no-sspi --region eu-central-1 --adfs-host adfs02.ivu-cloud.com
    aws --profile $Profile eks update-kubeconfig --name $ClusterName
}
```



The function expects two arguments: profile and cluster-name. Given what was shown in the previous paragraphs, the function may be called like this:
```
refresh-eks-access ivu-cloud-e2x titanic-e2x
refresh-eks-access ivu-cloud-e20 main
```



