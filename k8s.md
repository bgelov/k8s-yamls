kubectl version

# Minikube

# Dashboard
https://github.com/bgelov/k8s-yamls/tree/main/web-ui-dashboard
kubectl proxy

# Config
kubectl config get-clusters

kubectl config use-context k8s-cluster-1
kubectl config use-context k8s-cluster-2
kubectl config use-context minikube

# Pods

kubectl get pods --all-namespaces 

kubectl run xxx-app --image=bgelov/xxxx --port 8008
kubectl get pods 


kubectl exec -it xiu-app -- /bin/bash
kubectl exec -it xiu-app -- /bin/sh

kubectl get pod xiu-app -o yaml

Переадресация портов на хост систему для отладки
```
kubectl port-forward xiu-app 1935:1935
```

# Pod Logs

kubectl logs xiu-app

Если несколько контейнеров в Pod, то указываем конкретный контейнер
kubectl logs xiu-app --container xiu-app

При удалении подов логи не сохраняются. Надо делать общекластерное ведение логов.


# Pod labels

kubectl get pods --show-labels


# Service
Доступ к Pod из вне






kubectl get pod xiu-app -o yaml
