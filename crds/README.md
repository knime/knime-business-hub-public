# Custom Resource Definitions

If an environment is running in restricted RBAC mode, CRDs cannot be installed by the installer, therefore they need to be applied manually prior to starting the installation process.

CRDs can be installed with the following command:
```
kubectl create -R -f .
```

The above command will recursively install all CRDs organised in subfolders in the crds/ directory.
