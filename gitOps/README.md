# GitOps with ArgoCD
## Install ArgoCD
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
### ArgoCD cli
```
brew install argocd
```
### Port-forwarding to ArgoCD API
```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
### Login using the CLI

Get the initial admin password

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
Change the initiated pwd
```
argocd account update-password
```
### Create App from Git repo
use `https://kubernetes.default.svc` to create App from a git repo `https://github.com/argoproj/argocd-example-apps.git` and path `guestbook`
