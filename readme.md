# Namespaces Code Examples

## Service deployment across namepaces

Review existing namespaces and create new namespaces 
```
kubectl get namespaces
kubectl create -f namespace-dev.yml
kubectl create -f https://k8s.io/examples/admin/namespace-prod.json
```

Review each of the yml files, notice the differences between each.
```
kubectl create -f react-app-pod.yml
kubectl create -f react-app-pod-DEV.yml
kubectl create -f react-app-pod-PROD.yml
```

Use the following commands to observe which services are running per namespace
```
kubectl get pods -l app=react-app --namespace=development
kubectl get pods --namespace=production 
kubectl get pods -l app=react-app --namespace=default
```

Change namespace from default and verify how this changes perspective
```
kubectl config set-context --current --namespace=production
kubectl get pods
kubectl get pods -l app=react-app --namespace=default
```

Check which namespace has been configured
```
kubectl config get-contexts
```

Review pods across all namespaces
```
kubectl get pods --all-namespaces
```
