# Setup GitOps Core Components

```
cd gitops-labs
k apply -f namespaces
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm dependency build sealed-secrets
helm install sealed-secrets sealed-secrets

```

- Create a Azure AKS cluster.

- Add the Cluster in Rancher and enable monitoring. Import the dashboard in Grafana as per [ArgoCD Metrics Documentation](https://argo-cd.readthedocs.io/en/stable/operator-manual/metrics/).

- Install akv2k8s on all admin and workload clusters. Replace the values in the command with actual values. Be sure to move gitops-system namespace to `system` project in Rancher. This is because the ServiceMonitor resource (currently defined in gitops-system namespace) must be in the same project with Prometheus resource. Otherwise, the dashboard will not work.

```console
$ kubectl create ns gitops-system velero
$ helm repo add spv-charts https://charts.spvapi.no
$ helm repo update
$ helm upgrade --install akv2k8s spv-charts/akv2k8s \
  --namespace gitops-system \
  --set global.keyVaultAuth=environment \
  --set global.env.AZURE_TENANT_ID=<REPLACE_ME> \
  --set global.env.AZURE_CLIENT_ID=<REPLACE_ME> \
  --set global.env.AZURE_CLIENT_SECRET=<REPLACE_ME> \
  --set global.env.AZURE_ENVIRONMENT=AzureChinaCloud \
  --set env_injector.enabled=false --version 2.0.11
```

- Comment `- base/argocd-certificate.yaml` and `- base/argocd-ui-ingress.yaml` in argocd/kustomization file (cert-manager and ingress controller not installed yet). Install the GitOps engine by executing following commands. Wait until all apps are setup successfully. You may need to sync the apps manually in case of issues. Revert the comments once ArgoCD is installed.

```console
$ k ns gitops-system
$ git clone git@http://gitlab.gitops.local/gitops-gitops/gitops-labs.git
$ cd gitops-labs
$ kustomize build argocd | kubectl apply -f -
$ kubectl apply -f app-of-apps.yaml
```

- Bind the Azure internal Load Balancer (10.224.14.210) with the K8S worker node. (Kubernetes Load Balancer service is not used for compliance considerations. Instead we use NodePort service with 32000 for http port and 32001 for https port).
