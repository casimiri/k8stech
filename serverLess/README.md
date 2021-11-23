# OpenFaaS
## Install arkade
```
curl -sLS https://get.arkade.dev | sudo sh
```

## Install OpenFaaS with arkade
```
arkade install openfaas
```
### Get the faas-cli
```
curl -SLsf https://cli.openfaas.com | sudo sh
```
### Forward Gateway port to local machine
```
kubectl port-forward -n openfaas svc/gateway 8080:8080 &
```
### If basic auth is enabled, you can now log into your gateway:
PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)
echo -n $PASSWORD | faas-cli login --username admin --password-stdin

### e.g Figlet App
faas-cli store deploy figlet

### Template
```
faas-cli template store list
faas-cli template store pull csharp
faas-cli new  kzapi2 --lang csharp
```
### Build image
Edit the yaml file to prefix with the docker registry
```
sudo faas-cli publish -f kzapi2.yml --platforms linux/amd64


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
```

### Configure DNS (Magic DNS with sslip.io)
```
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.0.0/serving-default-domain.yaml
```
### Install HPA (Horizontal Pod Autoscaling) (Optional)
```
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.0.0/serving-hpa.yaml
```

## Knative Serving: Install first knative app
```
kn service create hello \
--image gcr.io/knative-samples/helloworld-go \
--port 8080 \
--env TARGET=World \
--revision-name=world
```
Call the service by pasting the Kn URL of the service to the browser.

### Update service revision
```
kn service update hello \
--env TARGET=Knative \
--revision-name=knative
````

### Traffic splitting
List the revisions
```
kn revisions ls
```
Split traffic between the two revisions
```
kn service update hello \
--traffic hello-world=50 \
--traffic @latest=50
```

## Knative Eventing
https://knative.dev/docs/install/eventing/install-eventing-with-yaml/
### Required CRDs
```
kubectl apply -f https://github.com/knative/eventing/releases/download/knative-v1.0.0/eventing-crds.yaml
```
### Core components
```
kubectl apply -f https://github.com/knative/eventing/releases/download/knative-v1.0.0/eventing-core.yaml
```
### Verify the installation
```
kubectl get pods -n knative-eventing
```
### Install Kafka Channel(messaging)
#### Install Strimzi
```
kubectl create namespace kafka
```
CRDs
```
kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka
```
Provision Apache Kafka cluster
```
kubectl apply -f https://strimzi.io/examples/latest/kafka/kafka-persistent-single.yaml -n kafka
```
Wait until cluster is ready
```
kubectl wait kafka/my-cluster --for=condition=Ready --timeout=300s -n kafka 
```
##### Test Send & Receive Messages
Send messages
```
kubectl -n kafka run kafka-producer -ti --image=quay.io/strimzi/kafka:0.26.0-kafka-3.0.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic my-topic
```
Receive on another terminal
```
kubectl -n kafka run kafka-consumer -ti --image=quay.io/strimzi/kafka:0.26.0-kafka-3.0.0 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning
``
### Install Broker

#### Install Kafka controller
```
kubectl apply -f https://github.com/knative-sandbox/eventing-kafka-broker/releases/download/knative-v1.0.0/eventing-kafka-controller.yaml
```
#### Install Kafka Broker data plane
```
kubectl apply -f https://github.com/knative-sandbox/eventing-kafka-broker/releases/download/knative-v1.0.0/eventing-kafka-broker.yaml
```

```
kn broker list
```

