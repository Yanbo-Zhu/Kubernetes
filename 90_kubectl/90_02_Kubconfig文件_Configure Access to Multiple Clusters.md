

https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/

https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/



A file that is used to configure access to clusters is called a kubeconfig file. This is a generic way of referring to configuration files. It does not mean that there is a file named kubeconfig.

==默认使用 名字为 config 的文件 in$HOME/.kube, ==
如果要特意使用某个 file 需要通过 kubeconfig environment 给出 或者 用 --kubeconfig flag. 指出 
By default, kubectl looks for a file named config in the $HOME/.kube directory. You can specify other kubeconfig files by setting the KUBECONFIG environment variable or by setting the --kubeconfig flag.



# 1 创立kubectl config 文件

1. Create a directory .kube (mind the dot) in the home directory of the user you want to grant control over the cluster.  
    1. Linux home is `/home/<username> `by default, or /root for root.  
        1. `/root/.kube/config`
    2. Windows home is `C:\Users\<username>` by default.
        1. `c:\Users\yzh\.kube\config`



# 2 查看某个 kubeconfig 文件中的内容 


kubectl config --kubeconfig=config-demo view



see only the configuration information associated with the current context
```
kubectl config --kubeconfig=config-demo view --minify
```


# 3 Define clusters, users, and contexts


Create a directory named `config-exercise.` In your config-exercise directory, create a file named `config-demo `with this content:

```
apiVersion: v1
kind: Config
preferences: {}

clusters:
- cluster:
  name: development
- cluster:
  name: test

users:
- name: developer
- name: experimenter

contexts:
- context:
  name: dev-frontend
- context:
  name: dev-storage
- context:
  name: exp-test
```


## 3.1 新增cluster 

Go to your config-exercise directory.  执行下面的命令 
```
kubectl config --kubeconfig=config-demo set-cluster development --server=https://1.2.3.4 --certificate-authority=fake-ca-file
kubectl config --kubeconfig=config-demo set-cluster test --server=https://5.6.7.8 --insecure-skip-tls-verify
```
1 首先 要进入到 config-demo这个文件所在的文件夹 下 执行 kubectl config 
2 --kubeconfig 给出文件名 
3 set-cluster 指定  config-demo文件中定义的某个cluster的名字 
4 --server 在那个server 上面跑 


kubeconfig 文件的内容变成
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority: fake-ca-file
    server: https://1.2.3.4
  name: development
- cluster:
    insecure-skip-tls-verify: true
    server: https://5.6.7.8
  name: test
```



## 3.2 加入某个user登录这个cluster 需要credential

```
kubectl config --kubeconfig=config-demo set-credentials developer --client-certificate=fake-cert-file --client-key=fake-key-seefile
kubectl config --kubeconfig=config-demo set-credentials experimenter --username=exp --password=some-password
```

```
users:
- name: developer
  user:
    client-certificate: fake-cert-file
    client-key: fake-key-file
- name: experimenter
  user:
    # Documentation note (this comment is NOT part of the command output).
    # Storing passwords in Kubernetes client config is risky.
    # A better alternative would be to use a credential plugin
    # and store the credentials separately.
    # See https://kubernetes.io/docs/reference/access-authn-authz/authentication/#client-go-credential-plugins
    password: some-password
    username: exp

