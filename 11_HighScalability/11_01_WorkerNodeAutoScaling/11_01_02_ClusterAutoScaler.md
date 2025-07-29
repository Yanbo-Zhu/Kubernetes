

https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md


On AWS, Cluster Autoscaler utilizes Amazon EC2 Auto Scaling Groups to manage node
groups. Cluster Autoscaler typically runs as a `Deployment` in your cluster.

# 1 Requirements

Cluster Autoscaler requires Kubernetes v1.3.0 or greater.

# 2 Permissions

Cluster Autoscaler requires the ability to examine and modify EC2 Auto Scaling Groups. We recommend using [IAM roles for Service
Accounts](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) to associate the Service Account that the Cluster Autoscaler Deployment runs as
with an IAM role that is able to perform these functions. If you are unable to use IAM Roles for Service Accounts, you may associate an IAM service role with the EC2 instance on which the Cluster Autoscaler pod runs.

## 2.1 IAM Policy

There are a number of ways to run the autoscaler in AWS, which can significantly impact the range of IAM permissions required for the Cluster Autoscaler to function properly. Two options are provided below, one which will allow use of all of the features of the Cluster Autoscaler, the second with a more limited range of IAM actions enabled, which enforces using certain configuration options in the Cluster Autoscaler binary.

It is strongly recommended to restrict the target resources for the autoscaling actions by either [specifying Auto Scaling Group ARNs](https://docs.aws.amazon.com/autoscaling/latest/userguide/control-access-using-iam.html#policy-auto-scaling-resources) in the `Resource` list of the policy or
[using tag based conditionals](https://docs.aws.amazon.com/autoscaling/ec2/userguide/control-access-using-iam.html#security_iam_service-with-iam-tags). The [minimal policy](#minimal-iam-permissions-policy) includes an example of restricting by ASG ARN.

### 2.1.1 Full Cluster Autoscaler Features Policy (Recommended)

Permissions required when using [ASG Autodiscovery](#Auto-discovery-setup) and Dynamic EC2 List Generation (the default behaviour). In this example, only the second block of actions should be updated to restrict the resources/add conditionals:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "autoscaling:DescribeAutoScalingGroups",
        "autoscaling:DescribeAutoScalingInstances",
        "autoscaling:DescribeLaunchConfigurations",
        "autoscaling:DescribeScalingActivities",
        "ec2:DescribeImages",
        "ec2:DescribeInstanceTypes",
        "ec2:DescribeLaunchTemplateVersions",
        "ec2:GetInstanceTypesFromInstanceRequirements",
        "eks:DescribeNodegroup"
      ],
      "Resource": ["*"]
    },
    {
      "Effect": "Allow",
      "Action": [
        "autoscaling:SetDesiredCapacity",
        "autoscaling:TerminateInstanceInAutoScalingGroup"
      ],
      "Resource": ["*"]
    }
  ]
}
```

### 2.1.2 Minimal IAM Permissions Policy

*NOTE:* The below policies/arguments to the Cluster Autoscaler need to be modified as appropriate for the names of your ASGs, as well as account ID and AWS region before being used.

The following policy provides the minimum privileges necessary for Cluster Autoscaler to run. When using this policy,==you cannot use autodiscovery of ASGs== . In addition, it restricts the IAM permissions to the node groups the Cluster Autoscaler is configured to scale.

This in turn means that you must pass the following arguments to the Cluster Autoscaler binary, replacing min and max node counts and the ASG:

```bash
--aws-use-static-instance-list=false
--nodes=1:100:exampleASG1
--nodes=1:100:exampleASG2
```

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "autoscaling:DescribeAutoScalingGroups",
        "autoscaling:DescribeAutoScalingInstances",
        "autoscaling:DescribeLaunchConfigurations",
        "autoscaling:DescribeScalingActivities",
        "autoscaling:SetDesiredCapacity",
        "autoscaling:TerminateInstanceInAutoScalingGroup",
        "eks:DescribeNodegroup"
      ],
      "Resource": ["arn:aws:autoscaling:${YOUR_CLUSTER_AWS_REGION}:${YOUR_AWS_ACCOUNT_ID}:autoScalingGroup:*:autoScalingGroupName/${YOUR_ASG_NAME}"]
    }
  ]
}
```

The `"eks:DescribeNodegroup"` permission allows Cluster Autoscaler to pull labels and taints from the EKS DescribeNodegroup API for EKS managed nodegroups. (Note: When an EKS DescribeNodegroup API label and a tag on the underlying autoscaling group have the same key, the EKS DescribeNodegroup API label value will be saved by the Cluster Autoscaler over the autoscaling group tag value.) Currently the Cluster Autoscaler will only call the EKS DescribeNodegroup API when a managed nodegroup is created with 0 nodes and has never had any nodes added to it. Once nodes are added, even if the managed nodegroup is scaled back to 0 nodes, this functionality will not be called anymore. In the case of a Cluster Autoscaler restart, the Cluster Autoscaler will need to repopulate caches so it will call this functionality again if the managed nodegroup is at 0 nodes. Enabling this functionality any time there are 0 nodes in a managed nodegroup (even after a scale-up then scale-down) would require changes to the general shared Cluster Autoscaler code which could happen in the future.

NOTE: For private clusters, in order for the EKS DescribeNodegroup API to work, you need to create an interface endpoint for Amazon EKS (AWS PrivateLink), as described at the [AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/vpc-interface-endpoints.html).

## 2.2 Using OIDC Federated Authentication (IAM roles for service accounts (IRSA))

OIDC federated authentication allows your service to assume an IAM role and interact with AWS services without having to store credentials as environment variables. For an example of how to use AWS IAM OIDC with the Cluster Autoscaler please see [here](CA_with_AWS_IAM_OIDC.md).

https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/CA_with_AWS_IAM_OIDC.md

A) Create an IAM OIDC identity provider for your cluster with the AWS Management Console using the [documentation](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html) .
B) Create a test [IAM policy](https://docs.aws.amazon.com/eks/latest/userguide/create-service-account-iam-policy-and-role.html) for your service accounts.

C) Create an IAM role for your service accounts in the console.
- Retrieve the OIDC issuer URL from the Amazon EKS console description of your cluster . It will look something identical to: '[https://oidc.eks.us-east-1.amazonaws.com/id/xxxxxxxxxx](https://oidc.eks.us-east-1.amazonaws.com/id/xxxxxxxxxx)'
- While creating a new IAM role, In the "Select type of trusted entity" section, choose "Web identity".
- In the "Choose a web identity provider" section: For Identity provider, choose the URL for your cluster. For Audience, type sts.amazonaws.com.
- In the "Attach Policy" section, select the policy to use for your service account, that you created in Section B above.
- After the role is created, choose the role in the console to open it for editing.
- Choose the "Trust relationships" tab, and then choose "Edit trust relationship". Edit the OIDC provider suffix and change it from :aud to :sub. Replace sts.amazonaws.com to your service account ID.
- Update trust policy to finish.

