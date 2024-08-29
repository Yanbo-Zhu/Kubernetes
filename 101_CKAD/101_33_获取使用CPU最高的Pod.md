
# 1 题设

```
Quick Reference
Cluster/配置环境 k8s
Namespace cpu-stress

您必须切换到正确的 Cluster/配置环境。不这样做可能导致零分。
[candidate@node-1] $ kubectl config use-context k8s

Task
监控在 namespace cpu-stress 中运行的 Pod，
并将消耗最多 CPU 的 Pod 的名称写入文件 /ckad/CKAD00010/pod.txt
注意：文件 /ckad/CKAD00010/pod.txt 已存在
```



# 2 解题 


kubectl top pod --sort-by="cpu" -n cpu-stress

echo redis-test > /ckad/CKAD00010/pod.txt

[![CKAD考试题解析](https://www.ljh.cool/wp-content/uploads/2023/02/image-106.png)](https://www.ljh.cool/wp-content/uploads/2023/02/image-106.png)
