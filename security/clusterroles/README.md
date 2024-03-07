# Manual Cluster Role Provisioning

KNIME Business Hub requires certain Kubernetes [ClusterRole](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole) and and [ClusterRoleBinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding) resources to operate. By default, the Hub installer creates these resources. If the installer does not have permission to create them, they need to be provisioned manually by a privileged user **before the KNIME Business Hub installation** via
```
kubectl apply -f https://bitbucket.org/KNIME/knime-business-hub-public/raw/main/security/clusterroles/clusterroles.yaml
```

Make sure to disable the ClusterRole provisioning in the advanced settings of the installer.

![manual-cluster-role-provisioning](images/manual-cluster-role-provisioning.png)
