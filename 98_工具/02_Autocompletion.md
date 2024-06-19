
yum -y install bash-completion


自动补全 
echo 'source <(kubectl completion bash)' >>~/.bashrc 
kubectl completion bash >/etc/bash_completion.d/kubectl

全局
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null
source /usr/share/bash-completion/bash_completion



