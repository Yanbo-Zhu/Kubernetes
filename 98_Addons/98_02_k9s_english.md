# 1 Install 

https://k9scli.io/topics/install/


# 2 Configration 


error
```
ubuntu@k9s-node:~$ k9s
Command 'k9s' not found, did you mean:
  command 'kas' from deb kas (2.6.3-2)
Try: sudo apt install <deb name>
```

add it in ~/.bashrc  
```
alias k9s=/snap/k9s/current/bin/k9s
```

Then 
source ~/.bashrc 


# 3 Usage 

## 3.1 pre-works

kubectl config use-context arn:aws:eks:eu-central-1:861001182512:cluster/training  

## 3.2 UI 


![](https://i.cloudnative.to/~gitbook/image?url=https%3A%2F%2F2547706445-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-legacy-files%2Fo%2Fassets%252F-MNIMs8Ubg1Uqdpgfw0o%252F-MO9zTNxi9Wxg2V7HfLc%252F-MOAPJYScwIoFVzm7ySC%252Fimage.png%3Falt%3Dmedia%26token%3D861cfd23-379b-47c2-ad29-c8b121265fa3&width=768&dpr=4&quality=100&sign=ce4bc075&sv=1)

## 3.3 Common Key Bindings

Filter out a resource view given a filter : `/ <keyword>`

portfoward: shift-f 

| View a Kubernetes resource using singular/plural or short-name<br><br>用单数/复数或短名来查看 Kubernetes 资源，如 `po`, `dp`, `svc`, `cm`, `sec`, `rb`, etc. | `:`pod⏎               | accepts singular, plural, short-name or alias ie pod or pods |
| --------------------------------------------------------------------------------------------------------------------------------------------- | --------------------- | ------------------------------------------------------------ |
| View a Kubernetes resource in a given namespace                                                                                               | `:`pod namespaceName⏎ |                                                              |


## 3.4 customized Hotkey

1. Windows: `c:\Users\yzh\AppData\Local\k9s\hotkeys.yaml`
2. Add the following to your `hotkeys.yaml`. You can use resource name/short name to specify a command ie same as typing it while in command mode.

用这个: 
```yaml
# $HOME/.k9s/hotkeys.yaml
hotKeys:
  # Hitting Shift-1 navigates to your cluster 
  shift-1:
    shortCut:    Shift-1
    override:    true # => will override the default shortcut related action if set to true (default to false)
    description: View cluster
    command:     ctx
  # Hitting Shift-2 navigates to your nodes 
  shift-2:
    shortCut:    Shift-2
    override:    true # => will override the default shortcut related action if set to true (default to false)
    description: View nodes
    command:     no
  # Hitting Shift-3 navigates to your services 
  shift-3:
    shortCut:    Shift-3
    override:    true # => will override the default shortcut related action if set to true (default to false)
    description: View namespaces
    command:     ns
  # Hitting Shift-4 navigates to your deployments
  shift-4:
    shortCut:    Shift-4
    override:    true # => will override the default shortcut related action if set to true (default to false)
    description: View deployments
    command:     dp
  # Hitting Shift-3 navigates to your pod view
  shift-5:
    shortCut:    Shift-5
    override:    true # => will override the default shortcut related action if set to true (default to false)
    description: Viewing pods
    command:     pods
  # Hitting Shift-6 navigates to your services 
  shift-6:
    shortCut:    Shift-6
    override:    true # => will override the default shortcut related action if set to true (default to false)
    description: View services
    command:     svc
  # Hitting Shift-9 navigates to your endpoints
  shift-7:
    shortCut:    Shift-7
    description: View endpoints
    command:     ep
  # Hitting Shift-0 navigates to your endpoints
  shift-8:
    shortCut:    Shift-8
    description: View configmaps
    command:     cm
  # Hitting Shift-8 navigates to your daemonsets
  shift-9:
    shortCut:    Shift-9
    override:    true # => will override the default shortcut related action if set to true (default to false)
    description: View secret
    command:     sec
  # Hitting Shift-7 navigates to your statefulsets view
  shift-0:
    shortCut:    Shift-0
    override:    true # => will override the default shortcut related action if set to true (default to false)
    description: Viewing statefulsets
    command:     sts
  # Hitting Shift-8 navigates to your daemonsets
  shift-P:
    shortCut:    Shift-P
    override:    true # => will override the default shortcut related action if set to true (default to false)
    description: View daemonsets
    command:     ds
  # Hitting Shift-X navigates to your xray deployments
  shift-X:
    shortCut:    Shift-X
    override:    true # => will override the default shortcut related action if set to true (default to false)
    description: Xray Deployments
    command:     xray deploy
  # Hitting Shift-S view the resources in the namespace of your current selection
  shift-s:
    shortCut:    Shift-S
    override:    true # => will override the default shortcut related action if set to true (default to false)
    description: Namespaced resources
    command:     "$RESOURCE_NAME $NAMESPACE"
    keepHistory: true # whether you can return to the previous view
```
