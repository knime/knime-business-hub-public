# Traefik

This folder contains a Helm-based Traefik install for AWS: TLS terminates on an **Network Load Balancer** with **ACM**, and Traefik forwards HTTP to the istio-ingressgateway (see `traefik-values.yaml`).

## Prerequisites

- `kubectl` configured for the target cluster
- [Helm 3](https://helm.sh/)
- Nodes (or a node pool) labeled/tainted as expected by `traefik-values.yaml` (`hub.knime.com/role=core`), or edit those fields to match your cluster

## Install Traefik

From this directory:

1. **Create the namespace**

   ```shell
   kubectl create namespace traefik
   ```

2. **Add the Traefik Helm repository**

   ```shell
   helm repo add traefik https://traefik.github.io/charts
   helm repo update
   ```

3. **Install or upgrade with the provided values**

   ```shell
   helm upgrade --install traefik traefik/traefik \
     --namespace traefik \
     -f traefik-values.yaml
   ```

4. **Customize before production**

   - In `traefik-values.yaml`, replace the ACM certificate ARN, AWS region/account in annotations, and any load balancer settings to match your environment.
   - Adjust affinity/tolerations if your node labels differ from `hub.knime.com/role=core`.

## Optional: IngressRoute example

`ingress.yaml` is an **IngressRoute** that sends traffic for several hostnames to the Istio ingress gateway (`istio-system/istio-ingressgateway`). It is **not** applied by the Helm chart; apply it after you edit hosts and namespaces to match your setup:

```shell
kubectl apply -f ingress.yaml
```

Ensure the Traefik CRDs are installed (they ship with the official Traefik Helm chart and are enabled in the example values file).
