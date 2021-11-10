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


