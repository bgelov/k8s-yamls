# k8s-yamls
Some kubernetes yaml files in this repository

# Cheat sheet
More kubernetes cheatsheet: https://kubernetes.io/ru/docs/reference/kubectl/cheatsheet/
## k8s autocomplete

```
sudo apt-get install bash-completion
source <(kubectl completion bash) # настройка автодополнения в текущую сессию bash, предварительно должен быть установлен пакет bash-completion .
echo "source <(kubectl completion bash)" >> ~/.bashrc # добавление автодополнения autocomplete постоянно в командную оболочку bash.
```

## k8s alias

```
alias k=kubectl
complete -F __start_kubectl k
```