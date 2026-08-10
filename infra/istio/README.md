# Istio

- Install `istioctl`, see [Istio Docs](https://istio.io/latest/docs/setup/getting-started/#download), using the demo profile, without any gateways:

```sh
bin/istioctl install -f samples/bookinfo/demo-profile-no-gateways.yaml -y
```

- Install Gateway API CRDs (if missing)

```sh
kubectl get crd gateways.gateway.networking.k8s.io &> /dev/null || \
{ kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v1.5.1" | kubectl apply -f -; }
```

- Add a namespace label to instruct Istio to `automatically inject` Envoy sidecar proxies

```sh
kubectl label namespace default istio-injection=enabled
```

- Deploy Gateway

```sh
kubectl apply -f infra/istio/gateway/istio-gw
```

- Test Service

```sh
export INGRESS_HOST=$(kubectl get gtw git-ops-istio -o jsonpath='{.status.addresses[0].value}')
export INGRESS_PORT=$(kubectl get gtw git-ops-istio -o jsonpath='{.spec.listeners[?(@.protocol=="HTTP")].port}')
curl -s "http://${INGRESS_HOST}:${INGRESS_PORT}"
```

## References

- [Istio / Getting Started](https://istio.io/latest/docs/setup/getting-started)
- [Istio / Bookinfo Application](https://istio.io/latest/docs/examples/bookinfo/#whats-next)
- [Istio / Canary Deployments using Istio](https://istio.io/latest/blog/2017/0.1-canary/)
