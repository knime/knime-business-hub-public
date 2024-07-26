# Setup velero for existing cluster

Knime Business Hub is using [Velero](https://github.com/vmware-tanzu/velero) to create and restore snapshots.


## How to install:

Add the helm repository for Velero:

```
helm repo add vmware-tanzu https://vmware-tanzu.github.io/helm-charts
helm repo update
```

[AWS related sample values](knime-aws-s3-values.yaml).

[Azure sample values](knime-azure-s3-values.yaml).

Install velero with the following command:

`helm upgrade -i velero vmware-tanzu/velero -n velero --create-namespace --version 4.1.3 -f < values file name >.yaml`


---
### Official documentaions:

AWS docs: https://github.com/vmware-tanzu/velero-plugin-for-aws#setup

Azure docs: https://github.com/vmware-tanzu/velero-plugin-for-microsoft-azure#setup

Azure necessary permissions: https://stackoverflow.com/questions/52769758/azure-blob-storage-authorization-permission-mismatch-error-for-get-request-wit
