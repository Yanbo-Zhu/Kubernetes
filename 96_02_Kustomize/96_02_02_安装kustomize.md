

# 1 brew 的方式

```
# mac  
brew install kustomize  
# curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
```


# 2 go方式 


```
GOBIN=$(pwd)/ GO111MODULE=on go get sigs.k8s.io/kustomize/kustomize/v3


cd $GOPATH/src/sigs.k8s.io
git clone https://github.com/kubernetes-sigs/kustomize.git
unset GO111MODULES
cd kustomize/kustomize
go install
```

**从源代码安装kustomize CLI，而不克隆存储库**

For `go version` ≥ `go1.17` 对于 `go version` ≥ `go1.17`

```bash
GOBIN=$(pwd)/ GO111MODULE=on go install sigs.k8s.io/kustomize/kustomize/v5@latest
```

> **Note** 除了直接使用 kustomize 命令外，kubernetes 自 v1.14 之后也可以使用 `kubectl kustomize`的方式执行 kustomize

# 3 无需安装

除了直接使用 kustomize 命令外，kubernetes 自 v1.14 之后也可以使用 `kubectl kustomize`的方式执行 kustomize





