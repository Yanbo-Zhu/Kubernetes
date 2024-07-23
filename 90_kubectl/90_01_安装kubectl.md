



> 扩展：kubectl可以在node节点上运行吗 ?

kubectl的运行是需要进行配置的，它的配置文件是$HOME/.kube，如果想要在node节点运行此命令，需要将master上的.kube文件复制到node节点上，即在master节点上执行下面操作：

```shell
scp  -r  HOME/.kube   node1: HOME/
```



# 1 install kubectl

1. For Linux, use [https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/), but stop at (excluding) 'Verify kubectl configuration'.
2. For Windows, use [https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/), but stop at (excluding) 'Verify kubectl configuration'.

推荐通过 choco 安装 
- choco install kubernetes-cli

安装完了以后
1. Append or prepend the kubectl binary folder to your PATH environment variable.
2. Test to ensure the version of kubectl is the same as downloaded: `kubectl version --client` and `kubectl version --client --output=yaml`

 随着 Docker Desktop for Windows  已经自动安装了 kubectl 
 ==随意我就没有额外安装了==
` c:\Program Files\Docker\Docker\resources\bin`\
 ![](image/Pasted%20image%2020240723113535.png)

BWS18067 
```
 ⚡ 🦄  kubectl version --client --output=yaml
clientVersion:
  buildDate: "2024-02-14T10:40:49Z"
  compiler: gc
  gitCommit: 4b8e819355d791d96b7e9d9efe4cbafae2311c88
  gitTreeState: clean
  gitVersion: v1.29.2
  goVersion: go1.21.7
  major: "1"
  minor: "29"
  platform: windows/amd64
kustomizeVersion: v5.0.4-0.20230601165947-6ce0bf390ce3
```