```

## 3.3 在kubeconfig文件中 删除一些信息

- To delete a user you can run `kubectl --kubeconfig=config-demo config unset users.<name>`
- To remove a cluster, you can run `kubectl --kubeconfig=config-demo config unset clusters.<name>`
- To remove a context, you can run `kubectl --kubeconfig=config-demo config unset contexts.<name>`


## 3.4 Base64-encoded for credentials 

The `fake-ca-file, fake-cert-file and fake-key-file `above are the placeholders for the pathnames of the certificate files. You need to change these to the actual pathnames of certificate files in your environment.

Sometimes you may want to use Base64-encoded data embedded here instead of separate certificate files; in that case you need to add the suffix -data to the keys, for example,` certificate-authority-data, client-certificate-data, client-key-data`

## 3.5 context

### 3.5.1 在kubeconfig文件中加入context项

kubectl config --kubeconfig=config-demo set-context dev-frontend --cluster=development --namespace=frontend --user=developer
kubectl config --kubeconfig=config-demo set-context dev-storage --cluster=development --namespace=storage --user=developer
kubectl config --kubeconfig=config-demo set-context exp-test --cluster=test --namespace=default --user=experimenter

--cluster=development --namespace=frontend --user=developer 都写要写入到 context dev-frontend 的值

得到 kubeconfig文件中现在的内容

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority: fake-ca-file
    server: https://1.2.3.4
  name: development
- cluster:
    insecure-skip-tls-verify: true
    server: https://5.6.7.8
  name: test
contexts:
- context:
    cluster: development
    namespace: frontend
    user: developer
  name: dev-frontend
- context:
    cluster: development
    namespace: storage
    user: developer
  name: dev-storage
- context:
    cluster: test
    namespace: default
    user: experimenter
  name: exp-test
current-context: ""
kind: Config
preferences: {}
users:
- name: developer
  user:
    client-certificate: fake-cert-file
    client-key: fake-key-file
- name: experimenter
  user:
    # Documentation note (this comment is NOT part of the command output).
    # Storing passwords in Kubernetes client config is risky.
    # A better alternative would be to use a credential plugin
    # and store the credentials separately.
    # See https://kubernetes.io/docs/reference/access-authn-authz/authentication/#client-go-credential-plugins
    password: some-password
    username: exp
```



### 3.5.2 使用指定使用某个已经在kubeconfig中定义好的 context

```
kubectl config --kubeconfig=config-demo use-context dev-frontend
```
Now whenever you enter a kubectl command, the action will apply to the cluster, and namespace listed in the dev-frontend context. And the command will use the credentials of the user listed in the dev-frontend context.

这样的话 使用config 的时候  就自动使用 这个context 中对应的 (cluster, user, namespace)
也会使用这个 clutser 的 namespace 以及 credentials of the user listed in this context.






### 3.5.3 将默认的current context 写入kubeconfig 中 


```
current-context: ""
```


### 3.5.4 see only the configuration information associated with the current context

```
kubectl config --kubeconfig=config-demo view --minify
```


The output shows configuration information associated with the `dev-frontend` context:

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority: fake-ca-file
    server: https://1.2.3.4
  name: development
contexts:
- context:
    cluster: development
    namespace: frontend
    user: developer
  name: dev-frontend
current-context: dev-frontend
kind: Config
preferences: {}
users:
- name: developer
  user:
    client-certificate: fake-cert-file
    client-key: fake-key-file
```


# 4 Proxy
You can configure `kubectl` to use a proxy per cluster using `proxy-url` in your kubeconfig file, like this:

```yaml
apiVersion: v1
kind: Config

clusters:
- cluster:
    proxy-url: http://proxy.example.org:3128
    server: https://k8s.example.org/k8s/clusters/c-xxyyzz
  name: development

users:
- name: developer

contexts:
- context:
  name: development
```

# 5 Create a second configuration file


`c:\Users\yzh\.kube\`
下可以放多个 kubeconfig 文件, 必须是不同的名字 
标准的名字是 config 
再来一个 file 可以是config-e20-d2034, 可以使 config-demo

```
apiVersion: v1
kind: Config
preferences: {}

contexts:
- context:
    cluster: development
    namespace: ramp
    user: developer
  name: dev-ramp-up
