# Kubernetes resources
## Cluster information
```
kubectl cluster-info
kubectl version
kubectl get nodes
```
## Run a Pod
```
kubectl run myapp --image rinaahm/k8s_demo --dry-run=client -o yaml > myapp.yaml
kubectl apply -f myapp.yaml
kubectl get pods
kubectl describe pod myapp
kubectl exec -it myapp -- sh
kubectl delete pod myapp
```
## Deployment
```
kubectl create deploy appdeploy --image nginx --replicas=1  --dry-run=client -o yaml > app-deploy.yaml
kubectl apply -f app-deploy.yaml
kubectl get deploy
kubectl scale deploy appdeploy --replicas 5
kubectl scale deploy appdeploy --replicas 2
```
## Service
```
kubectl expose appdeploy --type IPCluster --dry-run=client -o yaml > app-service.yaml
kubectl apply -f app-service.yaml
```

## Container Image

Start with following image:

[rinaahm/k8s_demo](https://hub.docker.com/repository/docker/rinaahm/k8s_demo)
