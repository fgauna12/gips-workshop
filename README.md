
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

Then fork the [front-end](https://github.com/microservices-demo/front-end) repository.

Create a GitHub Action to build and push to your GitHub registry. 
- It already has a workflow. Disable it.

[Push to a GitHub Container registry](https://docs.github.com/en/actions/publishing-packages/publishing-docker-images)

``` yaml
# This workflow uses actions that are not certified by GitHub.
# They are provided by a third-party and are governed by
# separate terms of service, privacy policy, and support
# documentation.

name: Create and publish a Docker image

on:
  workflow_dispatch:

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push-image:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Log in to the Container registry
        uses: docker/login-action@f054a8b539a109f9f41c372932f1ae047eff08c9
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@98669ae865ea3cffbcbaa878cf57c20bbf1c6c38
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}

      - name: Build and push Docker image
        uses: docker/build-push-action@ad44023a93711e3deb337508980b4b5e9bcdc5dc
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}

```

Then modify the shop manifests to use a new version of the UI.

### Switching the source

Delete the application definition

```
argocd app delete sock-shop
```

Then re-create with your new repository
```
argocd app create sock-shop \
    --repo https://github.com/fgauna12/gitops-workshop.git \
    --path k8s \
    --dest-namespace sock-shop \
    --dest-server https://kubernetes.default.svc \
    --directory-recurse --sync-policy auto

```

### Making a change

We're going to change the "Offer of the day" button from green to blue.

