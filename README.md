
## Pre-Requisites
- ArgoCD
- kind

## Overview

Create the cluster with [ingress](https://kind.sigs.k8s.io/docs/user/ingress/)

```bash
kind create cluster --config kind-with-ingress.yaml
```

```bash
kubectl cluster-info --context kind-kind
```


## Install NGINX Ingress 

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

```
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

## Install Argo CD

``` bash
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Then port-forward

``` bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Get the secret 

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
### Login with Argo CLI

``` bash
argocd login localhost:8080
argocd account update-password
```

### Create an app from the Argo CLI

``` 
argocd app create guestbook --repo https://github.com/argoproj/argocd-example-apps.git --path guestbook --dest-namespace default --dest-server https://kubernetes.default.svc --directory-recurse

```

Then sync it from the UI or CLI

Then port-forward

``` bash
kubectl port-forward svc/guestbook-ui 8081:80
```

## Deploying Microservices

``` bash
argocd app create shock-shop --repo https://github.com/argoproj/argocd-example-apps.git --path sock-shop --dest-namespace default --dest-server https://kubernetes.default.svc --directory-recurse
```

Ahhhhh!

Then port-forward

``` bash
kubectl port-forward svc/front-end 8082:80
```

## Customizing the front-end

Fork this argo samples repository like `socks-shop-infra`

Then fork the `ui` repository.

Create a GitHub Action to build and push to your GitHub registry. 

Then modify the shop manifests to use a new version of the UI.


