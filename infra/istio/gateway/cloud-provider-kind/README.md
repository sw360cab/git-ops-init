# Gateway API Service

## Deploying

```sh
kubectl apply -f infra/istio/gateway/cloud-provider-kind
```

## Test HttpRoute

- Getting Gateway address

```sh
GW_ADDR=$(kubectl get gateway -n default git-ops-gw -o jsonpath='{.status.addresses[0].value}')
```

- Test via cURL resolving domain

```sh
curl --resolve some.exampledomain.example:8085:${GW_ADDR} http://some.exampledomain.example:8085
```

## Gateway API in KinD

Reference to [Cloud Provider Kind](../../../../README.md#load-balancer-cloud-provider-kind)

**Reference:** [Experimenting with Gateway API using Kind](https://kubernetes.io/blog/2026/01/28/experimenting-gateway-api-with-kind/)
(Kubernetes blog).
