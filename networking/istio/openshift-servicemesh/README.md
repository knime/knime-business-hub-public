# Openshift Service Mesh

> See [Openshift Docs: Service Mesh](https://docs.openshift.com/container-platform/latest/service_mesh/v2x/ossm-about.html) for information on setting up the Service Mesh.

Installation instructions differ based on if Service Mesh version 2.x or 3.x is installed. Please use the appropriate configuration

## Version 2.x Installation

When using the Openshift Service Mesh version 2.x then use the following command after the Service Mesh operator has been installed. Note that this is the minimal configuration needed for KNIME Business Hub to work and can be modified as needed. If Hub is being
installed into a shared cluster where Service Mesh is already being utilised then this config will need to be adapted to suit the cluster's setup (e.g. the proxy section of the ServiceMeshControlPlane
resource may need to be converted into an Istio ProxyConfig custom resource.)

```sh
kubectl apply -n istio-system -f openshift-servicemesh-v2-config.yaml
```

## Version 3.x Installation

With Service Mesh 3.x the istio control plane is installed separately to the ingress gateway resources.

Run the following command after the Openshift Service Mesh operator has been installed to configure the istio control plane.

```sh
kubectl apply -n istio-system -f openshift-servicemesh-v3-config.yaml
```

Run the following command to install the istio ingress gateway resource.

```sh
kubectl apply -n <hub-namespace> -f ingress-gateway.yaml
```

The istio ingress gateway can also be installed via helm with the following command. Issues may be seen on clusters using the restricted-v2 Security Context Constraint and the above method is currently preferred.

```sh
helm install -n <hub-namespace> istio-ingressgateway istio/gateway --set global.platform=openshift --version=1.24.6 -f gateway-values.yaml
```