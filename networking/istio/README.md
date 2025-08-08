# Istio Service Mesh

## Installation

Istio can be manually installed by installing the `istioctl` cli tool and using the `istio-config.yaml` file above to configure and install the service-mesh via its CNI plugin option, which will be more security compliant if security policies exist in the target cluster.

The commands below are a working example of installing Istio based on the provided `istio-config.yaml` file.

Install `istioctl` cli tool:

```sh
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.24.2 TARGET_ARCH=x86_64 sh -
cd istio-1.24.2/
export PATH=$PWD/bin:$PATH
```

In [istio-config.yaml](istio-config.yaml) you can find a minimal istio configuration that works with KNIME Business Hub. Consult the istio documentation for all possible options:

- [IstioOperator resource configuration](https://istio.io/latest/docs/reference/config/istio.operator.v1alpha1/)
- [Global mesh configuration](https://istio.io/latest/docs/reference/config/istio.mesh.v1alpha1/) for everything under `meshConfig`

However, not all settings you can find there might work well with KNIME Business Hub, compare your changes to this file if something is not working.

Before installing, check the namespace of the `istio-ingressgateway` and the `auth-exchanger` extension provider under `spec.meshConfig.extensionProviders.envoyExtAuthzHttp.service`, and update it to the KNIME Business Hub namespace if you installed in a different namespace than `knime`.

Then install istio:

```sh
istioctl install -f istio-config.yaml --verify
```

This will install and verify the Istio service mesh install into the `istio-system` namespace.

Additionally, some EnvoyFilters can be found in `networking/istio/envoyfilter/` which are recommended to be deployed after installing istio, to enable http compression in istio or more user friendly error messages. You may need to update the namespace for each EnvoyFilter before deploying.

## Openshift

Istio can be installed manually into an Openshift cluster or setup using the Openshift Service Mesh Operator.

> See [Openshift Docs: Service Mesh](https://docs.openshift.com/container-platform/latest/service_mesh/v2x/ossm-about.html) for information on setting up the Service Mesh.

When installing on Openshift manually use the following command.

```sh
istioctl install --set profile=openshift -f istio-config.yaml --verify
```

When using the Openshift Service Mesh then use the following command. Note that this is the minimal configuration needed for KNIME Business Hub to work and can be modified as needed. If Hub is being
installed into a shared cluster where Service Mesh is already being utilised then this config will need to be adapted to suit the cluster's setup (e.g. the proxy section of the ServiceMeshControlPlane
resource may need to be converted into an Istio ProxyConfig custom resource.)

```sh
kubectl apply -f openshift-servicemesh-config.yaml
```

## Native Helm based installs

Istio now also offers native helm charts: https://istio.io/latest/docs/setup/install/helm/. 

We do not provide values files for these charts, however you can use `istioctl manifest translate` to let istio generate values files for you based on the istio-config.yaml in this repository: https://istio.io/latest/docs/reference/commands/istioctl/#istioctl-manifest-translate.

## Enable JSON Logging for Istio

Istio is not configured for JSON logging by default. To enable it, the `IstioOperator` resource (`networking/istio/istio-config.yaml`) should be updated with the following:

* `.spec.values.global.logAsJson`
* `.spec.meshConfig.accessLogEncoding`
* `.spec.meshConfig.accessLogFile`
* `.spec.meshConfig.accessLogFormat`

See below for an example of how to configure Istio for JSON logging. All other config in the `networking/istio/istio-config.yaml` file would remain the same and has been excluded from the example.

> See [Istio Docs: Envoy Access Logs](https://istio.io/latest/docs/tasks/observability/logs/access-log/) for more details on the access log format.

```yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  name: istio-operator
spec:
  values:
    global:
      logAsJson: true
  meshConfig:
    accessLogEncoding: JSON
    accessLogFile: /dev/stdout
    accessLogFormat: |
      {
        "etag": "%RESP(etag?ETag)%",
        "if_none_match": "%REQ(If-None-Match)%",
        "knime_api_version": "%REQ(KNIME-API-Version)%",
        "x_knime_client_type": "%REQ(X-KNIME-Client-Type)%",
        "bytes_sent": "%BYTES_SENT%",
        "upstream_cluster": "%UPSTREAM_CLUSTER%",
        "downstream_remote_address": "%DOWNSTREAM_REMOTE_ADDRESS%",
        "authority": "%REQ(:AUTHORITY)%",
        "path": "%REQ(X-ENVOY-ORIGINAL-PATH?:PATH)%",
        "x_envoy_original_path": "%REQ(X-ENVOY-ORIGINAL-PATH)%",
        "path_path": "%REQ(PATH)%",
        "protocol": "%PROTOCOL%",
        "upstream_service_time": "%REQ(x-envoy-upstream-service-time)%",
        "upstream_local_address": "%UPSTREAM_LOCAL_ADDRESS%",
        "duration": "%DURATION%",
        "upstream_transport_failure_reason": "%UPSTREAM_TRANSPORT_FAILURE_REASON%",
        "route_name": "%ROUTE_NAME%",
        "downstream_local_address": "%DOWNSTREAM_LOCAL_ADDRESS%",
        "user_agent": "%REQ(USER-AGENT)%",
        "response_code": "%RESPONSE_CODE%",
        "response_flags": "%RESPONSE_FLAGS%",
        "start_time": "%START_TIME%",
        "method": "%REQ(:METHOD)%",
        "request_id": "%REQ(X-REQUEST-ID)%",
        "upstream_host": "%UPSTREAM_HOST%",
        "x_forwarded_for": "%REQ(X-FORWARDED-FOR)%",
        "x_forwarded_port": "%REQ(X-FORWARDED-PORT)%",
        "x_forwarded_proto": "%REQ(X-FORWARDED-PROTO)%",
        "requested_server_name": "%REQUESTED_SERVER_NAME%",
        "bytes_received": "%BYTES_RECEIVED%",
        "istio_policy_status": "-"
      }
```
