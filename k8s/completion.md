# kubernetes completion（自动补全）

## bash

```bash
kubeadm completion bash > ~/.kube/kubeadm_completion.bash.inc
printf "\n# Kubeadm shell completion\nsource '$HOME/.kube/kubeadm_completion.bash.inc'\n" >> $HOME/.bash_profile
echo "source <(kubectl completion bash)" >> ~/.bash_profile
source $HOME/.bash_profile
```

## zsh

```bash
echo "source <(kubeadm kubectl completion zsh)" >> ~/.zshrc
source ~/.zshrc
```
