# Deploy and Access the Kubernetes Dashboard

- Last version on: 
https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/


```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

Get token
```
kubectl -n kubernetes-dashboard create token admin-user
```

Start dashboart
```
kubectl proxy
```

Dashboard link: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
