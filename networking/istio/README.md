# Istio Service Mesh

Istio can be manually installed by installing the `istioctl` cli tool and using the `istio-config.yaml` file above to configure and install the service-mesh via its CNI plugin option, which will be more security compliant if security policies exist in the target cluster. 

The commands below are a working example of installing Istio based on the provided `istio-config.yaml` file. 

Install `istioctl` cli tool
```sh
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.18.7 TARGET_ARCH=x86_64 sh -
cd istio-1.18.7/
export PATH=$PWD/bin:$PATH
```

Install Istio
```sh
istioctl install -f istio-config.yaml --verify
```

This will install and verify the Istio service mesh install into the `istio-system` namespace. 