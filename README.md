# KNIME Business Hub - Supplemental Installation Resources

The following supplemental install instructions, as well as additional resources found in this repository, assume the install is performed with access to the internet on an existing Kubernetes cluster (version 1.25 to 1.28), that it has **volume provisioner** (CSI) installed, as well as an Ingress controller installed (i.e. ingress-nginx or similar). 

An optional prerequisite is that the Istio Service Mesh be manually installed and configured in the target cluster if cluster security configuration and permissions disallow the creation of the mesh. 

The recommended version of Istio is `1.18.7`. Releases can be found [here](https://github.com/istio/istio/releases/tag/1.18.7), as well as detailed installation instructions [here](https://istio.io/latest/docs/setup/getting-started/#download).

If installing Istio manually, then the option to include Istio in the KOTs Configuration Dialog (when configuring the Hub release) will need to be disabled. If the Configuration Dialog specified that Istio is enabled (the **default** value), then the KNIME Business Hub release process will attempt to install Istio and its related Custom Resource Definitions (CRDs).

Additionally, the following steps also assume that namespaces, DNS entries and related TLS certificates are provisioned before proceeding with the install. 

## Namespaces

The following namespaces need to be initially created:

* `kots` - namespace where the KOTS Admin UI is deployed to (This namespace is specified when running the KOTS install shell script to deploy KOTS as it will act as the deployment administration tool for the Business Hub release. This can alternatively be `default` or any other namespace the admin prefers to use.)
* `istio-system` - the namespace where the Istio service mesh controller will run
* `knime` - namespace where Hub persistence layers run
* `hub` - namespace where Hub microservices run
  * The `hub` namespace additionally needs the `istio-injection=enabled` label set for inclusion in the service mesh
* `hub-execution` - namespace where KNIME executors will run

## Installing KOTS

The KOTS admin UI can be installed by running the following from a host machine where `kubectl` has already been configured to access the Kubernetes cluster

```sh
curl https://kots.io/install | bash kubectl kots install knime-hub
```

When prompted, specify the desire namespace (i.e. `kots`, `default`, etc.) where you prefer the KOTS stack to run. 

Once installation completes, a port-forward tunnel will be automatically opened to allow the browser to connect to the KOTS UI on https://localhost:8800

When installing KOTS into an existing Kubernetes cluster, a tunnel will need to be opened anytime the KOTS UI needs to be accessed for security reasons. 

The KOTS UI tunnel can be re-established by running:

`kubectl kots admin-console -n kots`

( `-n` defines the namespace where KOTS is running, update accordingly)

Alternatively, you can directly establish the port-forward tunnel to kots via:

`kubectl -n kots port-forward service/kotsadm 8800:3000`

## DNS Entries and TLS Certificates

KNIME Business Hub will be accessed via a **base URL**, with a number of subdomains also configured for more targeted purposes. 
The following will use `hub.example.com` as an example base address to highlight how the base address and subdomains are to be configured. 

DNS entries will need to be created for the following endpoints:

* hub.example.com - root URL also used for the Business Hub UI
* apps.hub.example.com - used for exposing workflow-defined Data Apps
* api.hub.example.com - exposed the Business Hub API
* ws.hub.example.com - used for websocket communication between Browser/AP and Business Hub
  * Currently used for exposing KNIME AI service available in Business Hub Standard or Enterprise `1.8.0` and higher
  * Requires the Loadbalancer in front of the Ingress controller supports Layer-4/TCP/TLS (i.e. AWS Application Load Balancer or Network Load Balancer, etc.)
* auth.hub.example.com - used for exposing the embedded Keycloak user store
* storage.hub.example.com - used for exposing the embedded Minio object store where workflows and data files are located

Note that the above domain and subdomains can be either directly set to the relevant Loadbalancer for the Ingress controller, or can alternatively be set by adding the necessary annotations to the Ingress rules below if using an automated DNS provisioner like `External-DNS`.

Additionally, a TLS cert will need to be provisioned either directly or via adding relevant Certificate resources and settings to the the Ingress rules below if using an automated Certificate provisioner like `Cert-Manager`.

If creating the cert explicitly, it's typically recommended to create a cert with the common name hub.example.com, and listing all domains above as subject alternative names, or using a wildcard for the subject alternative name (i.e. *.hub.example.com)

## Ingress Rules

`Ingress` rules will need to be manually created prior to install. A working template of the required `Ingresses` can be found under [/networking/ingress-nginx/ingress.yaml](https://bitbucket.org/KNIME/knime-business-hub-public/src/main/networking/ingress-nginx/ingress.yaml).

Ingress rules may need to be further updated depending on use of other tools to correctly set DNS, TLS, etc. 


## Install Business Hub

Once the following supplemental steps above have been completed, the admin should be able to proceed with the remainder of the install as documented at docs.knime.com

The high-level remaining steps include:

* Logging into the KOTS Admin UI
* Uploading the Replicated license which will fetch the latest release (`1.8.0` of Business Hub)
* Enter appropriate configuration parameters in the KOTS Config Dialog
  * Specifically, note that there is an option under the Networking section to use a "provided" Ingress controller. This will block Business Hub from attempting to deploy its own ingress-nginx.
  * Also, be sure to click on "Enable TLS" as TLS will be configured in front of the Business Hub install (likely at the Ingress Controller level). 
* Save the Config changes, allow the pre-flight checks to run, then click "Deploy"

