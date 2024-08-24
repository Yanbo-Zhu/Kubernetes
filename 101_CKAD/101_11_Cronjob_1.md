

# 1 题设

```
Quick Reference
Cluste/配置环境 k8s
Namespace default
您必须切换到正确的 Cluster/配置环境。不这样做可能导致零分。
[candidate@node-1] $ kubectl config use-context k8s

Task
1、创建一个名为 ppi 并执行一个运行以下单一容器的 Pod 的 CronJob：
- name: pi
 image: perl:5
 command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]

配置 CronJob 为：
⚫ 每隔 5 分钟执行一次
⚫ 保留 2 个已完成的 Job
⚫ 保留 4 个失败的 Job
⚫ 永不重启 Pod
⚫ 在 8 秒后终止 Pod

2、为测试目的，从 CronJob ppi 中手动创建并执行一个名为 ppi-test 的 Job。
```


# 2 参考 

[![CKAD考试题解析](https://www.ljh.cool/wp-content/uploads/2023/02/image-49.png)](https://www.ljh.cool/wp-content/uploads/2023/02/image-49.png)

[![CKAD考试题解析](https://www.ljh.cool/wp-content/uploads/2023/02/image-50.png)](https://www.ljh.cool/wp-content/uploads/2023/02/image-50.png)

[![CKAD考试题解析](https://www.ljh.cool/wp-content/uploads/2023/02/image-51.png)](https://www.ljh.cool/wp-content/uploads/2023/02/image-51.png)

[![CKAD考试题解析](https://www.ljh.cool/wp-content/uploads/2023/02/image-52.png)](https://www.ljh.cool/wp-content/uploads/2023/02/image-52.png)

# 3 解题



1 
vicronjob-1.yaml

```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: ppi
spec:
  schedule: "*/5 * * * *" #CronJob 的运行时间表，根据题目要求修改。这里是每 5 分钟！
  successfulJobsHistoryLimit: 2
  failedJobsHistoryLimit: 4
  jobTemplate:
    spec:
      activeDeadlineSeconds: 8
      template:
        spec:
          containers:
          - name: pi
            image: perl:5
            imagePullPolicy: IfNotPresent
            command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
          restartPolicy: Never #这里记得修改，永不重启 Pod 为 Never
```

```
*    *    *    *    *  
-    -    -    -    -  
|    |    |    |    |  
|    |    |    |    +----- 星期中星期几 (0 - 6) (星期天 为0)  
|    |    |    +---------- 月份 (1 - 12)   
|    |    +--------------- 一个月中的第几天 (1 - 31)  
|    +-------------------- 小时 (0 - 23)  
+------------------------- 分钟 (0 - 59)
```


2 创建 CronJob
kubectl apply -f cronjob-1.yaml

3 检查 CronJob
kubectl get cronjob

[![CKAD考试题解析](https://www.ljh.cool/wp-content/uploads/2023/02/image-53.png)](https://www.ljh.cool/wp-content/uploads/2023/02/image-53.png)

```
NAME   SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
ppi    */5 * * * *   False     0        30s             44s
```

4 手动触发一个 job of cronjob   
kubectl create job ppi-test --from=cronjob/ppi  

kubectl get jobs  
NAME                   COMPLETIONS   DURATION   AGE  
ppi-test               0/1           56s        56s
