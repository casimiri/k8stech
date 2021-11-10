# Service Mesh

## Install Linkerd

### Linkerd installation
```
curl -fsL https://run.linkerd.io/install | sh
```
```
export PATH=$PATH:/Users/kz/.linkerd2/bin
```
```
linkerd version
```
You should see the CLI version, and also Server version: unavailable. 

### Step 2 Validate K8s cluster
```
linkerd check --pre
```
### Step 3 Install linkerd control plane

```
linkerd install | kubectl apply -f -
```
```
linkerd check
```
 

## Install ISTIO
### 1. Install istio cli
```
curl -L https://istio.io/downloadIstio | sh -
```

```
istioctl install --set profile=demo -y
```

### 2. Install Bookinfo sample App
```
kubectl create ns istio-bookinfo
```
Annotate the namespace istio-bookinfo
```
kubectl label namespace istio-bookinfo istio-injection=enabled
```
```
kubectl -n istio-bookinfo apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

```
k -n istio-bookinfo get service
```
```
k -n istio-bookinfo get pod
```
#### Verify everything is working
```
kubectl -n istio-bookinfo exec "$(kubectl get -n istio-bookinfo pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
```
#### Install Gateway
```
kubectl -n istio-bookinfo apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```
#### Ensure there is no issues
```
istioctl -n istio-bookinfo analyze
```
