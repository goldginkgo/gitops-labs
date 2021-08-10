# Basic Usage

## Add a new K8S environment

- Install akv2k8s on the cluster

```console
$ kubectl create ns gitops-system
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

- Create API Key with permission to the K8S cluster in Rancher.
- Create credentail `<env>-cluster-credentials` in Azure Key Vault.
- Add file `gitops-labs/argocd/base/project-<env>.yaml`.
- Add file `gitops-labs/argocd/base/cluster-<env>-secret-sync.yaml`.
- Add the files in `gitops-labs/argocd/kustomization.yaml`.
- Add file `gitops-labs/secrets/<env>/<namespace>-regsecret-sync.yaml` for docker registry credentials.
  You need to delete existing secret because akv2k8s can not update existing secrets.
- Add file `gitops-labs/applications/<env>/<env>-secrets.yaml`, `gitops-labs/applications/<env>/<env>-mlp-apps.yaml`.
- Add the above file in `gitops-labs/applications/kustomization.yaml`.
- Add dynatrace support in the cluster. Currently it's done manually.
- Add Velero manually.
- Add helm charts in `gitops-gitops/mlp-helmcharts/stress/`.

## Manage a new application

Example application.

```yml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: <name>
  namespace: gitops-system
  annotations:
    argocd.argoproj.io/manifest-generate-paths: /<manifest-path>
    # notifications.argoproj.io/subscribe.on-created.teams: argocd-notifications
    # notifications.argoproj.io/subscribe.on-deleted.teams: argocd-notifications
    # notifications.argoproj.io/subscribe.on-deployed.teams: argocd-notifications
    # notifications.argoproj.io/subscribe.on-health-degraded.teams: argocd-notifications
    # notifications.argoproj.io/subscribe.on-sync-failed.teams: argocd-notifications
    # notifications.argoproj.io/subscribe.on-sync-status-unknown.teams: argocd-notifications
    # notifications.argoproj.io/subscribe.on-sync-succeeded.teams: argocd-notifications
  # Add a this finalizer ONLY if you want these to cascade delete.
  # finalizers:
  #   - resources-finalizer.argocd.argoproj.io
spec:
  project: <project-name>

  source:
    repoURL: http://560GHD11/gitops-gitops/<repository>
    targetRevision: HEAD
    path: <manifest-path>

  destination:
    server: <cluster-url>
    namespace: <namespace>

  syncPolicy:
    automated: # automated sync by default retries failed attempts 5 times with following delays between attempts ( 5s, 10s, 20s, 40s, 80s ); retry controlled using `retry` field.
      prune: true # Specifies if resources should be pruned during auto-syncing ( false by default ).
      selfHeal: true # Specifies if partial app sync should be executed when resources are changed only in target Kubernetes cluster and no git change detected ( false by default ).
      # allowEmpty: false # Allows deleting all application resources during automatic syncing ( false by default ).
    syncOptions: # Sync options which modifies sync behavior
      - Validate=false # disables resource validation (equivalent to 'kubectl apply --validate=false') ( true by default ).
      - CreateNamespace=true # Namespace Auto-Creation ensures that namespace specified as the application destination exists in the destination cluster.
      - ApplyOutOfSyncOnly=true
      - PrunePropagationPolicy=foreground # Supported policies are background, foreground and orphan.
      - PruneLast=true # Allow the ability for resource pruning to happen as a final, implicit wave of a sync operation
    retry:
      limit: 5 # number of failed sync attempt retries; unlimited number of attempts if less than 0
      backoff:
        duration: 5s # the amount to back off. Default unit is seconds, but could also be a duration (e.g. "2m", "1h")
        factor: 2 # a factor to multiply the base duration after each failed retry
        maxDuration: 3m # the maximum amount of time allowed for the backoff strategy


  # # Ignore differences at the specified json pointers
  # ignoreDifferences:
  # - group: apps
  #   kind: Deployment
  #   jsonPointers:
  #   - /spec/replicas
```

## Rollback applications to previous state

It is as simple as do a Git revert in GitLab.

## How to troubleshoot issues

- Disable automated syncing for `app-of-apps` application.
- Diable automated syncing for the application in trouble.
- Troubleshoot manually for the application.
- Enable automated syncing again.

## Self Healing

If manual changes are done in the target K8S cluster, ArgoCD will reconcile the changes and retore to the desired state when automated syncing is enabled for the application.
