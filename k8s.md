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


