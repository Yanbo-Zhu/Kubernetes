
https://www.cnblogs.com/xzkzzz/p/9889173.html


# 1 user Account 和 Service Account 的区别
所有的kubernetes集群中账户分为两类，Kubernetes管理的serviceaccount(服务账户)和useraccount（用户账户）。
大家都知道api server的集群的入口，对于kunbernetes的api server 是肯定不能随便访问。所以我们必须需要一些认证信息。例如：

上述是来自集群外部的访问，我们可以理解成是User Account，是给kubernetes集群外部用户，例如（系统管理员、用户/租户等）。

Service Account而是给运行在Pod的容器、或者Pod使用的身份认证。

# 2 UserAccount

> 来自集群外部的访问，我们可以理解成是User Account，

当用户访问集群（例如使用kubectl命令）时，apiserver 会将您认证为一个特定的 User Account（目前通常是admin，除非您的系统管理员自定义了集群配置）。Pod 容器中的进程也可以与 apiserver 联系。 当它们在联系 apiserver 的时候，它们会被认证为一个特定的 Service Account。


[![](https://img2018.cnblogs.com/blog/1076553/201811/1076553-20181102134841746-233102945.png)](https://img2018.cnblogs.com/blog/1076553/201811/1076553-20181102134841746-233102945.png)

因为kubernetes是高度模块化，所有认证方式和授权方式都可以通过插件的方式让客户自定义的，可以支持很多种。客户端请求的时候首先需要进行认证，认证通过后再进行授权检查，因有些操作需要级联到其他资源或者环境，但是级联环境是否有授权权限，这时候需要准入控制。

## 2.1 认证插件

- bearer token  当使用来自 http 客户端的 bearer token 时，API server 期望 `Authorization` header 中包含 `Bearer token` 的值。Bearer token 必须是一个字符串序列，只需使用 HTTP 的编码和引用功能就可以将其放入到 HTTP header 中。
- 客户端证书  客户端请求前需要，需要发送api server的办法的证书，由api server来确认是否他来签署的，引用的文件必须包含一个或多个证书颁发机构，用于验证提交给 API server 的客户端证书。如果客户端证书已提交并验证，则使用 subject 的 Common Name（CN）作为请求的用户名。反过来，api server也要验证客户端的证书，所有对于客户端也应该有一个证书，提供api server 验证，此过程是双向验证。
- HTTP BASE 认证： 通过用户名+密码方式认证。

当启用了多个认证模块时，第一个认证模块成功认证后将短路请求，不会进行第二个模块的认证。API server 不会保证认证的顺序。

## 2.2 用户账户需要哪些信息：

- user 保护UserName和UID，UserName是标识最终用户的字符串，UID是标识最终用户的字符串，比用户名更加一致切唯一。
- group 一组将用户和常规用户组相关联的字符串
- extra  包含其他有用认证信息的字符串列表



# 3 Service Account

Whenever you access your Kubernetes cluster with kubectl, you are authenticated by Kubernetes with your user account. User accounts are meant to be used by humans. But when a pod running in the cluster wants to access the Kubernetes API server, it needs to use a service account instead. Service accounts are just like user accounts but for non-humans.

Service Account为Pod中的进程和外部用户提供身份信息。

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

## 3.1 Why Do Kubernetes Service Accounts Exist?

Why do Kubernetes Service Accounts exist? The simple answer is because pods are not humans, so it's good to have a distinction from user accounts. It's especially important for security reasons. Also, once you start using an external user management system with Kubernetes, it becomes even more important since all your users will probably follow typical [firstname.lastname@your-company.com](mailto:firstname.lastname@your-company.com) usernames.

But, you may wonder, why would pods inside the Kubernetes cluster need to connect to the Kubernetes API at all? Well, there are multiple use cases for it. The most common one is when you have a CI/CD pipeline agent deploying your applications to the same cluster. Many cloud-native tools also need access to your Kubernetes API to do their jobs, such as logging or monitoring applications.

> CI/CD pipeline agent 部署应用到 cluster 中 ， CI/CD pipeline agent 需要一个 serviceAccount，用这个 ServiceAcoount 和 Kubernetes API server 交流， 这个 serviceAccount 已经有权利去assume a role 了 ， 这个role 有一些permission 

Service accounts are extremely useful. They provide a way for your pods to access the Kubernetes API. Many Kubernetes-native tools rely on this mechanism. Therefore, it's good to know what service accounts are and how they access the Kubernetes API.
However, you also need to be careful because a misconfigured service account can be a security risk. If, for example, to save time, you decide to increase the permission for a default service account (instead of creating a new one), you'll make it possible for any pod on the cluster to access the Kubernetes API. If that access is read-only, it can lead to data exposure, and if it's read-write, the damage can be even great. Fortunately, as you learned in this post, creating a service account is easy.


## 3.2 Default Service Account

That's because Kubernetes comes with a predefined service account called "default."

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



给pod需要配上serviceAccount意味着什么: 
So, it turns out that your pods have the default service account assigned even when you don't ask for it. This is because every pod in the cluster needs to have one (and only one) service account assigned to it. What can your pod do with that service account? Well, pretty much nothing. That default service account doesn't have any permissions assigned to it. (名字为 Default 的serviceAccount 其实并没有任何permission )

去验证 名字为 Default 的serviceAccount 其实并没有任何permission
We can validate that as well. Let's get into our freshly deployed nginx pod and try to connect to a Kubernetes API from there. For that, we'll need to export a few environment variables and then use the **curl** command to send an HTTP request to the Kubernetes API.

```
# Export the internal Kubernetes API server hostname $ APISERVER=https://kubernetes.default.svc # Export the path to ServiceAccount mount directory $ SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount # Read the ServiceAccount bearer token $ TOKEN=$(cat ${SERVICEACCOUNT}/token) # Reference the internal Kubernetes certificate authority (CA) $ CACERT=${SERVICEACCOUNT}/ca.crt # Make a call to the Kubernetes API with TOKEN $ curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/default/pods { "kind": "Status", "apiVersion": "v1", "metadata": {}, "status": "Failure", "message": "pods is forbidden: User \"system:serviceaccount:default:default\" cannot list resource \"pods\" in API group \"\" in the namespace \"default\"", "reason": "Forbidden", "details": { "kind": "pods" }, "code": 403 }# 
```

As you can see, the default service account indeed doesn't have many permissions. It's really only there to fulfil the requirement that each pod has a service account assigned to it.




## 3.3 创建一个serviceAcount


So, if you want your pod to actually be able to talk to the Kubernetes API and do something, you have two options. You either need to assign some permissions to the default service account, or you need to create a new service account. The first option is not recommended. In fact, you shouldn't use the default service account for anything. Let's choose the recommended option then, which is creating dedicated service accounts. It's also worth mentioning here that, just like with user access, you should create separate service accounts for separate needs.

### 3.3.1 例子1

The easiest way to create a service account is by executing the **kubectl create serviceaccount** command followed by a desired service account name.

```shell
$ kubectl create serviceaccount nginx-serviceaccount serviceaccount/nginx-serviceaccount created 
```

Just like with anything else in Kubernetes, it's worth knowing how to create one using the YAML definition file. In the case of service accounts, it's actually really simple and looks like this:

```makefile
apiVersion: v1 
kind: ServiceAccount 
metadata: name: nginx-serviceaccount 
```

I'll save it as nginx-sa.yaml and apply that simple YAML file using **kubectl apply -f nginx-sa.yaml:**

```shell
$ kubectl apply -f nginx-sa.yaml serviceaccount/nginx-serviceaccount created 
```

You can see from the output above that a service account was created, but you can double-check that with **kubectl get serviceaccounts,** or **kubectl get as** for short.

```csharp
$ kubectl get serviceaccounts NAME SECRETS AGE default 1 3h14m nginx-serviceaccount 1 72s $
```



### 3.3.2 例子2 
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



### 3.3.3 Terraform 中创建 ServiceAccount

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


## 3.4 Assigning Permissions to a Service Account

OK, you created a new service account for your pod, but by default, it won't do much more than the default service account (called "default") that you saw previously. To change that, you can use the standard Kubernetes role-based access control mechanism. This means that you can either use existing role or create a new one (or ClusterRole) and then use RoleBinding to bind a role with your new ServiceAccount.

For the purpose of the demo, you can assign a built-in Kubernetes ClusterRole called "view" that allows viewing all resources. You then need to create a RoleBinding for your Service Account.

```csharp
kubectl create rolebinding nginx-sa-readonly \ --clusterrole=view \ --serviceaccount=default:nginx-serviceaccount \ --namespace=default 
```

Now, any pod using the new ServiceAccount should be able to view all resources in the default namespace. To validate that, you can perform the same test you did earlier, but first you need to assign that ServiceAccount to your nginx pod.


## 3.5 Specifying ServiceAccount For Your Pod


### 3.5.1 例子1

As mentioned previously, if you don't specify any service account for your pod, it will be assigned a "default" service account. You just created a new service account for your needs, so you'll want to use that one instead. For that, you need to pass the service account name as the value for serviceAccountName key in a spec section of your deployment definition file.

```shell
apiVersion: apps/v1 kind: Deployment metadata: name: nginx1 labels: app: nginx1 spec: replicas: 2 selector: matchLabels: app: nginx1 template: metadata: labels: app: nginx1 spec: serviceAccountName: nginx-serviceaccount containers: - name: nginx1 image: nginx ports: - containerPort: 80 
```


That's it. Now, after applying this definition, you can try to perform the same test as before from within the pod.

```shell
$ curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/default/pods { "kind": "PodList", "apiVersion": "v1", "metadata": { "resourceVersion": "52233" }, "items": [ { "metadata": { "name": "nginx1-65448895f9-5j6b6", "generateName": "nginx1-65448895f9-", "namespace": "default", "uid": "b09bfa93-a388-4cd9-9495-131f620613d0", "resourceVersion": "49536", (...) 
```

And as expected, now it works fine. If, for any reason, your pods need access to a Kubernetes API, you create a new ServiceAccount for it, then assign a role or ClusterRole to it via RoleBinding, then specify the ServiceAccount name in your deployment.


### 3.5.2 例子2

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
  serviceAccountName: jaxzhai  # 这里 

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

