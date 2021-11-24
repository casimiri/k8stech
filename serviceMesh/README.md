# Service Mesh

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

#### Ingress IP and PORT(AKS)
```
kubectl get svc istio-ingressgateway -n istio-system
```
```
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
```
#### Gateway URL
```
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
```
### Verify external access
```
echo "http://$GATEWAY_URL/productpage"
```

#### Dashboard
```
kubectl apply -f samples/addons
```
#### Access Kiali dashboard
```
istioctl dashboard kiali
```
##### Send data
```
watch -n 1 curl -o /dev/null -s -w %{http_code} "http://$GATEWAY_URL/productpage"
```


```
for i in $(seq 1 100); do curl -s -o /dev/null "http://$GATEWAY_URL/productpage"; done
```
## Install argo rollouts
### Install argo rollout CRDs and Controllers
```
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml
```
### Install argo rollouts kubectl plugin
```
brew install argoproj/tap/kubectl-argo-rollouts
```

### Deploy initial rollout and service
```
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-rollouts/master/docs/getting-started/basic/rollout.yaml
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-rollouts/master/docs/getting-started/basic/service.yaml
```
### To watch the rollout 
```
kubectl argo rollouts get rollout rollouts-demo --watch
```
### Updating a rollout
```
kubectl argo rollouts set image rollouts-demo \
  rollouts-demo=argoproj/rollouts-demo:yellow
```
### Manually Promoting a rollout
```
kubectl argo rollouts promote rollouts-demo
```
### Aborting a rollout
```
kubectl argo rollouts set image rollouts-demo \
  rollouts-demo=argoproj/rollouts-demo:red

kubectl argo rollouts abort rollouts-demo

# to make the status Healthy instead of Degraded
kubectl argo rollouts set image rollouts-demo \
  rollouts-demo=argoproj/rollouts-demo:yellow
```

## Install Flagger
### Istio with telemetry support and Prometheus
```
istioctl manifest install --set profile=default

kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.8/samples/addons/prometheus.yaml
```

```
kubectl apply -k github.com/fluxcd/flagger//kustomize/istio
```
### BootStrap [Flagger]
```
kubectl create ns test
kubectl label namespace test istio-injection=enabled
```

### Install PodInfo Deployment and Horizontal Pod Autoscaler (HPA)
```
kubectl apply -k https://github.com/fluxcd/flagger//kustomize/podinfo?ref=main
```
### Load testing service to generate traffic
```
kubectl apply -k https://github.com/fluxcd/flagger//kustomize/tester?ref=main
```
### Create Canary custom resource
```YAML
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: podinfo
  namespace: test
spec:
  # deployment reference
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: podinfo
  # the maximum time in seconds for the canary deployment
  # to make progress before it is rollback (default 600s)
  progressDeadlineSeconds: 60
  # HPA reference (optional)
  autoscalerRef:
    apiVersion: autoscaling/v2beta2
    kind: HorizontalPodAutoscaler
    name: podinfo
  service:
    # service port number
    port: 9898
    # container port number or name (optional)
    targetPort: 9898
    # Istio gateways (optional)
    gateways:
    - public-gateway.istio-system.svc.cluster.local
    # Istio virtual service host names (optional)
    hosts:
    - app.example.com
    # Istio traffic policy (optional)
    trafficPolicy:
      tls:
        # use ISTIO_MUTUAL when mTLS is enabled
        mode: DISABLE
    # Istio retry policy (optional)
    retries:
      attempts: 3
      perTryTimeout: 1s
      retryOn: "gateway-error,connect-failure,refused-stream"
  analysis:
    # schedule interval (default 60s)
    interval: 1m
    # max number of failed metric checks before rollback
    threshold: 5
    # max traffic percentage routed to canary
    # percentage (0-100)
    maxWeight: 50
    # canary increment step
    # percentage (0-100)
    stepWeight: 10
    metrics:
    - name: request-success-rate
      # minimum req success rate (non 5xx responses)
      # percentage (0-100)
      thresholdRange:
        min: 99
      interval: 1m
    - name: request-duration
      # maximum req duration P99
      # milliseconds
      thresholdRange:
        max: 500
      interval: 30s
    # testing (optional)
    webhooks:
      - name: acceptance-test
        type: pre-rollout
        url: http://flagger-loadtester.test/
        timeout: 30s
        metadata:
          type: bash
          cmd: "curl -sd 'test' http://podinfo-canary:9898/token | grep token"
      - name: load-test
        url: http://flagger-loadtester.test/
        timeout: 5s
        metadata:
          cmd: "hey -z 1m -q 10 -c 2 http://podinfo-canary.test:9898/"
```
### Save to podinfo-canary.yaml and apply
```
kubectl apply -f ./podinfo-canary.yaml
```
### Trigger Canary by updating the image
```
kubectl -n test set image deployment/podinfo \
podinfod=stefanprodan/podinfo:3.1.1
```

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
 

