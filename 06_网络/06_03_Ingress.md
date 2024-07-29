
# 1 基础

其实 是 基于Nginx 的 controller  

Ingress controller（Ingress 控制器）是面向 Kubernetes（及其他容器化）环境的专用负载均衡器。Kubernetes 是容器化应用管理的事实标准。对许多企业而言，将生产工作负载迁移到 Kubernetes 会增加应用流量管理方面的挑战和复杂性。Ingress controller 能够将 Kubernetes 应用流量路由的复杂性抽象出来，并在 Kubernetes 服务和外部服务之间建立了一座桥梁。

Kubernetes Ingress Controller 的功能如下：

    接受来自 Kubernetes 平台外部的流量，并将其负载均衡到 Kubernetes 平台内部运行的 pod（容器）
    可管理集群内需要与集群外其他服务通信的服务的出向流量
    使用 Kubernetes API 进行配置，以部署名为“Ingress 资源”的对象
    监控 Kubernetes 中运行的 pod，并在服务添加或删除 pod 后自动更新负载均衡规则


您是否经常更改 Ingress controller 的配置？保护 Kubernetes 的 service 免受攻击是否是首要任务？如果是，那么您需要一个具有以下功能的生产级 Ingress controller：

- [**动态重新配置**](https://www.nginx-cn.net/products/nginx/load-balancing#load-balancing-api)
- 轻量级现代 [**Web 应用防火墙**](https://www.nginx-cn.net/products/nginx-app-protect/) (WAF)
- 使用久经考验的标准[**单点登录（SSO）解决方案**](https://docs.nginx.com/nginx/deployment-guides/single-sign-on/)（如 OpenID Connect (OIDC)）进行授权和身份验证
- 24/7 全天候专业[**支持**](https://www.nginx-cn.net/support/)，可帮助您将生产工作负载快速迁移到 Kubernetes

[**NGINX Ingress Controller**](https://www.nginx-cn.net/products/nginx-ingress-controller/) 是生产级 Ingress controller，可作为守护进程在 Kubernetes 环境中与 [**NGINX 开源版**](https://nginx.org/en)或 [**NGINX Plus**](https://www.nginx-cn.net/products/nginx/) 实例一同运行。该守护进程负责监控 [**NGINX Ingress 资源**](https://www.nginx-cn.net/products/nginx-ingress-controller/nginx-ingress-resources/)和 [**Kubernetes Ingress 资源**](https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource)，以识别需要进行入向负载均衡的服务请求。并且它还能够兼容我们的轻量级现代 WAF — [**NGINX App Protect**](https://www.nginx-cn.net/products/nginx-app-protect/)（可部署在 Ingress Controller 上作为 per-service 代理和 per-pod 代理）。

借助 NGINX Ingress Controller，您可以在四层至七层充分利用 Kubernetes 网络，从而增强 Kubernetes 的 service 之间的安全性和流量控制。


# 2 为什么需要 Ingress ？

- Service 可以使用 NodePort 暴露集群外访问端口，但是性能低、不安全并且端口的范围有限。
- Service 缺少七层（OSI 网络模型）的统一访问入口，负载均衡能力很低，不能做限流、验证非法用户、链路追踪等等。
- Ingress 公开了从集群外部到集群内 `服务` 的 HTTP 和 HTTPS 路由。流量路由由 Ingress 资源上定义的规则控制。
- 我们使用 Ingress 作为整个集群统一的入口，配置 Ingress 规则转发到对应的 Service 。

![22.png](https://cdn.nlark.com/yuque/0/2022/png/513185/1648106820246-30eb8f18-0886-4934-a0cd-1cdc1e114bbe.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_38%2Ctext_6K645aSn5LuZ%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fformat%2Cwebp)


# 3 Ingress介绍

在前面课程中已经提到，Service对集群之外暴露服务的主要方式有两种：NotePort和LoadBalancer，但是这两种方式，都有一定的缺点：
- NodePort方式的缺点是会占用很多集群机器的端口，那么当集群服务变多的时候，这个缺点就愈发明显
- LB方式的缺点是每个service需要一个LB，浪费、麻烦，并且需要kubernetes之外设备的支持

基于这种现状，kubernetes提供了Ingress资源对象，Ingress只需要一个NodePort或者一个LB就可以满足暴露多个Service的需求。工作机制大致如下图表示：

![img](https://gitee.com/yooome/golang/raw/main/22-k8s%E8%AF%A6%E7%BB%86%E6%95%99%E7%A8%8B-%E8%B0%83%E6%95%B4%E7%89%88/Kubenetes.assets/image-20200623092808049.png)

实际上，Ingress相当于一个7层的负载均衡器，是kubernetes对反向代理的一个抽象，它的工作原理类似于Nginx，可以理解成在**Ingress里建立诸多映射规则，Ingress Controller通过监听这些配置规则并转化成Nginx的反向代理配置 , 然后对外部提供服务**。在这里有两个核心概念：
- ingress：kubernetes中的一个对象，作用是定义请求如何转发到service的规则
- ingress controller：具体实现反向代理及负载均衡的程序，对ingress定义的规则进行解析，根据配置的规则来实现请求转发，实现方式有很多，比如Nginx, Contour, Haproxy等等

Ingress（以Nginx为例）的工作原理如下：
1. 用户编写Ingress规则，说明哪个域名对应kubernetes集群中的哪个Service
2. Ingress控制器动态感知Ingress服务规则的变化，然后生成一段对应的Nginx反向代理配置
3. Ingress控制器会将生成的Nginx配置写入到一个运行着的Nginx服务中，并动态更新
4. 到此为止，其实真正在工作的就是一个Nginx了，内部配置了用户定义的请求转发规则

![img](https://gitee.com/yooome/golang/raw/main/22-k8s%E8%AF%A6%E7%BB%86%E6%95%99%E7%A8%8B-%E8%B0%83%E6%95%B4%E7%89%88/Kubenetes.assets/image-20200516112704764.png)



# 4 Ingress nginx 和 nginx ingress
  
## 4.1 nginx Ingress（不用）

- nginx Ingress是 Nginx 官网适配 k8s 的，分为开源版和 nginx plus 版（收费）。
- [官网地址](https://www.nginx.com/products/nginx-ingress-controller)。  

![](https://cdn.nlark.com/yuque/0/2022/png/513185/1648106831619-16f9a56c-d3cd-43b0-b96b-2d23a0da8cfe.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_40%2Ctext_6K645aSn5LuZ%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

  

## 4.2 ingress nginx
  
- ingress nginx  是 k8s 官方出品的，会及时更新一些特性，性能很高，被广泛采用。
- [官网地址](https://kubernetes.io/zh/docs/concepts/services-networking/ingress/#ingress-%E6%98%AF%E4%BB%80%E4%B9%88)。

注意：以后如果不特别说明，ingress = ingress nginx 。


# 5 Ingress 安装 
●  自建集群采用裸金属安装方式。 
●  下载： wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.2/deploy/static/provider/baremetal/deploy.yaml

● 需要做如下修改： 
  ○ ① 修改 k8s.gcr.io/ingress-nginx/controller:v1.1.2 镜像为 registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v1.1.2 。
  ○ ② 修改 k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1 镜像为 registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.1.1。
  ○ ③ 修改 Service 为 ClusterIP，无需 NodePort 模式。
  ○ ④ 修改 Deployment 为 DaemonSet 。
  ○ ⑤ 修改 Container 使用主机网络，直接在主机上开辟 80 、443 端口，无需中间解析，速度更快，所有各个节点的 80 和 443 端口不能被其它进程占用。
  ○ ⑥ Container 使用主机网络，对应的 dnsPolicy 策略也需要改为主机网络的。
  ○ ⑦ 修改 DaemonSet 的 nodeSelector: ingress-node=true 。这样只需要给 node 节点打上 ingress-node=true 标签，即可快速的加入或剔除 ingress-controller的数量


创建文件  vi deploy.yaml


deploy.yaml 修改的内容如下

```
#GENERATED FOR K8S 1.20
apiVersion: v1
kind: Namespace
metadata:
  labels:
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  name: ingress-nginx
---
apiVersion: v1
automountServiceAccountToken: true
kind: ServiceAccount
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx
  namespace: ingress-nginx
---
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    app.kubernetes.io/component: admission-webhook
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-admission
  namespace: ingress-nginx
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx
  namespace: ingress-nginx
rules:
- apiGroups:
  - ""
  resources:
  - namespaces
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - configmaps
  - pods
  - secrets
  - endpoints
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - networking.k8s.io
  resources:
  - ingresses
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - networking.k8s.io
  resources:
  - ingresses/status
  verbs:
  - update
- apiGroups:
  - networking.k8s.io
  resources:
  - ingressclasses
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resourceNames:
  - ingress-controller-leader
  resources:
  - configmaps
  verbs:
  - get
  - update
- apiGroups:
  - ""
  resources:
  - configmaps
  verbs:
  - create
- apiGroups:
  - ""
  resources:
  - events
  verbs:
  - create
  - patch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  annotations:
    helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    app.kubernetes.io/component: admission-webhook
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-admission
  namespace: ingress-nginx
rules:
- apiGroups:
  - ""
  resources:
  - secrets
  verbs:
  - get
  - create
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx
rules:
- apiGroups:
  - ""
  resources:
  - configmaps
  - endpoints
  - nodes
  - pods
  - secrets
  - namespaces
  verbs:
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - networking.k8s.io
  resources:
  - ingresses
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - events
  verbs:
  - create
  - patch
- apiGroups:
  - networking.k8s.io
  resources:
  - ingresses/status
  verbs:
  - update
- apiGroups:
  - networking.k8s.io
  resources:
  - ingressclasses
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    app.kubernetes.io/component: admission-webhook
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-admission
rules:
- apiGroups:
  - admissionregistration.k8s.io
  resources:
  - validatingwebhookconfigurations
  verbs:
  - get
  - update
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx
  namespace: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: ingress-nginx
subjects:
- kind: ServiceAccount
  name: ingress-nginx
  namespace: ingress-nginx
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  annotations:
    helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    app.kubernetes.io/component: admission-webhook
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-admission
  namespace: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: ingress-nginx-admission
subjects:
- kind: ServiceAccount
  name: ingress-nginx-admission
  namespace: ingress-nginx
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: ingress-nginx
subjects:
- kind: ServiceAccount
  name: ingress-nginx
  namespace: ingress-nginx
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    helm.sh/hook: pre-install,pre-upgrade,post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    app.kubernetes.io/component: admission-webhook
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-admission
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: ingress-nginx-admission
subjects:
- kind: ServiceAccount
  name: ingress-nginx-admission
  namespace: ingress-nginx
---
apiVersion: v1
data:
  allow-snippet-annotations: "true"
kind: ConfigMap
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-controller
  namespace: ingress-nginx
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - appProtocol: http
    name: http
    port: 80
    protocol: TCP
    targetPort: http
  - appProtocol: https
    name: https
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  type: ClusterIP # NodePort
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-controller-admission
  namespace: ingress-nginx
spec:
  ports:
  - appProtocol: https
    name: https-webhook
    port: 443
    targetPort: webhook
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  type: ClusterIP
---
apiVersion: apps/v1
kind: DaemonSet # Deployment
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  minReadySeconds: 0
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app.kubernetes.io/component: controller
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/name: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/component: controller
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/name: ingress-nginx
    spec:
      dnsPolicy: ClusterFirstWithHostNet # dns 调整为主机网络 ，原先为 ClusterFirst
      hostNetwork: true # 直接让 nginx 占用本机的 80 和 443 端口，这样就可以使用主机网络
      containers:
      - args:
        - /nginx-ingress-controller
        - --election-id=ingress-controller-leader
        - --controller-class=k8s.io/ingress-nginx
        - --ingress-class=nginx
        - --report-node-internal-ip-address=true
        - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
        - --validating-webhook=:8443
        - --validating-webhook-certificate=/usr/local/certificates/cert
        - --validating-webhook-key=/usr/local/certificates/key
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: LD_PRELOAD
          value: /usr/local/lib/libmimalloc.so
        image: registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v1.1.2 # 修改 k8s.gcr.io/ingress-nginx/controller:v1.1.2@sha256:28b11ce69e57843de44e3db6413e98d09de0f6688e33d4bd384002a44f78405c
        imagePullPolicy: IfNotPresent
        lifecycle:
          preStop:
            exec:
              command:
              - /wait-shutdown
        livenessProbe:
          failureThreshold: 5
          httpGet:
            path: /healthz
            port: 10254
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        name: controller
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
        - containerPort: 443
          name: https
          protocol: TCP
        - containerPort: 8443
          name: webhook
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /healthz
            port: 10254
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources: # 资源限制
          requests:
            cpu: 100m
            memory: 90Mi
          limits: 
            cpu: 500m
            memory: 500Mi     
        securityContext:
          allowPrivilegeEscalation: true
          capabilities:
            add:
            - NET_BIND_SERVICE
            drop:
            - ALL
          runAsUser: 101
        volumeMounts:
        - mountPath: /usr/local/certificates/
          name: webhook-cert
          readOnly: true
      nodeSelector:
        node-role: ingress # 以后只需要给某个 node 打上这个标签就可以部署 ingress-nginx 到这个节点上了
        # kubernetes.io/os: linux
      serviceAccountName: ingress-nginx
      terminationGracePeriodSeconds: 300
      volumes:
      - name: webhook-cert
        secret:
          secretName: ingress-nginx-admission
---
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    helm.sh/hook: pre-install,pre-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    app.kubernetes.io/component: admission-webhook
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-admission-create
  namespace: ingress-nginx
spec:
  template:
    metadata:
      labels:
        app.kubernetes.io/component: admission-webhook
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
        app.kubernetes.io/version: 1.1.2
        helm.sh/chart: ingress-nginx-4.0.18
      name: ingress-nginx-admission-create
    spec:
      containers:
      - args:
        - create
        - --host=ingress-nginx-controller-admission,ingress-nginx-controller-admission.$(POD_NAMESPACE).svc
        - --namespace=$(POD_NAMESPACE)
        - --secret-name=ingress-nginx-admission
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        image: registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.1.1 # k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1@sha256:64d8c73dca984af206adf9d6d7e46aa550362b1d7a01f3a0a91b20cc67868660
        imagePullPolicy: IfNotPresent
        name: create
        securityContext:
          allowPrivilegeEscalation: false
      nodeSelector:
        kubernetes.io/os: linux
      restartPolicy: OnFailure
      securityContext:
        fsGroup: 2000
        runAsNonRoot: true
        runAsUser: 2000
      serviceAccountName: ingress-nginx-admission
---
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    helm.sh/hook: post-install,post-upgrade
    helm.sh/hook-delete-policy: before-hook-creation,hook-succeeded
  labels:
    app.kubernetes.io/component: admission-webhook
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-admission-patch
  namespace: ingress-nginx
spec:
  template:
    metadata:
      labels:
        app.kubernetes.io/component: admission-webhook
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
        app.kubernetes.io/version: 1.1.2
        helm.sh/chart: ingress-nginx-4.0.18
      name: ingress-nginx-admission-patch
    spec:
      containers:
      - args:
        - patch
        - --webhook-name=ingress-nginx-admission
        - --namespace=$(POD_NAMESPACE)
        - --patch-mutating=false
        - --secret-name=ingress-nginx-admission
        - --patch-failure-policy=Fail
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        image: registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.1.1 # k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1@sha256:64d8c73dca984af206adf9d6d7e46aa550362b1d7a01f3a0a91b20cc67868660
        imagePullPolicy: IfNotPresent
        name: patch
        securityContext:
          allowPrivilegeEscalation: false
      nodeSelector:
        kubernetes.io/os: linux
      restartPolicy: OnFailure
      securityContext:
        fsGroup: 2000
        runAsNonRoot: true
        runAsUser: 2000
      serviceAccountName: ingress-nginx-admission
---
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: nginx
spec:
  controller: k8s.io/ingress-nginx
---
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  labels:
    app.kubernetes.io/component: admission-webhook
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-admission
webhooks:
- admissionReviewVersions:
  - v1
  clientConfig:
    service:
      name: ingress-nginx-controller-admission
      namespace: ingress-nginx
      path: /networking/v1/ingresses
  failurePolicy: Fail
  matchPolicy: Equivalent
  name: validate.nginx.ingress.kubernetes.io
  rules:
  - apiGroups:
    - networking.k8s.io
    apiVersions:
    - v1
    operations:
    - CREATE
    - UPDATE
    resources:
    - ingresses
  sideEffects: None
```


给 Node 节点打标签
kubectl label node k8s-node1 node-role=ingress
kubectl label node k8s-node2 node-role=ingress

当然，也可以给 Master 节点打标签，但是 kubeadm 安装 k8s 集群的时候，会给 Master 节点打上污点，即使我们打上标签，也不会进行 Pod 的调度；换言之，Ingress 也不会在 Master 节点上安装，后面再讲。


安装 Ingress （需要关闭 Node 节点的 80 和 443 端口，不能有其他进程占用）：
kubectl apply -f deploy.yaml


验证是否安装成功：只需要在部署了 ingress 的主机，执行如下的命令：
netstat -ntlp | grep 80
netstat -ntlp | grep 443


卸载：
kubectl delete -f deploy.yaml
\


# 6 Ingress 工作原理

![26.png](https://cdn.nlark.com/yuque/0/2022/png/513185/1648106871470-c8097a0a-4a6c-4ce1-a5b5-0898641801ca.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_22%2Ctext_6K645aSn5LuZ%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fformat%2Cwebp%2Fresize%2Cw_777%2Climit_0)



● 工作原理： 
  ○ ① 用户编写 Ingress 规则，说明哪个域名对应 k8s 集群中的哪个 Service。
  ○ ② Ingress 控制器动态感知 Ingress 服务规则的变化，然后生成一段对应的 Nginx 的反向代理配置。
  ○ ③ Ingress 控制器会将生成的 Nginx 配置写入到一个运行着的 Nginx 服务中，并动态更新。




# 7 Ingress使用

## 7.1 环境准备 搭建ingress环境

```
# 创建文件夹
[root@k8s-master01 ~]# mkdir ingress-controller
[root@k8s-master01 ~]# cd ingress-controller/
# 获取ingress-nginx，本次案例使用的是0.30版本
[root@k8s-master01 ingress-controller]# wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml
[root@k8s-master01 ingress-controller]# wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/provider/baremetal/service-nodeport.yaml
# 修改mandatory.yaml文件中的仓库
# 修改quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.30.0
# 为quay-mirror.qiniu.com/kubernetes-ingress-controller/nginx-ingress-controller:0.30.0
# 创建ingress-nginx
[root@k8s-master01 ingress-controller]# kubectl apply -f ./
# 查看ingress-nginx
[root@k8s-master01 ingress-controller]# kubectl get pod -n ingress-nginx
NAME                                           READY   STATUS    RESTARTS   AGE
pod/nginx-ingress-controller-fbf967dd5-4qpbp   1/1     Running   0          12h
# 查看service
[root@k8s-master01 ingress-controller]# kubectl get svc -n ingress-nginx
NAME            TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx   NodePort   10.98.75.163   <none>        80:32240/TCP,443:31335/TCP   11h
```


![27.png](https://cdn.nlark.com/yuque/0/2022/png/513185/1648106886022-39954bc4-40e9-48bb-a227-9894e6e10963.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_31%2Ctext_6K645aSn5LuZ%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fformat%2Cwebp)


## 7.2 准备service和pod

为了后面的实验比较方便，创建如下图所示的模型

![img](https://gitee.com/yooome/golang/raw/main/22-k8s%E8%AF%A6%E7%BB%86%E6%95%99%E7%A8%8B-%E8%B0%83%E6%95%B4%E7%89%88/Kubenetes.assets/image-20200516102419998.png)

创建tomcat-nginx.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.1
        ports:
        - containerPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: tomcat-pod
  template:
    metadata:
      labels:
        app: tomcat-pod
    spec:
      containers:
      - name: tomcat
        image: tomcat:8.5-jre10-slim
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: dev
spec:
  selector:
    app: nginx-pod
  clusterIP: None
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: tomcat-service
  namespace: dev
spec:
  selector:
    app: tomcat-pod
  clusterIP: None
  type: ClusterIP
  ports:
  - port: 8080
    targetPort: 8080
```


```
# 创建
[root@k8s-master01 ~]# kubectl create -f tomcat-nginx.yaml

# 查看
[root@k8s-master01 ~]# kubectl get svc -n dev
NAME             TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
nginx-service    ClusterIP   None         <none>        80/TCP     48s
tomcat-service   ClusterIP   None         <none>        8080/TCP   48s
```


## 7.3 Http代理

创建ingress-http.yaml

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-http
  namespace: dev
spec:
  rules:
  - host: nginx.itheima.com
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx-service
          servicePort: 80
  - host: tomcat.itheima.com
    http:
      paths:
      - path: /
        backend:
          serviceName: tomcat-service
          servicePort: 8080
```

```
# 创建
[root@k8s-master01 ~]# kubectl create -f ingress-http.yaml
ingress.extensions/ingress-http created

# 查看
[root@k8s-master01 ~]# kubectl get ing ingress-http -n dev
NAME           HOSTS                                  ADDRESS   PORTS   AGE
ingress-http   nginx.itheima.com,tomcat.itheima.com             80      22s

# 查看详情
[root@k8s-master01 ~]# kubectl describe ing ingress-http  -n dev
...
Rules:
Host                Path  Backends
----                ----  --------
nginx.itheima.com   / nginx-service:80 (10.244.1.96:80,10.244.1.97:80,10.244.2.112:80)
tomcat.itheima.com  / tomcat-service:8080(10.244.1.94:8080,10.244.1.95:8080,10.244.2.111:8080)
...

# 接下来,在本地电脑上配置host文件,解析上面的两个域名到192.168.109.100(master)上
# 然后,就可以分别访问tomcat.itheima.com:32240  和  nginx.itheima.com:32240 查看效果了
```


## 7.4 Https代理

创建证书

```
# 生成证书
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/C=CN/ST=BJ/L=BJ/O=nginx/CN=itheima.com"
# 创建密钥
kubectl create secret tls tls-secret --key tls.key --cert tls.crt

```


创建ingress-https.yaml

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-https
  namespace: dev
spec:
  tls:
    - hosts:
      - nginx.itheima.com
      - tomcat.itheima.com
      secretName: tls-secret # 指定秘钥
  rules:
  - host: nginx.itheima.com
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx-service
          servicePort: 80
  - host: tomcat.itheima.com
    http:
      paths:
      - path: /
        backend:
          serviceName: tomcat-service
          servicePort: 8080
```


```
# 创建
[root@k8s-master01 ~]# kubectl create -f ingress-https.yaml
ingress.extensions/ingress-https created

# 查看
[root@k8s-master01 ~]# kubectl get ing ingress-https -n dev
NAME            HOSTS                                  ADDRESS         PORTS     AGE
ingress-https   nginx.itheima.com,tomcat.itheima.com   10.104.184.38   80, 443   2m42s

# 查看详情
[root@k8s-master01 ~]# kubectl describe ing ingress-https -n dev
...
TLS:
  tls-secret terminates nginx.itheima.com,tomcat.itheima.com
Rules:
Host              Path Backends
----              ---- --------
nginx.itheima.com  /  nginx-service:80 (10.244.1.97:80,10.244.1.98:80,10.244.2.119:80)
tomcat.itheima.com /  tomcat-service:8080(10.244.1.99:8080,10.244.2.117:8080,10.244.2.120:8080)
...

# 下面可以通过浏览器访问https://nginx.itheima.com:31335 和 https://tomcat.itheima.com:31335来查看了
```



# 8 实战


## 8.1 基本配置


语法

```
apiVersion: networking.k8s.io/v1
kind: Ingress 
metadata:
  name: ingress-http
  namespace: default
  annotations: 
    kubernetes.io/ingress.class: "nginx"
spec:
  rules: # 规则
  - host: it.nginx.com # 指定的监听的主机域名，相当于 nginx.conf 的 server { xxx }
    http: # 指定路由规则
      paths:
      - path: /
        pathType: Prefix # 匹配规则，Prefix 前缀匹配 it.nginx.com/* 都可以匹配到
        backend: # 指定路由的后台服务的 service 名称
          service:
            name: nginx-svc # 服务名
            port:
              number: 80 # 服务的端口
```


● pathType 说明： 
  ○ Prefix：基于以 / 分隔的 URL 路径前缀匹配。匹配区分大小写，并且对路径中的元素逐个完成。 路径元素指的是由 / 分隔符分隔的路径中的标签列表。 如果每个 p 都是请求路径 p 的元素前缀，则请求与路径 p 匹配。
  ○ Exact：精确匹配 URL 路径，且区分大小写。
  ○ ImplementationSpecific：对于这种路径类型，匹配方法取决于 IngressClass。 具体实现可以将其作为单独的 pathType 处理或者与 Prefix 或 Exact 类型作相同处理。

注意：ingress 规则会生效到所有安装了 IngressController 的机器的 nginx 配置。



### 8.1.1 示例


vi k8s-ingress.yaml

```
apiVersion: networking.k8s.io/v1
kind: Ingress 
metadata:
  name: ingress-http
  namespace: default
  annotations: 
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"  
spec:
  rules: # 规则
  - host: nginx.xudaxian.com # 指定的监听的主机域名，相当于 nginx.conf 的 server { xxx }
    http: # 指定路由规则
      paths:
      - path: /
        pathType: Prefix # 匹配规则，Prefix 前缀匹配 nginx.xudaxian.com/* 都可以匹配到
        backend: # 指定路由的后台服务的 service 名称
          service:
            name: nginx-svc # 服务名
            port:
              number: 80 # 服务的端口
  - host: tomcat.xudaxian.com # 指定的监听的主机域名，相当于 nginx.conf 的 server { xxx }
    http: # 指定路由规则
      paths:
      - path: /
        pathType: Prefix # 匹配规则，Prefix 前缀匹配 tomcat.xudaxian.com/* 都可以匹配到
        backend: # 指定路由的后台服务的 service 名称
          service:
            name: tomcat-svc # 服务名
            port:
              number: 8080 # 服务的端口
```

kubectl apply -f k8s-ingress.yaml

![30.png](https://cdn.nlark.com/yuque/0/2022/png/513185/1648106931626-b37bcc56-7bd2-4e7b-ac9a-760ea9ce63fb.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_33%2Ctext_6K645aSn5LuZ%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fformat%2Cwebp)



### 8.1.2 测试示例（后面有更简单的测试方法）： 

● 第一种方案： 
  ○ ① 在 win 中的 hosts（C:\Windows\System32\drivers\etc\hosts） 文件中添加如下的信息：
```
# 用来模拟 DNS ，其中 192.168.65.101 是 ingress 部署的机器，nginx.xudaxian.com 和 tomcat.xudaxian.com 是 ingress 文件中监听的域名
192.168.65.101 nginx.xudaxian.com
192.168.65.101 tomcat.xudaxian.com
```
  ②  在 win 中使用 curl 命令：


●  第二种方案： 
  ○ ① 重新建立一个虚拟机（安装有 CentOS7 系统，并且带有桌面），在 hosts （/etc/hosts）文件中添加如下的内容：

```
# 用来模拟 DNS ，其中 192.168.65.101 是 ingress 部署的机器，nginx.xudaxian.com 和 tomcat.xudaxian.com 是 ingress 文件中监听的域名
192.168.65.101 nginx.xudaxian.com
192.168.65.101 tomcat.xudaxian.com
```

  ② 通过浏览器访问：
证明：ingress 规则会生效到所有安装了 IngressController 的机器的 nginx 配置 

cat nginx.conf |  grep nginx.xudaxian.com -A 20



## 8.2 默认后端 


```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-http
  namespace: default
  annotations: 
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"  
spec:
  defaultBackend: # 指定所有未匹配的默认后端
    service:
      name: nginx-svc
      port:
        number: 80
  rules: 
    - host: tomcat.com 
      http: 
        paths:
          - path: /abc
            pathType: Prefix 
            backend: 
              service:
                name: tomcat-svc
                port:
                  number: 8080
```

效果：
- tomcat.com 域名的 非 `/abc` 开头的请求，都会转到 defaultBackend 。
- 非 tomcat.com 域名下的所有请求，也会转到 defaultBackend 。

## 8.3 Ingress 中的 nginx 的全局配置

- [官方文档](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/)。


方式一：在安装的时候，添加配置。
```
apiVersion: v1
data:
  allow-snippet-annotations: "true" # Nginx 的全局配置
kind: ConfigMap
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.1.2
    helm.sh/chart: ingress-nginx-4.0.18
  name: ingress-nginx-controller
  namespace: ingress-nginx
```


方式二：编辑 cm ：

```
kubectl edit cm ingress-nginx-controller -n ingress-nginx

# 配置项加上 
data:
  map-hash-bucket-size: "128" # Nginx 的全局配置
  ssl-protocols: SSLv2
```



## 8.4 限流

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rate-ingress
  namespace: default
  annotations: # https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#rate-limiting
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"  
    nginx.ingress.kubernetes.io/limit-rps: "1" # 限流
spec:
  rules:
  - host: nginx.xudaxian.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-svc
            port:
              number: 80
```



## 8.5 路径重写


路径重写，经常用于前后端分离。

  

![](https://cdn.nlark.com/yuque/0/2022/png/513185/1648107006130-ed856eae-d70e-4aee-85d4-5e7322d36c74.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_30%2Ctext_6K645aSn5LuZ%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)


```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-rewrite
  namespace: default
  annotations:
   nginx.ingress.kubernetes.io/rewrite-target: /$2 # 路径重写
spec:
  ingressClassName: nginx
  rules:
  - host: baidu.com
    http:
      paths:
      - path: /api(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: nginx-svc
            port:
              number: 80
```



## 8.6 基于 Cookie 的会话保持技术

 Service 只能基于 ClientIP，但是 ingress 是七层负载均衡，可以基于 Cookie 实现会话保持。 

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rate-ingress
  namespace: default
  annotations: # https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#rate-limiting
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"  
    nginx.ingress.kubernetes.io/affinity: "cookie"
spec:
  rules:
  - host: nginx.xudaxian.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-svc
            port:
              number: 80
              7
```


## 8.7 配置 SSL


 生成证书语法：
 
```
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ${KEY_FILE} -out ${CERT_FILE} -subj "/CN=${HOST:baidu.com}/O=${HOST:baidu.com}"
kubectl create secret tls ${CERT_NAME:baidu-tls} --key ${KEY_FILE:tls.key} --cert ${CERT_FILE:tls.cert}
```


生成证书

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.cert -subj "/CN=xudaxian.com/O=xudaxian.com"
kubectl create secret tls xudaxian-tls --key tls.key --cert tls.cert
```


例子 

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-tls
  namespace: default
  annotations: 
    kubernetes.io/ingress.class: "nginx"  
spec:
  tls:
  - hosts:
      - nginx.xudaxian.com # 通过浏览器访问 https://nginx.xudaxian.com
      - tomcat.xudaxian.com # 通过浏览器访问 https://tomcat.xudaxian.com
    secretName: xudaxian-tls
  rules:
  - host: nginx.xudaxian.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-svc
            port:
              number: 80
  - host: tomcat.xudaxian.com 
    http: 
      paths:
      - path: /
        pathType: Prefix 
        backend:
          service:
            name: tomcat-svc 
            port:
              number: 8080
```




# 9 Ingress 金丝雀发布

## 9.1 概述 


- 以前使用 Kubernetes 的 Service 配合 Deployment 进行金丝雀的部署，原理如下所示：

![](https://cdn.nlark.com/yuque/0/2022/png/513185/1648107035044-f472bdf8-bc59-4025-be59-defc0e1f1c0a.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_45%2Ctext_6K645aSn5LuZ%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

  
- 但是，也是有缺点的，就是不能自定义灰度逻辑，如：指定用户进行灰度（新功能只让 VIP 先体验一个月，一个月之后再将新功能解锁，让所有用户都体验到）。
- Ingress 金丝雀发布的原理如下所示：

![](https://cdn.nlark.com/yuque/0/2022/png/513185/1648107040221-b3946d57-fe50-46b0-aa2d-d9ed445e45d3.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_41%2Ctext_6K645aSn5LuZ%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)


## 9.2 准备工作


部署 Service  和 Deployment：
vi k8s-ingress-canary-deploy.yaml


```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: v1-deployment
  labels:
    app: v1-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: v1-pod
  template:
    metadata:
      name: nginx
      labels:
        app: v1-pod
    spec:
      initContainers:
        - name: alpine
          image: alpine
          imagePullPolicy: IfNotPresent
          command: ["/bin/sh","-c","echo v1-pod > /app/index.html;"]
          volumeMounts:
            - mountPath: /app
              name: app    
      containers:
        - name: nginx
          image: nginx:1.17.2
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
            limits:
              cpu: 250m
              memory: 500Mi 
          volumeMounts:
            - name: app
              mountPath: /usr/share/nginx/html             
      volumes:
        - name: app
          emptyDir: {}       
      restartPolicy: Always
---
apiVersion: v1
kind: Service
metadata:
  name: v1-service
  namespace: default
spec:
  selector:
    app: v1-pod
  type: ClusterIP
  ports:
  - name: nginx
    protocol: TCP
    port: 80
    targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: v2-deployment
  labels:
    app: v2-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: v2-pod
  template:
    metadata:
      name: v2-pod
      labels:
        app: v2-pod
    spec:
      containers:
        - name: nginx
          image: nginx:1.17.2
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
            limits:
              cpu: 250m
              memory: 500Mi 
      restartPolicy: Always
---
apiVersion: v1
kind: Service
metadata:
  name: v2-service
  namespace: default
spec:
  selector:
    app: v2-pod
  type: ClusterIP
  ports:
  - name: nginx
    protocol: TCP
    port: 80
    targetPort: 80
```


kubectl apply -f k8s-ingress-canary-deploy.yaml


### 9.2.1 部署普通的 ingress ：

vi k8s-ingress-v1.yaml

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-v1
  namespace: default
spec:
  ingressClassName: "nginx"
  rules:
  - host: nginx.xudaxian.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: v1-service
            port:
              number: 80


```


kubectl apply -f k8s-ingress-v1.yaml



测试 ingress ：
```

# curl 来模拟请求
curl -H "Host: canary.example.com" http://EXTERNAL-IP # EXTERNAL-IP 替换为 Nginx Ingress 自身对外暴露的 IP

```

curl -H "Host: nginx.xudaxian.com" http://192.168.65.101



## 9.3 基于 Header 的流量切分


vi k8s-ingress-header.yaml

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-canary
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/canary: "true" # 开启金丝雀 
    nginx.ingress.kubernetes.io/canary-by-header: "Region" # 基于请求头
    # 如果 请求头 Region = always ，就路由到金丝雀版本；如果 Region = never ，就永远不会路由到金丝雀版本。
    nginx.ingress.kubernetes.io/canary-by-header-value: "sz" # 自定义值
    # 如果 请求头 Region = sz ，就路由到金丝雀版本；如果 Region != sz ，就永远不会路由到金丝雀版本。
    # nginx.ingress.kubernetes.io/canary-by-header-pattern: "sh|sz"
    # 如果 请求头 Region = sh 或 Region = sz ，就路由到金丝雀版本；如果 Region != sz 并且 Region != sz ，就永远不会路由到金丝雀版本。
spec:
  ingressClassName: "nginx"
  rules:
  - host: nginx.xudaxian.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: v2-service
            port:
              number: 80
```

kubectl apply -f k8s-ingress-header.yaml


测试 

```
curl -H "Host: nginx.xudaxian.com" http://192.168.65.101

curl -H "Host: nginx.xudaxian.com" -H "Region: sz" http://192.168.65.101
curl -H "Host: nginx.xudaxian.com" -H "Region: sh" http://192.168.65.101

```



## 9.4 基于 Cookie 的流量切分

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-canary
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/canary: "true" # 开启金丝雀 
    nginx.ingress.kubernetes.io/canary-by-cookie: "vip" # 如果 cookie 是 vip = always ，就会路由到到金丝雀版本；如果 cookie 是 vip = never ，就永远不会路由到金丝雀的版本。
spec:
  ingressClassName: "nginx"
  rules:
  - host: nginx.xudaxian.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: v2-service
            port:
              number: 80
```


curl -H "Host: nginx.xudaxian.com" --cookie "vip=always" http://192.168.65.101



# 10 基于服务权重的流量切分

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-canary
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/canary: "true" # 开启金丝雀 
    nginx.ingress.kubernetes.io/canary-weight: "10" # 基于服务权重
spec:
  ingressClassName: "nginx"
  rules:
  - host: nginx.xudaxian.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: v2-service
            port:
              number: 80
          
```


```
for i in {1..10}; do curl -H "Host: nginx.xudaxian.com" http://192.168.65.101; done;
```


