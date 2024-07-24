
https://www.cnblogs.com/xzkzzz/p/9889173.html


Service Account为Pod中的进程和外部用户提供身份信息。所有的kubernetes集群中账户分为两类，Kubernetes管理的serviceaccount(服务账户)和useraccount（用户账户）。

大家都知道api server的集群的入口，对于kunbernetes的api server 是肯定不能随便访问。所以我们必须需要一些认证信息。例如：

当用户访问集群（例如使用kubectl命令）时，apiserver 会将您认证为一个特定的 User Account（目前通常是admin，除非您的系统管理员自定义了集群配置）。Pod 容器中的进程也可以与 apiserver 联系。 当它们在联系 apiserver 的时候，它们会被认证为一个特定的 Service Account。



> 来自集群外部的访问，我们可以理解成是User Account，

# 1 UserAccount

[![](https://img2018.cnblogs.com/blog/1076553/201811/1076553-20181102134841746-233102945.png)](https://img2018.cnblogs.com/blog/1076553/201811/1076553-20181102134841746-233102945.png)



因为kubernetes是高度模块化，所有认证方式和授权方式都可以通过插件的方式让客户自定义的，可以支持很多种。客户端请求的时候首先需要进行认证，认证通过后再进行授权检查，因有些操作需要级联到其他资源或者环境，但是级联环境是否有授权权限，这时候需要准入控制。



## 1.1 认证插件

- bearer token  当使用来自 http 客户端的 bearer token 时，API server 期望 `Authorization` header 中包含 `Bearer token` 的值。Bearer token 必须是一个字符串序列，只需使用 HTTP 的编码和引用功能就可以将其放入到 HTTP header 中。
- 客户端证书  客户端请求前需要，需要发送api server的办法的证书，由api server来确认是否他来签署的，引用的文件必须包含一个或多个证书颁发机构，用于验证提交给 API server 的客户端证书。如果客户端证书已提交并验证，则使用 subject 的 Common Name（CN）作为请求的用户名。反过来，api server也要验证客户端的证书，所有对于客户端也应该有一个证书，提供api server 验证，此过程是双向验证。
- HTTP BASE 认证： 通过用户名+密码方式认证。

当启用了多个认证模块时，第一个认证模块成功认证后将短路请求，不会进行第二个模块的认证。API server 不会保证认证的顺序。

## 1.2 用户账户需要哪些信息：

- user 保护UserName和UID，UserName是标识最终用户的字符串，UID是标识最终用户的字符串，比用户名更加一致切唯一。
- group 一组将用户和常规用户组相关联的字符串
- extra  包含其他有用认证信息的字符串列表


# 2 Service Account

1 user Account 和 Service Account 的区别
上述是来自集群外部的访问，我们可以理解成是User Account，是给kubernetes集群外部用户，例如（系统管理员、用户/租户等）。

Service Account而是给运行在Pod的容器、或者Pod使用的身份认证。

正常情况下，为了确保kubernetes集群的安全性，Api Server 都会给客户端进行身份认证，但是Pod访问Kubernetes Api Server服务时，也是需要身份认证的。如下图：

[![](https://img2018.cnblogs.com/blog/1076553/201811/1076553-20181102163217774-448705871.png)](https://img2018.cnblogs.com/blog/1076553/201811/1076553-20181102163217774-448705871.png)

 我们在每一个namespace下看到都有一个secret,而我们看svc的时候，api server通过svc的访问访问的，他们的Endpoints是api server。
 
```
$ kubectl  get svc 
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   52d
$ kubectl describe svc kubernetes 
Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP:                10.96.0.1
Port:              https  443/TCP
TargetPort:        6443/TCP
Endpoints:         172.16.138.40:6443,172.16.138.41:6443
Session Affinity:  None
Events:            <none>
$ kubectl get secret  -n default
NAME                  TYPE                                  DATA      AGE
default-token-lplp6   kubernetes.io/service-account-token   3         52d
$ kubectl get secret  -n ingress-nginx
NAME                                       TYPE                                  DATA      AGE
default-token-v58zx                        kubernetes.io/service-account-token   3         52d
```


每个Namespace下都有一个名为default的默认的 Service Account对象，这个Service Account里面有一个名为Tokens的可以当作Volume一样被Mount到Pod里的secret，当Pod启动时候，这个Secret会自动Mount到Pod的指定目录下，用来完成Pod中的进程访问API server时的身份认证过程。下面可以看到
```
$  kubectl describe pod my-demo
.....
Volumes:
  appindex:
    Type:          HostPath (bare host directory volume)
    Path:          /data/pod/myapp
    HostPathType:  DirectoryOrCreate
  default-token-lplp6:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-lplp6
    Optional:    false
.....
```


## 2.1 创建一个serviceAcount

```
$ kubectl create serviceaccount jaxzhai
$ kubectl get sa 
NAME      SECRETS   AGE
default   1         52d
jaxzhai   1         8s
$ kubectl describe sa jaxzhai
Name:                jaxzhai
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   jaxzhai-token-9n5th
Tokens:              jaxzhai-token-9n5th
Events:              <none>  
$ kubectl get secret   
NAME                  TYPE                                  DATA      AGE  
default-token-lplp6   kubernetes.io/service-account-token   3         52d  
jaxzhai-token-9n5th   kubernetes.io/service-account-token   3         4m
```


这里我们看到Kubernetes集群会自动创建一个token的secert，并被jaxzhai这个serviceaccount引用。

设置非默认的 service account，只需要在 pod 的`spec.serviceAccountName` 字段中将name设置为您想要用的 service account 名字即可。

在 pod 创建之初 service account 就必须已经存在，否则创建将被拒绝。

您不能更新已创建的 pod 的 service account。


---
Terraform 中创建 ServiceAccount

```
#### _______________________________________________________________ Lookup AWS resources _____ ####

locals {
  resource_prefix                           = "eks-${var.cluster_name}"
  efs_file_system_tags                      = var.efs_file_system_id == null? { Name = local.resource_prefix } : null
  vpc_tags                                  = var.vpc_id == null? { Name = local.resource_prefix } : null
  aws_load_balancer_controller_iam_role_arn = (
    var.aws_load_balancer_controller_iam_role_arn == null? 
    data.aws_iam_role.aws_load_balancer_controller[0].arn : 
    var.aws_load_balancer_controller_iam_role_arn
  )
}


data "aws_iam_role" "loki_s3" {
  count = var.configure_loki_buckets? 1 : 0

  name = "${local.resource_prefix}-loki-s3"
}

#### _______________________________________________________________ Kubernetes resources _____ ####

locals {
  loki_iam_role_arn = var.configure_loki_buckets? data.aws_iam_role.loki_s3[0].arn : ""

  loki_namespace_manifest = <<-EOF
    kind: Namespace
    apiVersion: v1
    metadata:
      name: loki
      labels:
        kubernetes.io/metadata.name: loki
  EOF

  loki_service_account_manifest = <<-EOF
    kind: ServiceAccount
    apiVersion: v1
    metadata:
      name: loki-sa
      namespace: loki
      labels:
        app.kubernetes.io/instance: loki-stack
        app.kubernetes.io/name: loki
      annotations:
        eks.amazonaws.com/role-arn: ${local.loki_iam_role_arn}
  EOF

}

#### _______________________________________________________________ Create resources _____ ####
resource "kubernetes_manifest" "loki_namespace" {
  count = var.configure_loki_buckets? 1 : 0

  manifest = yamldecode(local.loki_namespace_manifest)
}

resource "kubernetes_manifest" "loki_service_account" {
  count = var.configure_loki_buckets? 1 : 0

  manifest = yamldecode(local.loki_service_account_manifest)

  depends_on = [
    kubernetes_manifest.loki_namespace
  ]
}




```

## 2.2 在pod中使用service account

```
apiVersion: v1
kind: Pod
metadata:
  name: my-sa-demo
  namespace: default
  labels:
    name: myapp
    tier: appfront
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    ports:
    - name: http
      containerPort: 80
  serviceAccountName: jaxzhai

$ kubectl apply -f myapp-serviceaccount.yaml 
pod/my-sa-demo created
$ kubectl describe pod my-sa-demo 
......
Volumes:
  jaxzhai-token-9n5th:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  jaxzhai-token-9n5th
    Optional:    false
QoS Class:       BestEffort
......
```

