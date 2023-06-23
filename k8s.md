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

Выгрузить конфигурацию в yaml
kubectl get pod bgelov-app -o yaml

kubectl exec -it bgelov-app -- /bin/bash
kubectl exec -it bgelov-app -- /bin/sh

kubectl get pod bgelov-app -o yaml

Переадресация портов на хост систему для отладки
```
kubectl port-forward bgelov-app 8888:8888
```

Удалить под
kubectl delete po bgelov-app

Удалить под с определённой меткой
kubectl delete po -l app=nginx

# Pod Logs

kubectl logs bgelov-app

Если несколько контейнеров в Pod, то указываем конкретный контейнер
kubectl logs bgelov-app --container bgelov-app

При удалении подов логи не сохраняются. Надо делать общекластерное ведение логов.


# Pod labels
Лейблы могут быть присоединены к любому объекту k8s
Посмотреть все метки
kubectl get pods --show-labels

Посмотреть только с меткой app
kubectl get pods -l app

Посмотреть только с меткой app и environment 
kubectl get pods -L app,environment 

Селекторы
kubectl get pods -l environment=dev
kubectl get pods -l environment!=dev
kubectl get pods -l environment=dev,app=nginx

kubectl get pods -l !environment
kubectl get pod -l "environment in (dev)"
kubectl get pods -l "app notin (nginx)"


Пометить pod
kubectl label pods bgelov-app-2 environment=dev

Пометить node
kubectl label node k8s-cluster-1 gpu=false

# Annotate
Например, чтобы знать кто создал под
kubectl annotate po bgelov-app-gpu-false company-name/creator-email="bgelov"

# Namespaces
Пространства имён
kubectl get ns

kubectl create namespace qa

```
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

Создадим тестовые поды в разных неймспейсах: bgelov-pod-with-namespaces.yaml

kubectl get po --all-namespaces


Удалить неймспейс вместе с подами внутри
kubectl delete ns qa


# Service
Доступ к Pod из вне




