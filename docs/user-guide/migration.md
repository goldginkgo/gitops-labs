# Migrating Helm Charts
* Get manifests from current environment
```
helm list
helm get manifest <helm-name>
```
* Get manifests from local git repository
```
export HELM_EXPERIMENTAL_OCI=1
helm dependency build
helm template ouser-service .
```
* Compare 2 manifest and change the values in local repository so that values are consistent.