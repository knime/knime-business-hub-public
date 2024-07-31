# Istio Service Mesh

## Installation

Istio can be manually installed by installing the `istioctl` cli tool and using the `istio-config.yaml` file above to configure and install the service-mesh via its CNI plugin option, which will be more security compliant if security policies exist in the target cluster.

The commands below are a working example of installing Istio based on the provided `istio-config.yaml` file.

Install `istioctl` cli tool:

```sh
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.20.3 TARGET_ARCH=x86_64 sh -
cd istio-1.20.3/
export PATH=$PWD/bin:$PATH
```

Install Istio:

```sh
istioctl install -f istio-config.yaml --verify
```

This will install and verify the Istio service mesh install into the `istio-system` namespace.

## Openshift

Istio can be installed manually into an Openshift cluster or setup using the Openshift Service Mesh Operator.

> See [Openshift Docs: Service Mesh](https://docs.openshift.com/container-platform/latest/service_mesh/v2x/ossm-about.html) for information on setting up the Service Mesh.

When installing on Openshift manually use the following command.

```sh
istioctl install --set profile=openshift -f istio-config.yaml --verify
```

When using the Openshift Service Mesh then use the following commands. Note that this is the minimal configuration needed for KNIME Business Hub to work and can be modified as needed. If Hub is being
installed into a shared cluster where Service Mesh is already being utilized then this config will need to be adapted to suit the cluster's setup (e.g. the proxy section of the ServiceMeshControlPlane
resource may need to be converted into an Istio ProxyConfig custom resource.)

You will need to replace the `<business-hub-namespace>` placeholder with the namespace KNIME Business Hub is installed in, for example using `sed`:

```sh
sed -i "s/<business-hub-namespace>/knime-business-hub/g" ingress.yaml
```

After modifying the resources, deploy them to the cluster:


```sh
kubectl apply -f openshift-servicemesh-config.yaml
```

## Enable JSON Logging for Istio

Istio is not configured for JSON logging by default. To enable it, the `IstioOperator` resource (`networking/istio/istio-config.yaml`) should be updated with the following:

* `.spec.global.logAsJson`
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
