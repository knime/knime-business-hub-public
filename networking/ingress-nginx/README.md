# ingress-nginx

KNIME Business Hub deploys and uses ingress-nginx for traffic ingress. However, the options exposed in the KOTS admin console are not enough to cover most existing cluster scenarios, and in some clusters there's an ingress-nginx controller already deployed that should be reused. In those cases KNIME Business Hub can be configured to not deploy ingress-nginx, and instead rely on the cluster operator to deploy necessary resources.

## Prerequesites

- `kubectl` matching the cluster version
- latest `helm` version, if installing ingress-nginx using a helm chart https://helm.sh/docs/intro/quickstart/

> **Note**: Some features in KNIME Business Hub, eg the Job Viewer, use websockets. If an external proxy or load balancer is used it needs to be websocket compatible. 

## Deploy ingress-nginx

If an ingress-nginx controller is not already deployed in the cluster you'll need to deploy one first. We are using the [official ingress-nginx chart developed by the Kubernetes project](https://github.com/kubernetes/ingress-nginx).

Start by adding the ingress-nginx helm repository to your local helm installation:

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

In [ingress-nginx-values.yaml](ingress-nginx-values.yaml) you'll find an example values file as a starting point. Please consult your cloud providers documentation on loadbalancer implementations and necessary custom service annotations.

After configuring the values file deploy the helm chart: 

```
helm upgrade -i -n <namespace> <release name> ingress-nginx/ingress-nginx --version <version> -f <values file>
```

Example:

```
helm upgrade -i -n knime business-hub-ingress-nginx ingress-nginx/ingress-nginx --version 4.9.0 -f ingress-nginx-values.yaml
```

**Please note**:

- If deploying in the `knime` namespace do not use `ingress-nginx` as the release name, as KOTS will think it needs to manage this release, and will ultimately delete it
- KNIME Business Hub has been tested with ingress-nginx chart version 4.9.0

## Deploy Ingress resources

In [ingress.yaml](ingress.yaml) you'll find Ingress resources needed for KNIME Business Hub that can be further configured to fit your desired ingress strategy.

In each Ingress resource you will also need to replace the `<baseurl>` placeholder with the Webapp URL configured in the KOTS admin console, for example using `sed`:

```
sed -i "s/<baseurl>/hub.example.com/g" ingress.yaml
```

Similarly, a `sed` command can be used to replace the `ingressClassName` of each `Ingress` resource.

```
sed -i "s/<ingressclass>/business-hub/g" ingress.yaml
```

After modifying the resources deploy them to the cluster:

```
kubectl apply -f ingress.yaml
```

## Configure KNIME Business Hub

To configure KNIME Business Hub to use a cluster-provided ingress-nginx controller: in the KOTS admin console go to the Config tab and first enable the "View Advanced Settings" option that you can find in the Global section at the top. Afterwards go to the Networking section and for "Ingress Controller Configuration" select "Provided by the cluster". Select the "Enable TLS" option if KNIME Business Hub will be secured with TLS, either by the ingress-nginx controller or a loadbalancer.

![kots-networking-configuration](images/kots-networking-configuration.png)
