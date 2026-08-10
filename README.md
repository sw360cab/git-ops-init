# GitOps Basics

Basic setup to test GitOps with ArgoCD.

## Docker Image

- Build and push a basic image onto Docker Hub Image registry

```sh
docker build -t git-ops-init:0.1.0 app/
docker tag git-ops-init:0.1.0 sw360cab/git-ops-init:0.1.0
docker push sw360cab/git-ops-init:0.1.0
```

## ArgoCD

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

### ArgoCD Image Updater

**Note:**: Image Updater is in a separate companion project rather then basic ArgoCD (`argoproj`)

```sh
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/config/install.yaml
kubectl apply -f infra/gitops/argocd/image-updater.yaml
```

## Kind Cluster

Create the local cluster from the config file:

```sh
kind create cluster --config infra/cluster/kind.yaml
```

### Load Balancer (cloud-provider-kind)

[cloud-provider-kind](https://github.com/kubernetes-sigs/cloud-provider-kind/) provides
`Service` type `LoadBalancer` support for Kind clusters (Kind has no cloud controller of its own).

Install via Homebrew:

```sh
brew install cloud-provider-kind
```

Run it (keep it running in its own terminal; needs `sudo` to set up the
port mapping from the host to the Docker network):

```sh
sudo cloud-provider-kind
```

**Recommended:** use a cluster with a dedicated `worker` node (see `infra/cluster/kind.yaml`).
cloud-provider-kind then has a worker to use as a `LoadBalancer` backend while keeping the
control-plane node available for other features no label changes needed, and it works for
all use cases (LoadBalancer **and** Gateway API).

#### Single control-plane node only (LoadBalancer use case)

When running a cluster with **only** a control-plane node, cloud-provider-kind skips it as a
`LoadBalancer` backend by default. To let it be used as a backend, remove the control-plane label:

```sh
kubectl label node git-ops-init-control-plane node-role.kubernetes.io/control-plane-
```

> ⚠️ This is **only** needed for the plain `LoadBalancer` use case on a single-node cluster, and
> it **breaks Gateway API**: cloud-provider-kind needs a control-plane node to use as the gateway
> routing host, so removing the label causes
> `could not find any control-plane node to use as a gateway for services`.
> When using Gateway API, do **not** remove the label — use a worker node instead.
