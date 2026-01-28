
# knative

## Create Cluster

```bash
k3d cluster create knative \
  --port "8080:80@loadbalancer" \
  --agents 2 \
  --k3s-arg "--disable=traefik@server:0"
```

## Serving CRDs + Core

```bash
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.19.6/serving-crds.yaml
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.19.6/serving-core.yaml
```

## Kourier (Ingress)

> Version should really match that of knative, but was not yet available

```bash
kubectl apply -f https://github.com/knative-extensions/net-kourier/releases/download/knative-v1.19.5/kourier.yaml
```

## Patch ingress class

```bash
kubectl patch configmap/config-network \
  --namespace knative-serving \
  --type merge \
  --patch '{"data":{"ingress-class":"kourier.ingress.networking.knative.dev"}}'s
```

## Patch domain for local access

```bash
kubectl patch configmap/config-domain \
  --namespace knative-serving \
  --type merge \
  --patch '{"data":{"127.0.0.1.sslip.io":""}}'
```

## Validate

```bash
kubectl get pods -n knative-serving
kubectl get pods -n kourier-system
```

## Build service

```bash
docker build -t ttl.sh/hello-knative:1h .
docker push ttl.sh/hello-knative:1h
```

## Deploy service

```bash
kubectl apply -f hello-knative.yaml
```
