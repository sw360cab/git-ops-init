# GitOps Basics

Basic setup to test GitOps with ArgoCD

## Docker Image

- Build and push a basic image onto Docker Hub Image registry

```sh
docker build -t git-ops-init:0.1.0 app/
docker tag git-ops-init:0.1.0 sw360cab/git-ops-init:0.1.0
docker push sw360cab/git-ops-init:0.1.0
```

## Argo CD

### Install

```sh
kubectl create namespace argocd
kubectl apply --server-side -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

```

**Note:** `--server-side` is required. the `applicationsets.argoproj.io` CRD exceeds the 256 KB annotation limit used by client-side `kubectl apply`. See [the official docs](https://argo-cd.readthedocs.io/en/stable/getting_started)

### Deploy ArgoCD Application

```sh
kubectl apply -f infra/gitops/argocd/gitops-app.yaml
```

### Web UI

Accessing UI is possible by installing the `argocd CLI` or  by directly using port-forward of `argocd-server` UI

```sh
kubectl -n argocd port-forward svc/argocd-server 8080:443
# https://localhost:8080   (self-signed cert → accept the warning)
```