D) Set up [Cluster Autoscaler Auto-Discovery](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml) using the [tutorial](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md#auto-discovery-setup) .
- Open the Amazon EC2 console, and then choose EKS worker node Auto Scaling Groups from the navigation pane.
- In the "Add/Edit Auto Scaling Group Tags" window, please make sure you enter the following tags by replacing 'awsExampleClusterName' with the name of your EKS cluster. Then, choose "Save".

|Plugin|README|
|---|---|
|Key:|k8s.io/cluster-autoscaler/enabled|
|Key:|k8s.io/cluster-autoscaler/'awsExampleClusterName'|
Note: The keys for the tags that you entered don't have values. Cluster Autoscaler ignores any value set for the keys.




## 2.3 Using AWS Credentials  (not recommended)

**NOTE** The following is not recommended for Kubernetes clusters running on AWS. If you are using Amazon EKS, consider using [IAM roles for Service Accounts](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) instead.

For on-premise clusters, you may create an IAM user subject to the above policy and provide the IAM credentials as environment variables in the Cluster Autoscaler deployment manifest. Cluster Autoscaler will use these credentials to
authenticate and authorize itself.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: aws-secret
type: Opaque
data:
  aws_access_key_id: BASE64_OF_YOUR_AWS_ACCESS_KEY_ID
  aws_secret_access_key: BASE64_OF_YOUR_AWS_SECRET_ACCESS_KEY
```

Please refer to the [relevant Kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/secret/#creating-a-secret-manually) for creating a secret manually.

```yaml
env:
  - name: AWS_ACCESS_KEY_ID
    valueFrom:
      secretKeyRef:
        name: aws-secret
        key: aws_access_key_id
  - name: AWS_SECRET_ACCESS_KEY
    valueFrom:
      secretKeyRef:
        name: aws-secret
        key: aws_secret_access_key
  - name: AWS_REGION
    value: YOUR_AWS_REGION
```

# 3 Auto-Discovery Setup

## 3.1 install autodiscover

Auto-Discovery Setup is the preferred method to configure Cluster Autoscaler.

Example deployment:

```
kubectl apply -f examples/cluster-autoscaler-autodiscover.yaml
```

https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml

```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
  name: cluster-autoscaler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["events", "endpoints"]
    verbs: ["create", "patch"]
  - apiGroups: [""]
    resources: ["pods/eviction"]
    verbs: ["create"]
  - apiGroups: [""]
    resources: ["pods/status"]
    verbs: ["update"]
  - apiGroups: [""]
    resources: ["endpoints"]
    resourceNames: ["cluster-autoscaler"]
    verbs: ["get", "update"]
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["watch", "list", "get", "update"]
  - apiGroups: [""]
    resources:
      - "namespaces"
      - "pods"
      - "services"
      - "replicationcontrollers"
      - "persistentvolumeclaims"
      - "persistentvolumes"
    verbs: ["watch", "list", "get"]
  - apiGroups: ["extensions"]
    resources: ["replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["policy"]
    resources: ["poddisruptionbudgets"]
    verbs: ["watch", "list"]
  - apiGroups: ["apps"]
    resources: ["statefulsets", "replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses", "csinodes", "csidrivers", "csistoragecapacities", "volumeattachments"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["batch", "extensions"]
    resources: ["jobs"]
    verbs: ["get", "list", "watch", "patch"]
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    verbs: ["create"]
  - apiGroups: ["coordination.k8s.io"]
    resourceNames: ["cluster-autoscaler"]
    resources: ["leases"]
    verbs: ["get", "update"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["create", "list", "watch"]
  - apiGroups: [""]
    resources: ["configmaps"]
    resourceNames: ["cluster-autoscaler-status", "cluster-autoscaler-priority-expander"]
    verbs: ["delete", "get", "update", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    app: cluster-autoscaler
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
      annotations:
        prometheus.io/scrape: 'true'
        prometheus.io/port: '8085'
    spec:
      priorityClassName: system-cluster-critical
      securityContext:
        runAsNonRoot: true
        runAsUser: 65534
        fsGroup: 65534
        seccompProfile:
          type: RuntimeDefault
      serviceAccountName: cluster-autoscaler
      containers:
        - image: registry.k8s.io/autoscaling/cluster-autoscaler:v1.32.1
          name: cluster-autoscaler
          resources:
            limits:
              cpu: 100m
              memory: 600Mi
            requests:
              cpu: 100m
              memory: 600Mi
          command:
            - ./cluster-autoscaler
            - --v=4
            - --stderrthreshold=info
            - --cloud-provider=aws
            - --skip-nodes-with-local-storage=false
            - --expander=least-waste
            - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<YOUR CLUSTER NAME>
          volumeMounts:
            - name: ssl-certs
              mountPath: /etc/ssl/certs/ca-certificates.crt # /etc/ssl/certs/ca-bundle.crt for Amazon Linux Worker Nodes
              readOnly: true
          imagePullPolicy: "Always"
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop:
                - ALL
            readOnlyRootFilesystem: true
      volumes:
        - name: ssl-certs
          hostPath:
            path: "/etc/ssl/certs/ca-bundle.crt"
```



## 3.2 Instance types selection

Cluster Autoscaler will respect the minimum and maximum values of each Auto Scaling Group. It will only adjust the desired value.


1 <u>Instance types selection</u>
Each Auto Scaling Group should be composed of instance types that provide approximately equal capacity. For example, ASG "xlarge" could be composed of m5a.xlarge, m4.xlarge, m5.xlarge, and m5d.xlarge instance types, because each of those provide 4 vCPUs and 16GiB RAM. Separately, ASG "2xlarge" could be composed of m5a.2xlarge, m4.2xlarge, m5.2xlarge, and m5d.2xlarge instance types, because each of those provide 8 vCPUs and 32GiB RAM.
Cluster Autoscaler will attempt to determine the CPU, memory, and GPU resources provided by an Auto Scaling Group ==based on the instance type specified in its Launch Configuration or Launch Template==. It will also examine any overrides
provided in an ASG's ==Mixed Instances Policy==. If any such overrides are found, only the first instance type found will be used. See [Using Mixed Instances Policies and Spot Instances](#Using-Mixed-Instances-Policies-and-Spot-Instances) for details.


## 3.3 如何让CA 发现某些的ASG 

To enable this, provide the `--node-group-auto-discovery` flag as an argument whose value is a list of tag keys that should be looked for. 
For example, `--node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<cluster-name>`
will find the ASGs that have at least all the given tags. Without the tags, the Cluster Autoscaler will be unable to add new instances to the ASG as it has not been discovered. In the example, a value is not given for the tags and in this case any value will be ignored and will be arbitrary - only the tag name matters. 
这个设置将会寻找至少拥有所有这些指定标签的 EC2 Auto Scaling Groups (ASG)。**如果没有这些标签，Cluster Autoscaler 将无法发现这些 ASG，因此也无法对其进行扩容**。
在上述示例中，**标签没有指定具体的值**，此时只要标签键存在，标签值将被忽略 —— 也就是说，只要标签名匹配，就能被识别，**标签值可以是任意的**。


Optionally, the tag value can be set to be usable and custom tags can also be added.  For example,
`--node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled=foo,k8s.io/cluster-autoscaler/<cluster-name>=bar,my-custom-tag=custom-value`.
Now the ASG tags must have the correct values as well as the custom tag to be successfully discovered by the Cluster Autoscaler.

在这个示例中，Cluster Autoscaler 仅会发现那些同时拥有以下三个标签 _及其指定值_ 的 ASG：
1. `k8s.io/cluster-autoscaler/enabled=foo`
2. `k8s.io/cluster-autoscaler/<cluster-name>=bar`
3. `my-custom-tag=custom-value`
只有当 ASG 拥有这些完整的标签键值对时，才能被成功识别并用于自动扩缩容

---

给 Cluster Autoscaler 的 Deployment 添加标注，以便它识别自己管理的集群和 ASG：
```yaml
spec:
  template:
    metadata:
      annotations:
        cluster-autoscaler.kubernetes.io/safe-to-evict: "false"


containers:
  - name: cluster-autoscaler
    image: k8s.gcr.io/autoscaling/cluster-autoscaler:<version>
    command:
      - ./cluster-autoscaler
      - --cloud-provider=aws
      - --namespace=kube-system
      - --cluster-name=<your-cluster-name>
      - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<your-cluster-name>
      - --balance-similar-node-groups
      - --skip-nodes-with-system-pods=false


```

## 3.4 Pod Scheduling
When scaling up from 0 nodes, the Cluster Autoscaler reads ASG tags to derive information about the specifications of the nodes i.e labels and taints in that ASG. Note that it does not actually apply these labels or taints - this is done by an AWS generated user data script. ==(The ASG tags only allow the Cluster Autoscaler to discover the Auto Scaling Groups and perform specific operations on the underlying nodes. However, these ASG tags do **not** automatically label or taint the nodes with the same values as the ASG tags.)==
It gives the Cluster Autoscaler information about whether pending pods will be able to be scheduled should a new node be spun up for a particular ASG with the assumption the ASG tags accurately reflect the labels/taint actually applied.
当从 0 个节点开始扩容时，Cluster Autoscaler 会读取 Auto Scaling Group（ASG） 的标签，以获取该 ASG 中节点的配置信息，例如 labels（标签） 和 taints（污点）。
需要注意的是：Cluster Autoscaler 本身并不会真正应用这些标签或污点 —— 实际应用这些配置的是由 AWS 生成的 user data 脚本。
这些 ASG 标签只是让 Cluster Autoscaler 能够判断：如果为某个特定 ASG 启动一个新节点，当前处于 Pending 状态的 Pod 是否能够成功被调度到这个节点上，前提是假设这些标签和污点在节点实际创建时会被准确地应用。

2.1 NodeSelector 配套的 打在node 上的label 
以下内容仅在 从 0 个节点开始扩容 时才是必需的： 如果某个 Deployment 使用了 NodeSelector，那么对应的 自动伸缩组（ASG） 必须带有特定的 标签（Tag），否则 Cluster Autoscaler 无法识别该 ASG 拥有该标签，从而不会触发扩容操作。
The following is only required if scaling up from 0 nodes. The Cluster Autoscaler will require the label tag on the ASG should a deployment have a NodeSelector, else no scaling will occur as the Cluster Autoscaler does not realise the ASG has that particular label. 

The tag is of the format
`k8s.io/cluster-autoscaler/node-template/label/<label-name>`: `<label-value>` is the name of the label and the value of each tag specifies the label value.
Example tags:
- `k8s.io/cluster-autoscaler/node-template/label/foo`: `bar`

Cluster Autoscaler 会通过 ASG 上的这些标签来预估扩容后的节点是否满足待调度 Pod 的调度条件（例如是否匹配 NodeSelector）。但请注意，**这些标签并不会真正应用到节点上**，它们只是用来供 autoscaler 判断是否值得扩容。实际应用标签是在启动节点时由 AWS 的 UserData 脚本完成的。



2.2 Pod Toleranz 配套的 打在 node 上的 taint 
如果某个 ASG 中的节点需要带有 taint（污点），那么必须在该 ASG 上添加相应的 taint 标签，否则 Cluster Autoscaler 可能会错误地扩容出带有 taint 的节点，而这些节点又不能运行待调度的 Pod，导致调度失败。
The following is only required if scaling up from 0 nodes. The Cluster Autoscaler will require the taint tag on the ASG, else tainted nodes may get spun up that cannot actually have the pending pods run on it. 
The tag is of the format `k8s.io/cluster-autoscaler/node-template/taint/<taint-name>`:`<taint-value:taint-effect>` is the name of the taint and the value of each tag specifies the taint value and effect with the format `<taint-value>:<taint-effect>`.

Example tags:
- `k8s.io/cluster-autoscaler/node-template/taint/dedicated`: `true:NoSchedule`

- `<taint-name>` 是污点的名称
- `<taint-value>` 是污点的值
- `<taint-effect>` 是污点的影响（例如 `NoSchedule`, `PreferNoSchedule`, `NoExecute`）

这个标签的含义是：该节点会有一个名为 `dedicated` 的污点，其值为 `true`，效果为 `NoSchedule`。Cluster Autoscaler 通过这些标签来**预测**新建节点的属性，从而决定是否扩容。注意：实际的 taint 是由 AWS 的启动脚本（UserData）应用到节点上的。

**NOTE:** It is your responsibility to ensure such labels and/or taints are applied via the node's kubelet configuration at startup. Cluster Autoscaler will not set the node taints for you.
你需要自行确保通过节点启动时的 kubelet 配置来应用这些标签和/或污点（taints）。Cluster Autoscaler 不会自动为你设置节点的污点。
## 3.5 用ASG 标签表明这个ASG中资源的大小, 帮助 CA 决策 


这些标签仅用于 **扩容前的模拟调度决策** —— 实际的资源仍然由 EC2 实例类型决定，而标签只是告诉 Cluster Autoscaler 这些资源的预计数量。这样在 **从 0 节点扩容时**，Autoscaler 才能判断新建节点是否满足待调度 Pod 的资源请求。
From version 1.14, Cluster Autoscaler can also determine the resources provided by each Auto Scaling Group via tags. 

The tag is of the format  `k8s.io/cluster-autoscaler/node-template/resources/<resource-name>`.
`<resource-name>` is the name of the resource, such as `ephemeral-storage`. The value of each tag specifies the amount of resource provided. The units are identical to the units used in the `resources` field of a Pod specification.

Example tags:
- `k8s.io/cluster-autoscaler/node-template/resources/ephemeral-storage`: `100G`

- `<resource-name>` 是资源的名称，例如 `ephemeral-storage`（临时存储）。 这个示例表示：该 Auto Scaling Group 中的节点提供 100 GB 的临时存储空间（`ephemeral-storage`）。
- 标签的值表示该资源的数量，其单位与 Pod 规范中 `resources` 字段中使用的单位相同（如 Mi、Gi、m、G 等）。


## 3.6 使用ASG 标签可以指定自动扩缩容选项， 覆盖为整个 Cluster Autoscaler 设置的全局配置


ASG 标签可以指定自动扩缩容选项，从而覆盖为整个 Cluster Autoscaler 设置的全局配置。
ASG labels can specify autoscaling options, overriding the global cluster-autoscaler settings for the labeled ASGs. Those labels takes the same values format as the cluster-autoscaler command line flags they override (a float or a duration, encoded as string). Currently supported autoscaling options (and example values) are:
* `k8s.io/cluster-autoscaler/node-template/autoscaling-options/scaledownutilizationthreshold`: `0.5`
  (overrides `--scale-down-utilization-threshold` value for that specific ASG)
* `k8s.io/cluster-autoscaler/node-template/autoscaling-options/scaledowngpuutilizationthreshold`: `0.5`
  (overrides `--scale-down-gpu-utilization-threshold` value for that specific ASG)
* `k8s.io/cluster-autoscaler/node-template/autoscaling-options/scaledownunneededtime`: `10m0s`
  (overrides `--scale-down-unneeded-time` value for that specific ASG)
* `k8s.io/cluster-autoscaler/node-template/autoscaling-options/scaledownunreadytime`: `20m0s`
  (overrides `--scale-down-unready-time` value for that specific ASG)
* `k8s.io/cluster-autoscaler/node-template/autoscaling-options/ignoredaemonsetsutilization`: `true`
  (overrides `--ignore-daemonsets-utilization` value for that specific ASG)

`--scale-down-utilization-threshold` 是加在 这里的 https://github.com/kubernetes/autoscaler/blob/3d748040d9951522c5445abbf6fa379c5ec71ed2/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml#L165






## 3.7 Recommendations:
- It is recommended to use a second tag like `k8s.io/cluster-autoscaler/<cluster-name>` when `k8s.io/cluster-autoscaler/enabled` is used across many clusters to prevent ASGs from different clusters having conflicts.
  An ASG must contain at least all the tags specified and as such secondary tags can differentiate between different clusters ASGs.
- To prevent conflicts, do not provide a `--nodes` argument if `--node-group-auto-discovery` is specified.
- Be sure to add `autoscaling:DescribeLaunchConfigurations` or `ec2:DescribeLaunchTemplateVersions` to the `Action` list of the IAM Policy used by Cluster Autoscaler, depending on whether your ASG utilizes Launch Configurations or Launch Templates.
- If Cluster Autoscaler adds a node to the cluster, and the node has taints applied when it joins the cluster that Cluster Autoscaler was unaware of (because the tag wasn't supplied in ASG), this can lead to significant confusion and misbehaviour.

- 如果在多个集群中都使用了 `k8s.io/cluster-autoscaler/enabled` 标签，建议再加上一个 `k8s.io/cluster-autoscaler/<cluster-name>` 标签来区分不同集群的 ASG，避免冲突。
- 如果启用了 `--node-group-auto-discovery`，请不要再设置 `--nodes` 参数，以避免配置冲突。
- 请确保将 `autoscaling:DescribeLaunchConfigurations` 或 `ec2:DescribeLaunchTemplateVersions` 加入到 Cluster Autoscaler 所使用的 IAM Policy 的 `Action` 列表中，具体取决于你的 ASG 是使用 Launch Configuration 还是 Launch Template。    
- 如果 Cluster Autoscaler 向集群添加节点，而该节点在加入集群时带有未被 ASG 标签声明的污点，这可能导致调度行为异常或令人困惑。



## 3.8 Special note on GPU instances

节点上提供 GPU 资源的设备插件需要一些时间才能将 GPU 资源通告给集群，这可能会导致 Cluster Autoscaler 多次不必要地进行扩容。
The device plugin on nodes that provides GPU resources can take some time to advertise the GPU resource to the cluster. This may cause Cluster Autoscaler to unnecessarily scale out multiple times.

To avoid this, you can configure `kubelet` on your GPU nodes to label the node before it joins the cluster by passing it the `--node-labels` flag. The label format is as follows:
- Cluster Autoscaler < 1.15: `cloud.google.com/gke-accelerator=<gpu-type>`
- Cluster Autoscaler >= 1.15: `k8s.amazonaws.com/accelerator=<gpu-type>`

`<gpu-type>` varies by instance type. On P2 instances, for example, the value is `nvidia-tesla-k80`.


# 4 ASG 上标签的例子 


- 你有一个 EKS 集群名为 `my-eks-cluster`
- 你想让 Cluster Autoscaler 能发现这个 ASG 并正确推断其节点的 labels、taints、资源等信息
- 同时你想为这个 ASG 配置自定义的 scale down 策略

必需的 ASG 标签（基础自动发现）

| Key                                        | Value   | 说明                    |
| ------------------------------------------ | ------- | --------------------- |
| `k8s.io/cluster-autoscaler/enabled`        | `true`  | 启用自动发现（值可以任意）         |
| `k8s.io/cluster-autoscaler/my-eks-cluster` | `owned` | 指明属于哪个集群（值任意，但建议保持一致） |

可选标签：Node Labels / Taints / Resources / Autoscaling Options

| Key                                                                                         | Value             | 示例说明                           |
| ------------------------------------------------------------------------------------------- | ----------------- | ------------------------------ |
| `k8s.io/cluster-autoscaler/node-template/label/node-type`                                   | `spot`            | 设置节点标签 node-type=spot          |
| `k8s.io/cluster-autoscaler/node-template/taint/dedicated`                                   | `true:NoSchedule` | 设置污点 dedicated=true:NoSchedule |
| `k8s.io/cluster-autoscaler/node-template/resources/ephemeral-storage`                       | `100Gi`           | 设置本地临时存储容量                     |
| `k8s.io/cluster-autoscaler/node-template/autoscaling-options/scaledownutilizationthreshold` | `0.4`             | 单独为该 ASG 设置更激进的缩容阈值            |
| `k8s.io/cluster-autoscaler/node-template/autoscaling-options/scaledownunneededtime`         | `5m`              | 缩容前等待时间缩短为 5 分钟                |

```yaml
resource "aws_autoscaling_group" "example" {
  name                      = "example-asg"
  min_size                  = 0
  max_size                  = 5
  desired_capacity          = 0
  vpc_zone_identifier       = ["subnet-xxx"]
  launch_template {
    id      = aws_launch_template.example.id
    version = "$Latest"
  }

  tags = [
    {
      key                 = "k8s.io/cluster-autoscaler/enabled"
      value               = "true"
      propagate_at_launch = true
    },
    {
      key                 = "k8s.io/cluster-autoscaler/my-eks-cluster"
      value               = "owned"
      propagate_at_launch = true
    },
    {
      key                 = "k8s.io/cluster-autoscaler/node-template/label/node-type"
      value               = "spot"
      propagate_at_launch = true
    },
    {
      key                 = "k8s.io/cluster-autoscaler/node-template/taint/dedicated"
      value               = "true:NoSchedule"
      propagate_at_launch = true
    },
    {
      key                 = "k8s.io/cluster-autoscaler/node-template/autoscaling-options/scaledownutilizationthreshold"
      value               = "0.4"
      propagate_at_launch = true
    }
  ]
}


```


- 所有标签必须设置 `propagate_at_launch = true`，否则新节点不会继承这些标签。    
- 如果你使用 Launch Template，请确保 `userData` 里正确注入了节点标签和污点，否则标签只是让 Cluster Autoscaler 能识别，但实际节点并不会拥有这些属性。



## 4.1 propagate_at_launch = true 的意义


在 Auto Scaling Group（ASG）中，你可以为 ASG 设置标签。但默认情况下，这些标签只存在于 ASG 本身，并不会自动传递给该 ASG 启动的 EC2 实例。

要让某个标签 在实例启动时自动附加到 EC2 实例上，你必须设置： propagate_at_launch = true


标签只存在于 ASG，不会出现在 EC2 实例上。
对于一些依赖 EC2 实例标签的工具（如 CloudWatch、IAM 条件策略、某些自动化脚本）来说，它们可能就无法识别这些标签。


| 机制                               | 作用对象            | 是否影响 Kubernetes Node 的标签/污点      | 用途说明                                            |
| -------------------------------- | --------------- | -------------------------------- | ----------------------------------------------- |
| `propagate_at_launch = true`     | AWS 层的 EC2 实例标签 | ❌ 不会影响 Kubernetes Node 标签或 taint | 只是让 **EC2 实例继承 ASG 标签**，对 K8s 节点行为无直接影响         |
| **Launch Template 的 `userData`** | 节点启动时执行的脚本      | ✅ 直接决定节点是否带有 label / taint       | 必须在 userData 里注入相关 kubelet 参数或脚本，才能让 Node 被正确标记 |

`propagate_at_launch = true` 这个标签会出现在 **EC2 实例标签上**，但 K8s 中的 `kubectl get nodes --show-labels` 并不会看到 `node-type=gpu`。Cluster Autoscaler 只会根据这个标签**预估扩容时要选哪个 ASG**，不会修改任何 Node 标签。


----

只是让 EC2 实例继承 ASG 标签，对 K8s 节点行为无直接影响   那么 让 EC2 实例继承 ASG 标签的意义是什么 

让 EC2 实例继承 ASG 标签（`propagate_at_launch = true`）的意义主要有以下几点：
1. **方便 AWS 层资源管理和识别**  
    ASG 标签会自动传递给 EC2 实例，方便你在 AWS 控制台或者通过 API 识别、筛选属于哪个 ASG 的实例。  
    例如，你可以通过标签快速查询所有属于某个 ASG 的实例，方便监控、计费和管理。
    
2. **权限和策略应用**  
    有些 AWS IAM 策略或自动化工具，可能会根据 EC2 实例标签来授予权限或做其他操作。  
    例如，基于标签的权限控制 (IAM Conditions) 可以限制某些实例执行特定操作。
    
3. **自动化脚本和运维工具使用**  
    许多自动化工具（监控、备份、配置管理）会根据 EC2 实例标签来决定执行哪些任务，继承标签后能确保工具正常识别实例属性。
    
4. **Cluster Autoscaler 用于识别 ASG**  
    Cluster Autoscaler 通过查看 ASG 标签来知道它管理的哪些 ASG，虽然标签不会直接影响 Kubernetes 节点标签，但 Cluster Autoscaler 依赖这些标签信息来判断如何扩缩容。


- **AWS 实例标签是 AWS 资源层面的管理标识**，方便你在 AWS 生态内管理和操作实例。
- **Kubernetes 节点标签/污点是集群内部的调度和管理标识**，用于 Pod 调度等逻辑。
- 两者服务于不同层面，标签继承是 AWS 资源管理的基础操作，不影响 Kubernetes 内部行为。




## 4.2 你使用 Launch Template/Conguration，请确保 userData 里正确注入了节点标签和污点, 才会将 真的把这些属性“设置”到新节点上。

你在 Auto Scaling Group（ASG）里添加了这样的标签：
k8s.io/cluster-autoscaler/node-template/label/node-type = spot
k8s.io/cluster-autoscaler/node-template/taint/dedicated = true:NoSchedule

这会告诉 **Cluster Autoscaler**：

> “嘿，这个 ASG 中的节点应该有 `node-type=spot` 标签，并且有一个 `dedicated=true:NoSchedule` 的 taint，所以如果有 Pod 需要这些属性，我就可以扩容这组节点。”

但实际上 —— Cluster **Autoscaler 不会真正去设置这些标签或 taint 到新节点上**。它只是“相信”这些新节点在启动时已经具备这些属性。


---

你自己在 Launch Template 的 userData（启动脚本）里设置！

比如通过 kubelet 的启动参数指定：

--node-labels=node-type=spot
--register-with-taints=dedicated=true:NoSchedule

这些参数通常写在 Launch Template 的 `userData` 部分中（base64 编码的脚本），用于实例启动时自动注入到节点。


---

 如果你没有在 userData 中设置这些参数会怎样？

- 节点实际上 没有 node-type=spot 标签，
- 节点也 没有 dedicated=true:NoSchedule 的 taint，
- 但是 Cluster Autoscaler 以为它们有！
- 结果：Pod 仍然调度不上，扩容白做了，系统出现“误判”。

---

示例：Terraform 配置 Launch Template 的方式（包含设置节点标签和 taint）
```yaml


resource "aws_launch_template" "example" {
  name_prefix   = "example-"
  image_id      = "ami-12345678" # 替换为你自己的 AMI ID
  instance_type = "t3.medium"

  # 你需要在 `userData` 中传递 kubelet 启动参数，让新 EC2 节点启动后，kubelet 运行时自动添加标签和污点。
  user_data = base64encode(<<-EOF
    #!/bin/bash
    set -ex

    # 安装必要的工具，如 awscli、kubelet 等（略）

    # 启动 kubelet，注入标签和 taint
    /etc/eks/bootstrap.sh ${cluster_name} \
      --kubelet-extra-args "--node-labels=node-type=spot,env=dev --register-with-taints=dedicated=true:NoSchedule"
  EOF
  )

  # 如果你使用 Launch Template 与 EKS 结合，还可以配置以下内容：
  lifecycle {
    create_before_destroy = true
  }

  tag_specifications {
    resource_type = "instance"
    tags = {
      Name = "example-node"
    }
  }

  tag_specifications {
    resource_type = "volume"
    tags = {
      Name = "example-node-volume"
    }
  }
}


```




----

使用  launch_template  配合 Cluster Autoscaler 的标签（设置在 ASG 上）：
Cluster Autoscaler 依赖以下标签识别节点组：
如果你使用 eks_node_group 模块（EKS 托管节点组），你也可以通过 launch_template 块传入这个 Launch Template ID。
```yaml
resource "aws_autoscaling_group" "example" {
  name_prefix = "example-asg"
  ...

  tag {
    key                 = "k8s.io/cluster-autoscaler/enabled"
    value               = "true"
    propagate_at_launch = true
  }

  tag {
    key                 = "k8s.io/cluster-autoscaler/${cluster_name}"
    value               = "owned"
    propagate_at_launch = true
  }

  tag {
    key                 = "k8s.io/cluster-autoscaler/node-template/label/node-type"
    value               = "spot"
    propagate_at_launch = true
  }

  tag {
    key                 = "k8s.io/cluster-autoscaler/node-template/taint/dedicated"
    value               = "true:NoSchedule"
    propagate_at_launch = true
  }
}

```


---


用 Launch Configuration 设置节点标签和 taint 的方法

⚠️ Launch Configuration 和 Launch Template 的核心区别是：Launch Configuration 是旧版机制，不支持像 Launch Template 那样使用更灵活的参数（如版本、多个网络接口等）。建议优先使用 Launch Template，但如果你必须使用 Launch Configuration，也可以实现节点标签和污点注入。


```
resource "aws_launch_configuration" "example" {
  name_prefix   = "eks-node-"
  image_id      = data.aws_ami.eks_worker.id
  instance_type = "t3.large"
  key_name      = var.ssh_key_name
  iam_instance_profile = aws_iam_instance_profile.eks_nodes.name
  security_groups      = [aws_security_group.eks_nodes.id]

  user_data = base64encode(<<-EOF
    #!/bin/bash
    /etc/eks/bootstrap.sh ${var.cluster_name} \
      --kubelet-extra-args '--node-labels=node-type=spot --register-with-taints=dedicated=true:NoSchedule'
  EOF
  )

  lifecycle {
    create_before_destroy = true
  }
}

```



---

核心区别：Launch Configuration vs Launch Template
Launch Configuration 虽然还在支持，但：
⚠️ AWS 早已建议迁移到 Launch Template，并且某些新功能只能通过 Launch Template 使用。

| 功能/特性                                       | **Launch Configuration** | **Launch Template**                                                                                                                   |
| ------------------------------------------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------- |
| 🔄 **版本管理**                                 | ❌ 不支持                    | ✅ **支持版本控制**（可以保留多个版本并指定默认版本）                                                                                                         |
| 🖥️ **多个网络接口**                              | ❌ 不支持                    | ✅ 支持多个 ENI（弹性网络接口）配置                                                                                                                  |
| 🧾 **支持 Launch Template Overrides（混合实例策略）** | ❌ 不支持                    | ✅ **必须使用 Launch Template**                                                                                                            |
| 💾 **EBS 卷配置灵活性**                           | 🚫 有限                    | ✅ 更灵活、支持高级选项（如加密、删除策略）                                                                                                                |
| 📝 **User Data 动态参数支持**                     | ❌ 只能静态内容                 | ✅ 支持 [Instance Metadata substitution](https://docs.aws.amazon.com/autoscaling/ec2/userguide/launch-templates.html#template-user-data) |
| 🧩 **GPU 等资源类型支持**                          | 🚫 旧资源支持较差               | ✅ 推荐用于 **GPU/加速型实例、Spot 配置**                                                                                                          |
| 🚀 **未来支持**                                 | ❌ AWS 已不再建议使用            | ✅ **AWS 推荐新项目使用 Launch Template**                                                                                                     |





# 5 Manual configuration: 限定ASG 的max min node number 

Cluster Autoscaler can also be configured manually if you wish by passing the `--nodes` argument at startup. 
The format of the argument is `--nodes=<min>:<max>:<asg-name>`, where `<min>` is the minimum number of nodes, `<max>` is the maximum number of nodes, and `<asg-name>` is the Auto Scaling Group name.

You can pass multiple `--nodes` arguments if you have multiple Auto Scaling Groups you want Cluster Autoscaler to use.

**NOTES**:

- Both `<min>` and `<max>` must be within the range of the minimum and maximum instance counts specified by the Auto Scaling group.
- When manual configuration is used, all Auto Scaling groups must use EC2 instance types that provide equal CPU and memory capacity.

Examples:
One ASG Setup (min: 1, max: 10, ASG Name: k8s-worker-asg-1)
可以在启动 Cluster Autoscaler 时加上： `--nodes=1:10:k8s-worker-asg-1`

在 Kubernetes 中可以通过如下方式部署配置文件（示例）：

```
kubectl apply -f examples/cluster-autoscaler-one-asg.yaml
```


```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
  name: cluster-autoscaler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["events", "endpoints"]
    verbs: ["create", "patch"]
  - apiGroups: [""]
    resources: ["pods/eviction"]
    verbs: ["create"]
  - apiGroups: [""]
    resources: ["pods/status"]
    verbs: ["update"]
  - apiGroups: [""]
    resources: ["endpoints"]
    resourceNames: ["cluster-autoscaler"]
    verbs: ["get", "update"]
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["watch", "list", "get", "update"]
  - apiGroups: [""]
    resources:
      - "namespaces"
      - "pods"
      - "services"
      - "replicationcontrollers"
      - "persistentvolumeclaims"
      - "persistentvolumes"
    verbs: ["watch", "list", "get"]
  - apiGroups: ["extensions"]
    resources: ["replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["policy"]
    resources: ["poddisruptionbudgets"]
    verbs: ["watch", "list"]
  - apiGroups: ["apps"]
    resources: ["statefulsets", "replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses", "csinodes", "csidrivers", "csistoragecapacities", "volumeattachments"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["batch", "extensions"]
    resources: ["jobs"]
    verbs: ["get", "list", "watch", "patch"]
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    verbs: ["create"]
  - apiGroups: ["coordination.k8s.io"]
    resourceNames: ["cluster-autoscaler"]
    resources: ["leases"]
    verbs: ["get", "update"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["create", "list", "watch"]
  - apiGroups: [""]
    resources: ["configmaps"]
    resourceNames: ["cluster-autoscaler-status", "cluster-autoscaler-priority-expander"]
    verbs: ["delete", "get", "update", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    app: cluster-autoscaler
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
      annotations:
        prometheus.io/scrape: 'true'
        prometheus.io/port: '8085'
    spec:
      priorityClassName: system-cluster-critical
      securityContext:
        runAsNonRoot: true
        runAsUser: 65534
        fsGroup: 65534
        seccompProfile:
          type: RuntimeDefault
      serviceAccountName: cluster-autoscaler
      containers:
        - image: registry.k8s.io/autoscaling/cluster-autoscaler:v1.32.1
          name: cluster-autoscaler
          resources:
            limits:
              cpu: 100m
              memory: 600Mi
            requests:
              cpu: 100m
              memory: 600Mi
          command:
            - ./cluster-autoscaler
            - --v=4
            - --stderrthreshold=info
            - --cloud-provider=aws
            - --skip-nodes-with-local-storage=false
            - --nodes=1:10:k8s-worker-asg-1
          volumeMounts:
            - name: ssl-certs
              mountPath: /etc/ssl/certs/ca-certificates.crt # /etc/ssl/certs/ca-bundle.crt for Amazon Linux Worker Nodes
              readOnly: true
          imagePullPolicy: "Always"
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop:
                - ALL
            readOnlyRootFilesystem: true
      volumes:
        - name: ssl-certs
          hostPath:
            path: "/etc/ssl/certs/ca-bundle.crt"
```



----

Multiple ASG Setup
如果你希望 Cluster Autoscaler 同时管理多个 ASG，你可以重复加 --nodes 参数，比如：
```
--nodes=1:5:k8s-worker-spot-asg \
--nodes=2:8:k8s-worker-on-demand-asg
```

部署时，示例 YAML 文件可能如下：
```
kubectl apply -f examples/cluster-autoscaler-multi-asg.yaml
```


```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
  name: cluster-autoscaler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["events", "endpoints"]
    verbs: ["create", "patch"]
  - apiGroups: [""]
    resources: ["pods/eviction"]
    verbs: ["create"]
  - apiGroups: [""]
    resources: ["pods/status"]
    verbs: ["update"]
  - apiGroups: [""]
    resources: ["endpoints"]
    resourceNames: ["cluster-autoscaler"]
    verbs: ["get", "update"]
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["watch", "list", "get", "update"]
  - apiGroups: [""]
    resources:
      - "namespaces"
      - "pods"
      - "services"
      - "replicationcontrollers"
      - "persistentvolumeclaims"
      - "persistentvolumes"
    verbs: ["watch", "list", "get"]
  - apiGroups: ["extensions"]
    resources: ["replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["policy"]
    resources: ["poddisruptionbudgets"]
    verbs: ["watch", "list"]
  - apiGroups: ["apps"]
    resources: ["statefulsets", "replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses", "csinodes", "csidrivers", "csistoragecapacities", "volumeattachments"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["batch", "extensions"]
    resources: ["jobs"]
    verbs: ["get", "list", "watch", "patch"]
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    verbs: ["create"]
  - apiGroups: ["coordination.k8s.io"]
    resourceNames: ["cluster-autoscaler"]
    resources: ["leases"]
    verbs: ["get", "update"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["create", "list", "watch"]
  - apiGroups: [""]
    resources: ["configmaps"]
    resourceNames: ["cluster-autoscaler-status", "cluster-autoscaler-priority-expander"]
    verbs: ["delete", "get", "update", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    app: cluster-autoscaler
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
      annotations:
        prometheus.io/scrape: 'true'
        prometheus.io/port: '8085'
    spec:
      priorityClassName: system-cluster-critical
      securityContext:
        runAsNonRoot: true
        runAsUser: 65534
        fsGroup: 65534
        seccompProfile:
          type: RuntimeDefault
      serviceAccountName: cluster-autoscaler
      containers:
        - image: registry.k8s.io/autoscaling/cluster-autoscaler:v1.32.1
          name: cluster-autoscaler
          resources:
            limits:
              cpu: 100m
              memory: 600Mi
            requests:
              cpu: 100m
              memory: 600Mi
          command:
            - ./cluster-autoscaler
            - --v=4
            - --stderrthreshold=info
            - --cloud-provider=aws
            - --skip-nodes-with-local-storage=false
            - --expander=least-waste
            - --nodes=1:10:k8s-worker-asg-1
            - --nodes=1:3:k8s-worker-asg-2
          volumeMounts:
            - name: ssl-certs
              mountPath: /etc/ssl/certs/ca-certificates.crt # /etc/ssl/certs/ca-bundle.crt for Amazon Linux Worker Nodes
              readOnly: true
          imagePullPolicy: "Always"
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop:
                - ALL
            readOnlyRootFilesystem: true
      volumes:
        - name: ssl-certs
          hostPath:
            path: "/etc/ssl/certs/ca-bundle.crt"
```




---

手动配置时，你必须确保 min 和 max 都在 ASG 的实际设置范围之内（即 Auto Scaling Group 控台里设置的最小和最大实例数）。

所有被手动配置的 ASG，必须使用相同规格的实例类型（即 CPU 和内存一致），否则 CA 无法正确做出扩缩容决策。

如果你使用这种手动 --nodes 配置方式，就不要使用自动发现（--node-group-auto-discovery），避免冲突。


# 6 Control Plane (previously referred to as master) Node Setup

**NOTE**: This setup is not compatible with Amazon EKS.

To run a CA pod on a control plane node the CA deployment should tolerate the `master` taint and `nodeSelector` should be used to schedule the pods on a control plane node.

Please replace `{{ node_asg_min }}`, `{{ node_asg_max }}` and `{{ name }}` with your ASG setting in the yaml file.

```
kubectl apply -f examples/cluster-autoscaler-run-on-control-plane.yaml
```

```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
  name: cluster-autoscaler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["events", "endpoints"]
    verbs: ["create", "patch"]
  - apiGroups: [""]
    resources: ["pods/eviction"]
    verbs: ["create"]
  - apiGroups: [""]
    resources: ["pods/status"]
    verbs: ["update"]
  - apiGroups: [""]
    resources: ["endpoints"]
    resourceNames: ["cluster-autoscaler"]
    verbs: ["get", "update"]
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["watch", "list", "get", "update"]
  - apiGroups: [""]
    resources:
      - "namespaces"
      - "pods"
      - "services"
      - "replicationcontrollers"
      - "persistentvolumeclaims"
      - "persistentvolumes"
    verbs: ["watch", "list", "get"]
  - apiGroups: ["extensions"]
    resources: ["replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["policy"]
    resources: ["poddisruptionbudgets"]
    verbs: ["watch", "list"]
  - apiGroups: ["apps"]
    resources: ["statefulsets", "replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses", "csinodes", "csidrivers", "csistoragecapacities", "volumeattachments"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["batch", "extensions"]
    resources: ["jobs"]
    verbs: ["get", "list", "watch", "patch"]
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    verbs: ["create"]
  - apiGroups: ["coordination.k8s.io"]
    resourceNames: ["cluster-autoscaler"]
    resources: ["leases"]
    verbs: ["get", "update"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["create", "list", "watch"]
  - apiGroups: [""]
    resources: ["configmaps"]
    resourceNames: ["cluster-autoscaler-status", "cluster-autoscaler-priority-expander"]
    verbs: ["delete", "get", "update", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    app: cluster-autoscaler
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
      annotations:
        prometheus.io/scrape: 'true'
        prometheus.io/port: '8085'
    spec:
      priorityClassName: system-cluster-critical
      securityContext:
        runAsNonRoot: true
        runAsUser: 65534
        fsGroup: 65534
        seccompProfile:
          type: RuntimeDefault
      serviceAccountName: cluster-autoscaler
      tolerations:
        - effect: NoSchedule
          operator: "Equal"
          value: "true"
          key: node-role.kubernetes.io/control-plane
      nodeSelector:
        kubernetes.io/role: control-plane
      containers:
        - image: registry.k8s.io/autoscaling/cluster-autoscaler:v1.32.1
          name: cluster-autoscaler
          resources:
            limits:
              cpu: 100m
              memory: 600Mi
            requests:
              cpu: 100m
              memory: 600Mi
          command:
            - ./cluster-autoscaler
            - --v=4
            - --stderrthreshold=info
            - --cloud-provider=aws
            - --skip-nodes-with-local-storage=false
            - --nodes={{ node_asg_min }}:{{ node_asg_max }}:{{ name }}
          volumeMounts:
            - name: ssl-certs
              mountPath: /etc/ssl/certs/ca-certificates.crt # /etc/ssl/certs/ca-bundle.crt for Amazon Linux Worker Nodes
              readOnly: true
          imagePullPolicy: "Always"
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop:
                - ALL
            readOnlyRootFilesystem: true
      volumes:
        - name: ssl-certs
          hostPath:
            path: "/etc/ssl/certs/ca-bundle.crt"
```


# 7 Using Mixed Instances Policies and Spot Instances

https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-properties-autoscaling-autoscalinggroup-mixedinstancespolicy.html

**NOTE:** The minimum version of cluster autoscaler to support MixedInstancePolicy is v1.14.x.

If your workloads can tolerate interruption, consider taking advantage of Spot Instances for a lower price point.

To enable diversity among On Demand and Spot Instances, as well as specify multiple EC2 instance types in order to tap into multiple Spot capacity pools, use a [mixed instances policy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-autoscaling-autoscalinggroup-mixedinstancespolicy.html) on your ASG. ==Note that the instance types should have the same amount of RAM and number of CPU cores, since this is fundamental to CA's scaling calculations==.  注意：你所指定的实例类型**应具有相同的内存和 CPU 核心数量**，因为这是 Cluster Autoscaler 执行扩缩容计算的基础.   Using mismatched instances types can produce unintended results. See an example below.

![](image/Pasted%20image%2020250727173338.png)


for example:

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: beta.kubernetes.io/instance-type
                operator: In
                values:
                  - r5.2xlarge
                  - r5d.2xlarge
                  - r5a.2xlarge
                  - r5ad.2xlarge
                  - r5n.2xlarge
                  - r5dn.2xlarge
                  - r4.2xlarge
                  - i3.2xlarge
```



1 nodeSelector 不太合适
还有其他会影响扩缩容行为的因素，例如 节点标签（node labels）。如果你正在使用 nodeSelector 并依赖 beta.kubernetes.io/instance-type 标签来调度 Pod，那么你应该为 ASG 添加一个统一传播的标签来替代它，因为这个 instance-type 标签不再可靠。
例如，如果你的 ASG 中有多个不同的实例类型，这个标签每个节点都不一样，就无法作为统一调度依据。
Additionally, there are other factors which affect scaling, such as node labels. If you are currently using `nodeSelector` with the
[beta.kubernetes.io/instance-type](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#interlude-built-in-node-labels) label, you will need to apply a common propagating label to the ASG and use that instead, since the instance-type label can no longer be relied upon. One may also use auto-generated tags such as `aws:cloudformation:stack-name` for this purpose. 

2 nodeAffinity 合适 
而 节点亲和性（nodeAffinity）和反亲和性（anti-affinity） 不会受这个问题影响，因为它们本身就支持多个值。
[Node affinity and anti-affinity](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity) are not affected in the same way, since these selectors natively accept multiple values; one must add all the configured instances types to the list of values


3 --balancing-label 参数
同样地，如果你使用了 --balancing-label 参数（用于在节点间做负载均衡），你必须确保使用的标签在该节点组的所有节点上具有相同的值。否则，由于 ASG 中的不同实例可能会拥有不同的标签值，就会导致 **Cluster Autoscaler 行为异常**。
Similarly, if using the `balancing-label` flag, you should only choose labels which have the same value for all nodes in the node group.  Otherwise you may get unexpected results, as the flag values will vary based on the nodes created by the ASG.

## 7.1 Example usage

- Create a [Launch  Template](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-autoscaling-autoscalinggroup-launchtemplate.html) (LT) with an instance type, for example, r5.2xlarge. Consider this the 'base' instance type. Do not define any spot purchase options here.
    - 此模板会定义启动脚本（userData）、标签、IAM 角色、网络设置等。
- Create an ASG with a `MixedInstancesPolicy` that refers to the newly-created LT.
    - 使用刚才创建的 Launch Template，来创建一个带有 **MixedInstancesPolicy（混合实例策略）** 的 Auto Scaling Group。
    - 这个策略允许你在一个 ASG 中同时运行不同类型和定价模型（On-Demand + Spot）的实例。
- Set `LaunchTemplateOverrides` in `MixedInstancesPolicy`  to include the 'base' instance type r5.2xlarge and suitable alternatives, e.g. r5d.2xlarge, i3.2xlarge, r5a.2xlarge and r5ad.2xlarge. Differing processor types and speeds should be evaluated depending on your use-case(s).
    - 在 MixedInstancesPolicy 中，通过 `LaunchTemplateOverrides` 设置多个候选的 EC2 实例类型. 这些实例应当具有相似的 **vCPU 和内存**（也就是容量等价），否则 Cluster Autoscaler 可能计算不准确。 你可以选择不同 CPU 架构（Intel、AMD、Graviton），但要考虑性能差异是否影响你的业务。
- Set [InstancesDistribution](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_InstancesDistribution.html) in `MixedInstancesPolicy` according to your needs.
    - `InstancesDistribution` 控制 Spot 与 On-Demand 实例的比例：
        - 比如你可以设置 `OnDemandPercentageAboveBaseCapacity: 20`，意味着 20% 的新实例是按需购买，80% 是 Spot。
    - 还可以设置其他策略，比如希望优先使用 Spot、最多尝试几个替代类型等等。
- See capacity [Allocation Strategies](https://docs.aws.amazon.com/autoscaling/ec2/userguide/asg-purchase-options.html#asg-allocation-strategies) in `MixedInstancesPolicy`
    - for information about how the ASG fulfils capacity from the specified instance types. It is recommended to use the capacity-optimized allocation strategy, which will automatically launch Spot Instances into the most available pools by looking at real-time capacity data and.
    - `lowest-price`：选最便宜的
    - `capacity-optimized`：优先选择容量最多（最稳定）的 Spot 实例池
    - 推荐使用 `capacity-optimized`，因为 Spot 实例可能突然中断，而这个策略能更智能地选择当前“容量最充足、被抢占概率最小”的实例池。
- 使用多个 ASG 分散风险 & 提高弹性
    - For the same workload or for the generic capacity in your cluster, you can also create more node groups with a vCPU/Mem ratio that is a good fit for your workloads, but from different instance sizes. 
    - For example: Node group 1: m5.xlarge, m5a.xlarge, m5d.xlarge, m5ad.xlarge, m4.xlarge. Node group 2: m5.2xlarge, m5a.2xlarge, m5d.2xlarge, m5ad.2xlarge, m4.2xlarge. 
    - This approach increases the chance of achieving your desired scale at the lowest cost by tapping into many Spot capacity pools.



| 步骤  | 内容                            | 目的                  |
| --- | ----------------------------- | ------------------- |
| 1   | 创建基础 Launch Template          | 提供统一配置起点            |
| 2   | 创建 MixedInstancesPolicy 的 ASG | 实现实例混合部署            |
| 3   | 设置 Overrides 类型列表             | 拓展实例可选范围            |
| 4   | 设置 InstancesDistribution      | 控制 Spot/OnDemand 比例 |
| 5   | 使用 capacity-optimized 策略      | 提高稳定性，降低中断风险        |
| 6   | 多 node group + 多实例大小          | 提高调度成功率，提升弹性与性价比    |


---

See CloudFormation example [here](MixedInstancePolicy.md).
The following is an excerpt from a CloudFormation template showing how a MixedInstancesPolicy can be used with ClusterAutoscaler:


```
{
    "Resources": {
        "LaunchTemplate": {
            "Type": "AWS::EC2::LaunchTemplate",
            "Properties": {
                "LaunchTemplateName": "memory-opt-2xlarge",
                "LaunchTemplateData": {
                    "InstanceType": "r5.2xlarge"
                }
            }
        },
        "ASGA": {
            "Type": "AWS::AutoScaling::AutoScalingGroup",
            "Properties": {
                "MinSize": 1,
                "MaxSize": 10,
                "MixedInstancesPolicy": {
                    "InstancesDistribution": {
                        "OnDemandBaseCapacity": 0,
                        "OnDemandPercentageAboveBaseCapacity": 0
                    },
                    "LaunchTemplate": {
                        "LaunchTemplateSpecification": {
                            "LaunchTemplateId": {
                                "Ref": "LaunchTemplate"
                            },
                            "Version": {
                                "Fn::GetAtt": [
                                    "LaunchTemplate",
                                    "LatestVersionNumber"
                                ]
                            }
                        },
                        "Overrides": [
                            {
                                "InstanceType": "r5.2xlarge"
                            },
                            {
                                "InstanceType": "r5d.2xlarge"
                            },
                            {
                                "InstanceType": "i3.2xlarge"
                            },
                            {
                                "InstanceType": "r5a.2xlarge"
                            },
                            {
                                "InstanceType": "r5ad.2xlarge"
                            }
                        ]
                    }
                },
                "VPCZoneIdentifier": [
                    "subnet-###############"
                ],
            }
        },
        "ASGB": {},
        "ASGC": {}
    }
}
```

[r5.2xlarge](https://aws.amazon.com/ec2/instance-types/#Memory_Optimized) is the 'base' instance type, with overrides for r5d.2xlarge, i3.2xlarge, r5a.2xlarge and r5ad.2xlarge.

Note how one Auto Scaling Group is created per Availability Zone, since CA does not currently support ASGs that span multiple Availability Zones. See [Common Notes and Gotchas](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler/cloudprovider/aws#common-notes-and-gotchas).

----

以下是一个完整的 **Terraform 示例**，用于创建一个基于 **MixedInstancesPolicy（混合实例策略）** 的 **Auto Scaling Group (ASG)**，结合多个 EC2 实例类型（含 Spot 实例）并支持 Kubernetes 集群使用，例如 EKS：

1 创建 Launch Template
```yaml
resource "aws_launch_template" "eks_nodes" {
  name_prefix   = "eks-nodes-"
  image_id      = "ami-xxxxxxxx" # EKS Worker AMI，需替换为你的区域对应 AMI ID
  instance_type = "r5.2xlarge"   # 基础实例类型

  user_data = base64encode(<<-EOT
    #!/bin/bash
    set -o xtrace
    /etc/eks/bootstrap.sh ${var.cluster_name} \
      --kubelet-extra-args '--node-labels=node-type=spot,accelerator=nvidia-tesla-k80 --register-with-taints=dedicated=true:NoSchedule'
  EOT
  )

  key_name = "your-key-name"

  network_interfaces {
    associate_public_ip_address = true
    security_groups             = [aws_security_group.eks_nodes.id]
  }

  iam_instance_profile {
    name = aws_iam_instance_profile.eks_nodes.name
  }

  tag_specifications {
    resource_type = "instance"

    tags = {
      Name                                 = "eks-node"
      "k8s.io/cluster-autoscaler/enabled"  = "true"
      "k8s.io/cluster-autoscaler/${var.cluster_name}" = "owned"
    }
  }
}

```


2 创建 Auto Scaling Group（ASG）带 Mixed Instances Policy
```ruby
resource "aws_autoscaling_group" "eks_nodes" {
  name                      = "eks-spot-nodegroup"
  desired_capacity          = 2
  max_size                  = 5
  min_size                  = 1
  vpc_zone_identifier       = var.private_subnet_ids
  health_check_type         = "EC2"
  force_delete              = true

  mixed_instances_policy {
    launch_template {
      launch_template_specification {
        launch_template_id = aws_launch_template.eks_nodes.id
        version            = "$Latest"
      }

      overrides = [
        {
          instance_type = "r5.2xlarge"
        },
        {
          instance_type = "r5a.2xlarge"
        },
        {
          instance_type = "r5d.2xlarge"
        },
        {
          instance_type = "i3.2xlarge"
        },
        {
          instance_type = "r5ad.2xlarge"
        },
      ]
    }

    instances_distribution {
      on_demand_percentage_above_base_capacity = 20
      spot_allocation_strategy                 = "capacity-optimized"
    }
  }

  tag {
    key                 = "Name"
    value               = "eks-spot-node"
    propagate_at_launch = true
  }

  tag {
    key                 = "k8s.io/cluster-autoscaler/enabled"
    value               = "true"
    propagate_at_launch = true
  }

  tag {
    key                 = "k8s.io/cluster-autoscaler/${var.cluster_name}"
    value               = "owned"
    propagate_at_launch = true
  }

  tag {
    key                 = "k8s.io/cluster-autoscaler/node-template/label/node-type"
    value               = "spot"
    propagate_at_launch = true
  }

  tag {
    key                 = "k8s.io/cluster-autoscaler/node-template/taint/dedicated"
    value               = "true:NoSchedule"
    propagate_at_launch = true
  }

  lifecycle {
    create_before_destroy = true
  }
}

```

3 说明 

|部分|说明|
|---|---|
|`user_data`|使用 bootstrap.sh 注册标签和污点|
|`LaunchTemplateOverrides`|定义多个实例类型，以便 Spot 多样性和高可用性|
|`spot_allocation_strategy = "capacity-optimized"`|优先选择容量较大的 Spot 实例池|
|`ASG 标签`|供 Cluster Autoscaler 使用，用于识别和调度|



# 8 Use Static Instance List
Cluster Autoscaler（CA）如何获取 EC2 实例类型列表 以及在网络受限环境下如何处理：

The set of the latest supported EC2 instance types will be fetched by the CA at run time. You can find all the available instance types in the CA logs. If your
network access is restricted such that fetching this set is infeasible, you can specify the command-line flag `--aws-use-static-instance-list=true` to switch the CA back to its original use of a statically defined set. (来强制 CA 使用内置的静态 EC2 实例类型列表，而不再在线请求。)

To refresh static list, please run `go run ec2_instance_types/gen.go` under `cluster-autoscaler/cloudprovider/aws/`.

https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/ec2_instance_types/gen.go

# 9 Using the AWS SDK vendored in the AWS cloudprovider


如何**在 Cluster Autoscaler (CA) 的 AWS Cloudprovider 插件中手动替换 AWS SDK 的版本**

Cluster Autoscaler 的 AWS 插件默认会“内嵌”（vendor）一个固定版本的 AWS SDK（Go 语言版本），目前是： v1.48.7
如果你希望使用 更新版本的 AWS SDK（比如为了支持新特性、新实例类型、修复 bug），你可以手动替换这个 vendored 版本。

If you want to use a newer version of the AWS SDK than the version currently vendored as a direct dependency by Cluster Autoscaler, then you can use the version vendored under this AWS cloudprovider.
The current version vendored is `v1.48.7`.

If you want to update the vendored AWS SDK to a newer version, please make sure of the following:
1. Place the copy of the new desired version of the AWS SDK under the `aws-sdk-go` directory.
2. Remove folders : models and examples. Remove _test.go file `find . -name '*_test.go' -exec rm {}+`
3. Update the import statements within the newly-copied AWS SDK to reference the new paths (e.g., `github.com/aws/aws-sdk-go/aws/awsutil` -> `k8s.io/autoscaler/cluster-autoscaler/cloudprovider/aws/aws-sdk-go/aws/awsutil`). You can use this command from the aws-sdk-go folder `find . -type f -exec sed -i ‘s#github.com/aws/aws-sdk-go#k8s.io/autoscaler/cluster-autoscaler/cloudprovider/aws/aws-sdk-go#’ {} \;`
4. Update the version number above to indicate the new vendored version.

# 10 Using cloud config with helm

这段文档说明了**如何通过 Helm 为 Kubernetes 的 Cluster Autoscaler 加载自定义 AWS cloud 配置（cloud.conf）文件**。这在你希望配置自定义 endpoint（如使用 AWS 中国区、GovCloud 或私有 cloud）时非常有用。

WS 的 cloud.conf 文件是在 Kubernetes 中使用 AWS 作为云提供商（Cloud Provider）时提供必要配置的文件。它告诉 Kubernetes 如何与 AWS API 通信，并配置诸如区域、VPC、子网、IAM、实例元数据等信息。

这个文件常用于以下组件：
    kube-controller-manager
    cloud-controller-manager
    cluster-autoscaler


 示例：cloud.conf 内容
```ini
[Global]
Zone = us-west-2a
VPC = vpc-0a1234567890abcdef
SubnetID = subnet-0123456789abcdef0
KubernetesClusterTag = kubernetes.io/cluster/my-cluster
DisableSecurityGroupIngress = true
ElbSecurityGroup = sg-0123456789abcdef0
UseInstanceMetadata = true

```


----
If you want to use custom AWS cloud config e.g. endpoint urls

1. Create ConfigMap with cloud config file definition (see [example](examples/configmap-cloudconfig-example.yaml)):
```shell
kubectl apply -f examples/configmap-cloudconfig-example.yaml
```
2. Add the following in your `values.yaml`:
```yaml
cloudConfigPath: config/cloud.conf

extraVolumes:
  - name: cloud-config
    configMap:
      name: cloud-config
extraVolumeMounts:
  - name: cloud-config
    mountPath: config
```

3. Install (or upgrade) helm chart with updated values (see [example](examples/values-cloudconfig-example.yaml))
Please note: it is also possible to mount the cloud config file from host:
```yaml
extraVolumes:
  - name: cloud-config
    hostPath:
      path: /path/to/file/on/host

extraVolumeMounts:
  - name: cloud-config
    mountPath: config/cloud.conf
    readOnly: true
```

# 11 Common Notes and Gotchas:

- The `/etc/ssl/certs/ca-bundle.crt` should exist by default on ec2 instance in your EKS cluster. If you use other cluster provision tools like [kops](https://github.com/kubernetes/kops) with different operating systems other than Amazon Linux 2, please use `/etc/ssl/certs/ca-certificates.crt` or correct path on your host instead for the volume hostPath in your cluster autoscaler manifest.
- If you’re using Persistent Volumes, your deployment needs to run in the same AZ as where the EBS volume is, otherwise the pod scheduling could fail if it is scheduled in a different AZ and cannot find the EBS volume. To overcome this, either use a single AZ ASG for this use case, or an ASG-per-AZ while enabling [--balance-similar-node-groups](../../FAQ.md#im-running-cluster-with-nodes-in-multiple-zones-for-ha-purposes-is-that-supported-by-cluster-autoscaler). Alternately, and depending on your use-case, you might be able to switch from using EBS to using shared storage that is available across AZs (for each pod in its respective AZ). Consider AWS services like Amazon EFS or Amazon FSx for Lustre.
- On creation time, the ASG will have the [AZRebalance process](https://docs.aws.amazon.com/autoscaling/ec2/userguide/auto-scaling-benefits.html#AutoScalingBehavior.InstanceUsage) enabled, which means it will actively work to balance the number of instances
  between AZs, and possibly terminate instances. If your applications could be impacted from sudden termination, you can either suspend the AZRebalance feature, or use a tool for automatic draining upon ASG scale-in such as the [AWS Node Termination Handler](https://github.com/aws/aws-node-termination-handler/).
- By default, cluster autoscaler will not terminate nodes running pods in the kube-system namespace. You can override this default behaviour by passing in the `--skip-nodes-with-system-pods=false` flag.
- By default, cluster autoscaler will wait 10 minutes between scale down operations, you can adjust this using the `--scale-down-delay-after-add`,
  `--scale-down-delay-after-delete`, and `--scale-down-delay-after-failure` flag. E.g. `--scale-down-delay-after-add=5m` to decrease the scale down delay to 5 minutes after a node has been added.
- If you're running multiple ASGs, the `--expander` flag supports five options: `random`, `most-pods`, `least-waste`, `priority`, and `grpc`. `random` will
  expand a random ASG on scale up. `most-pods` will scale up the ASG that will schedule the most amount of pods. `least-waste` will expand the ASG that will waste the least amount of CPU/MEM resources. In the event of a tie, cluster autoscaler will fall back to`random`.  The `priority` expander lets you define a custom priority ranking in a ConfigMap for selecting ASGs, and the `grpc` expander allows you to write your own expansion logic.
- If you're managing your own kubelets, they need to be started with the `--provider-id` flag. The provider id has the format
  `aws:///<availability-zone>/<instance-id>`, e.g. `aws:///us-east-1a/i-01234abcdef`.
- If you want to use regional STS endpoints (e.g. when using VPC endpoint for  STS) the env `AWS_STS_REGIONAL_ENDPOINTS=regional` should be set.
- If you want to run it on instances with IMDSv1 disabled make sure your EC2 launch configuration has the setting `Metadata response hop limit` set to `2`. Otherwise, the `/latest/api/token` call will timeout and result in an error. See [AWS docs here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html#configuring-instance-metadata-options) for further information.
- If you don't use EKS managed nodegroups, don't add the `eks:nodegroup-name` tag to the ASG as this will lead to extra EKS API calls that could slow down scaling when there are 0 nodes in the nodegroup.
- Set `AWS_MAX_ATTEMPTS` to configure max retries
- If you are running a private cluster in a VPC without certain VPC interfaces for AWS services, the CA might crash on startup due to failing to dynamically fetch supported EC2-instance types. To avoid this, add the argument `--aws-use-static-instance-list=true` to the CA startup command. For more information on private cluster requirements, see [AWS docs here](https://docs.aws.amazon.com/eks/latest/userguide/private-clusters.html).


---

- **CA 需要 CA 证书路径：**
    - 在 EKS EC2 节点上，默认应该有 `/etc/ssl/certs/ca-bundle.crt`。
    - 如果你用的是 kops 等其他集群部署工具，且操作系统不是 Amazon Linux 2，路径可能不同（如 `/etc/ssl/certs/ca-certificates.crt`）。
    - 需要根据实际操作系统调整 `hostPath` 挂载路径，确保证书可用。
- **Persistent Volumes (PV) 与可用区 (AZ) 的匹配：**
    - EBS 卷只能在对应的 AZ 使用，Pod 必须调度到相同 AZ 的节点，否则无法挂载 PV。
    - 解决方案：
        - 使用单 AZ 的 ASG（Auto Scaling Group）部署
        - 或者为每个 AZ 创建独立的 ASG，并启用 `--balance-similar-node-groups` 参数，保证调度和扩缩容合理
        - 也可以考虑用跨 AZ 的共享存储服务，比如 Amazon EFS 或 Amazon FSx。
- **ASG 的 AZRebalance 进程：**
    - ASG 默认启用 AZRebalance，会自动平衡实例在各 AZ 之间的数量，有可能会终止部分实例。
    - 如果应用不能接受突然被终止，建议暂停该功能，或者配合自动优雅下线工具，比如 [AWS Node Termination Handler](https://github.com/aws/aws-node-termination-handler/)。
- **系统命名空间 Pod 不会被 CA 删除：**
    - 默认情况下，CA 不会删除运行在 `kube-system` 命名空间的 Pod，避免系统组件被误删。
    - 你可以通过参数 `--skip-nodes-with-system-pods=false` 来覆盖这一行为（让 CA 也能删除这类节点）。
- **节点缩容延迟：**
    - 默认 CA 在缩容操作之间会等待 10 分钟，避免过于频繁的节点变动。
    - 可通过以下参数调整时间：
        - `--scale-down-delay-after-add` （新增节点后延迟）
        - `--scale-down-delay-after-delete` （删除节点后延迟）
        - `--scale-down-delay-after-failure` （失败后延迟）
- **多个 ASG 时的扩容策略：**  
    `--expander` 参数支持以下策略：
    - `random`：随机选择一个 ASG 扩容
    - `most-pods`：扩容能调度最多 Pod 的 ASG
    - `least-waste`：扩容资源浪费最少的 ASG
    - `priority`：自定义优先级（需通过 ConfigMap 配置）
    - `grpc`：自定义扩容逻辑（自定义开发）  
        默认碰到同等情况会回退到 `random`。
- **kubelet 需要正确设置 Provider ID：**
    - kubelet 启动时需带 `--provider-id` 参数，格式如：  
        `aws:///<availability-zone>/<instance-id>`
    - 例子：`aws:///us-east-1a/i-01234abcdef`
- **使用区域性 STS 端点：**
    - 如果使用 VPC Endpoint 访问 STS，需要设置环境变量：  
        `AWS_STS_REGIONAL_ENDPOINTS=regional`
- **IMDSv1 被禁用时的配置：**
    - 如果 EC2 实例禁用了 IMDSv1，只开启 IMDSv2，且 CA 访问实例元数据服务会失败。
    - 需确保 EC2 的 `Metadata response hop limit` 设置为 `2`，以支持 CA 正确调用元数据服务。
- **不要给非 EKS 托管节点组的 ASG 添加 `eks:nodegroup-name` 标签：**
    - 否则会引起 EKS 额外的 API 调用，导致扩缩容时延迟增加，特别是当节点组为空时。
- **静态实例列表和私有集群限制：**
    - 如果你的集群是私有的，且 VPC 没有某些 AWS 服务的接口，CA 可能启动失败，因为它无法动态拉取支持的 EC2 实例类型列表。
    - 解决方法是在启动 CA 时添加参数：  
        `--aws-use-static-instance-list=true`
    - 这样 CA 会使用内置的静态实例列表避免请求失败。



