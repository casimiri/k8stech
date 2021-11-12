# Knative
## Install knative cli
```
brew install kn
```

## Install Knative serving
### Install Knative core CRDs
```
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.0.0/serving-crds.yaml
```

### Install Knative serving components
```
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.0.0/serving-core.yaml
```
### Install Istio Network layer
Knative Istio controller
```
kubectl apply -f https://github.com/knative/net-istio/releases/download/knative-v1.0.0/net-istio.yaml
```
Fetch External IP or CNAME
```
kubectl --namespace istio-system get service istio-ingressgateway
```
### Verify
```
kubectl get pods -n knative-serving
``

### Configure DNS (Magic DNS with sslip.io)
```
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.0.0/serving-default-domain.yaml
```
### Install HPA (Horizontal Pod Autoscaling) (Optional)
```
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.0.0/serving-hpa.yaml
``

## Install first knative app
```
kn service create hello \
--image gcr.io/knative-samples/helloworld-go \
--port 8080 \
--env TARGET=World \
--revision-name=world
```

