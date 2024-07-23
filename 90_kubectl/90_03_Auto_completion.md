
https://spacelift.io/blog/kubectl-auto-completion
https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/

```
# Installing bash completion on macOS using homebrew
## If running Bash 3.2 included with macOS
brew install bash-completion
## or, if running Bash 4.1+
brew install bash-completion@2
## If kubectl is installed via homebrew, this should start working immediately
## If you've installed via other means, you may need add the completion to your completion directory
kubectl completion bash > $(brew --prefix)/etc/bash_completion.d/kubectl
  
  
# Installing bash completion on Linux
## If bash-completion is not installed on Linux, install the 'bash-completion' package
## via your distribution's package manager.
## Load the kubectl completion code for bash into the current shell
source <(kubectl completion bash)
## Write bash completion code to a file and source it from .bash_profile
kubectl completion bash > ~/.kube/completion.bash.inc 
printf "
# kubectl shell completion
source '$HOME/.kube/completion.bash.inc'
" >> $HOME/.bash_profile
source $HOME/.bash_profile

# Load the kubectl completion code for zsh[1] into the current shell
source <(kubectl completion zsh)
# Set the kubectl completion code for zsh[1] to autoload on startup
kubectl completion zsh > "${fpath[1]}/_kubectl"


# Load the kubectl completion code for fish[2] into the current shell
kubectl completion fish | source
# To load completions for each session, execute once:
kubectl completion fish > ~/.config/fish/completions/kubectl.fish

# Load the kubectl completion code for powershell into the current shell
kubectl completion powershell | Out-String | Invoke-Expression
# Set kubectl completion code for powershell to run on startup
## Save completion code to a script and execute in the profile
kubectl completion powershell > $HOME\.kube\completion.ps1
Add-Content $PROFILE "$HOME\.kube\completion.ps1"
## Execute completion code in the profile
Add-Content $PROFILE "if (Get-Command kubectl -ErrorAction SilentlyContinue) {
kubectl completion powershell | Out-String | Invoke-Expression
}"
## Add completion code directly to the $PROFILE script
kubectl completion powershell >> $PROFILE
```

# 1 Bash 

## 1.1 Check if bash-completion is already installed:

Check if `bash-completion` is already installed:

```bash
type _init_completion
```

If it is already installed, you will see something like the following:

![kubectl autocompletion - bash-completion](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2022%2F08%2Fkubectl-autocompletion-bash-completion.png&w=3840&q=75)


## 1.2 install bash-completion

If it is not installed, install using apt or yum, depending on which package manager you are using (usually apt for Ubuntu):

```bash
apt-get install bash-completion 
```

or

```bash
yum install bash-completion
```

Set the kubectl completion script source for your shell sessions:



## 1.3 

对于当前user 设置 
echo 'source <(kubectl completion bash)' >>~/.bashrc 
kubectl completion bash >/etc/bash_completion.d/kubectl

对于 全局 设置 
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null
source /usr/share/bash-completion/bash_completion


## 1.4 测试 

Reload your bash shell. Type `kubectl -` followed by pressing tab twice to see the available options and verify auto-complete is working:




## 1.5 Set up an alias for kubectl and enable auto-completion

Set an alias for kubectl as `k`.
```bash
echo 'alias k=kubectl' >>~/.bashrc
```

Enable the alias for auto-completion.
```bash
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
```

Reload your bash shell. Type `k -` followed by pressing tab twice to see the available options and verify auto-complete is working with the alias.

```bash
k -
```

![kubectl autocompletion - Reload your bash shell again](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2022%2F08%2Fkubectl-autocompletion-Reload-your-bash-shell-again.png&w=3840&q=75)

💡 You might also like:


# 2 Powershell

 Load the kubectl completion code for powershell into the current shell
kubectl completion powershell | Out-String | Invoke-Expression


To set up kubectl autocomplete in Windows, you just need to run:

```bash
kubectl completion powershell >> $PROFILE
```



# 3 Mac zsh


To setup kubectl autocomplete, you will need to initially run the following commands:

```bash
mkdir -p ~/.oh-my-zsh/custom/plugins/kubectl-autocomplete/



kubectl completion zsh > ~/.oh-my-zsh/custom/plugins/kubectl-autocomplete/kubectl-autocomplete.plugin.zsh
```

Then, you should edit your zsh profile (~/.zshrc) and add the kubectl autocomplete inside the plugins:

```bash
plugins=(kubectl-autocomplete)
```

## 3.1 How to set up kubectl