```


要去使用这个 新的 configuration file 


```
kubectl config --kubeconfig=config-demo use-context dev-ramp-up
```


# 6 KUBECONFIG environment variable


为了不每次 都要 先切换到  `c:\Users\yzh\.kube\config` 下, kubectl 才能找到对应的 config 文件 

如果不设置 KUBECONFIG environment variable, 则 总会默认使用 `$HOME/.kube/config` 这一个 kube configuration file  , use the default kubeconfig file, `$HOME/.kube/config`, with no merging.

## 6.1 Windows 

就要将 `c:\Users\yzh\.kube\config;c:\Users\yzh\.kube\config-demo` 写入到环境变量 KUBECONFIG

不要写成  `$HOME\.kube\config`  没有用 , `$HOME` 不会被自动翻译的 

查看 KUBECONFIG 中的值 
```
run 
$Env:KUBECONFIG
```


通过 powershell 的命令 将新的路径 添加到 KUBECONFIG这个环境变量中 

`$Env:KUBECONFIG=("config-demo;config-demo-2")`
`$Env:KUBECONFIG="$Env:KUBECONFIG;$HOME\.kube\config"`


## 6.2 Linux 

```
export KUBECONFIG="${KUBECONFIG}:config-demo:config-demo-2"

export KUBECONFIG="${KUBECONFIG}:${HOME}/.kube/config"
```


## 6.3 查看多个kubeconig文件合并的效果 

kubectl config view

The output shows merged information from all the files listed in your KUBECONFIG environment variable. \
In particular, notice that the merged information has the dev-ramp-up context from the config-demo-2 file and the three contexts from the config-demo file

```
contexts:
- context:
    cluster: development
    namespace: frontend
    user: developer
  name: dev-frontend
- context:
    cluster: development
    namespace: ramp
    user: developer
  name: dev-ramp-up
- context:
    cluster: development
    namespace: storage
    user: developer
  name: dev-storage
- context:
    cluster: test
    namespace: default
    user: experimenter
  name: exp-test
```


# 7 Merging kubeconfig files rules 

https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/

As described previously, the output might be from a single kubeconfig file, or it might be the result of merging several kubeconfig files.

Here are the rules that `kubectl` uses when it merges kubeconfig files:

1. If the `--kubeconfig` flag is set, use only the specified file. Do not merge. Only one instance of this flag is allowed.
    
    Otherwise, if the `KUBECONFIG` environment variable is set, use it as a list of files that should be merged. Merge the files listed in the `KUBECONFIG` environment variable according to these rules:
    
    - Ignore empty filenames.
    - Produce errors for files with content that cannot be deserialized.
    - The first file to set a particular value or map key wins.
    - Never change the value or map key. Example: Preserve the context of the first file to set `current-context`. Example: If two files specify a `red-user`, use only values from the first file's `red-user`. Even if the second file has non-conflicting entries under `red-user`, discard them.
    
    For an example of setting the `KUBECONFIG` environment variable, see [Setting the KUBECONFIG environment variable](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/#set-the-kubeconfig-environment-variable).
    
    Otherwise, use the default kubeconfig file, `$HOME/.kube/config`, with no merging.
    
2. Determine the context to use based on the first hit in this chain:
    
    1. Use the `--context` command-line flag if it exists.
    2. Use the `current-context` from the merged kubeconfig files.
    
    An empty context is allowed at this point.
    
3. Determine the cluster and user. At this point, there might or might not be a context. Determine the cluster and user based on the first hit in this chain, which is run twice: once for user and once for cluster:
    
    1. Use a command-line flag if it exists: `--user` or `--cluster`.
    2. If the context is non-empty, take the user or cluster from the context.
    
    The user and cluster can be empty at this point.
    
4. Determine the actual cluster information to use. At this point, there might or might not be cluster information. Build each piece of the cluster information based on this chain; the first hit wins:
    
    1. Use command line flags if they exist: `--server`, `--certificate-authority`, `--insecure-skip-tls-verify`.
    2. If any cluster information attributes exist from the merged kubeconfig files, use them.
    3. If there is no server location, fail.
5. Determine the actual user information to use. Build user information using the same rules as cluster information, except allow only one authentication technique per user:
    
    1. Use command line flags if they exist: `--client-certificate`, `--client-key`, `--username`, `--password`, `--token`.
    2. Use the `user` fields from the merged kubeconfig files.
    3. If there are two conflicting techniques, fail.
6. For any information still missing, use default values and potentially prompt for authentication information

