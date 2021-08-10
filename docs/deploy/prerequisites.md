# Prerequisites

## Local Desktop

- Install Git/kubectl/Kustomize/ArgoCD CLI locally.

## DNS

`10.224.14.210` is the IP address for the internal Load Balancer for the admin cluster ingress controller.

| DNS                            |  IP Address   |
| :----------------------------- | :-----------: |
| argocd.gitops.local            | 10.224.14.210 |
| jenkins2.infra.gitops.local    | 10.224.14.210 |
| sonarqube2.infra.gitops.local  | 10.224.14.210 |
| nexus2.infra.gitops.local      | 10.224.14.210 |
| gitops-docs.infra.gitops.local | 10.224.14.210 |

## GitLab

- GitLab repositories: gitops-labs, mlp-helm, mlp-helmcharts, ...
- Create a local GitLab user `jenkins2` and grant `master` permissions for related repositories. Permissions required for committing code to the master branch.
- GitLab access token. Create a [GitLab personal access token](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html#creating-a-personal-access-token) with `api` permission.
- Register a new application in GitLab for SSO. Take a note of the appliation id and secret for later use.

```
Name: gitops ArgoCD
Redirect URI: https://argocd.gitops.local/api/dex/callback
Scopes: read_user, openid
```

- Enable automated syncing by creating a [GitLab webhook](https://docs.gitlab.com/ee/user/project/integrations/webhooks.html). This should be done both on gitops-labs, mlp-helmcharts and gitops-docs repositories.

```
URL: https://argocd.gitops.local/api/webhook
Trigger: Push Events
Enable SSL verification: false
```

## LDAP

- Service accounts: svc_Azure_ArgoCD, svc_Azure_Jenkins
- Groups: g_ArgoCD_Admin, g_ArgoCD_Operator, g_ArgoCD_Reader, g_Jenkins_Admin, g_Jenkins_Operator, g_Jenkins_Reader

## Azure

### Azure Container Registry

- acepi001cr01, aceda022cr01. They are also used as Helm chart repositories.

### Azure Key Vault

- Create Key Vault: acepi001kv02. (Firewall to restrict only K8S clusters can access will be configured later)
- Service principle: PKS-KeyVault (**Expires: 5/11/2023**)
- Assign get and list permission to keys/secrets/certificates for PKS-KeyVault to access acepi001kv02

Define credentials Azure Key Vault.

- argocd-repo-credentials: GitLab access token and Helm registry credentials for acepi001cr01
- acr-aceda022cr01: Azure container registry aceda022cr01 credentials
- poc-cluster-credentials: POC K8S cluster credentials
- argocd-secret-credentials: Credentials for argocd-secret.
- teams-webhook: Webhook URL for MS Teams.
- ...

The Azure Portal currently only supports single-line secret values. Please use Azure PowerShell to set multi-line values. For example:

```
az cloud set --name AzureChinaCloud
az login
az keyvault secret set --vault-name acepi001kv02 --name velero-cloud-credentials --value "AZURE_SUBSCRIPTION_ID=xxx
AZURE_TENANT_ID=xxx
AZURE_CLIENT_ID=xxx
AZURE_CLIENT_SECRET=xxx
AZURE_RESOURCE_GROUP=ahkpi001
AZURE_STORAGE_ACCOUNT_ACCESS_KEY=xxx
AZURE_CLOUD_NAME=AzurePublicCloud"
```

### Azure Load Balancer

- Create a Azure internal Load Balancer (10.224.14.210).

## Rancher

- Argo CD projects for different clusters. Need to create Rancher API token because we are using Rancher to manage our clusters as per `https://gist.github.com/janeczku/b16154194f7f03f772645303af8e9f80`. Update `bearerToken` in `argocd/base/<env>-cluster-secret.yaml` files.

## Microsoft Teams

- Microsoft Teams channel for notification. Refer to [Microsoft Teams channel](https://argocd-notifications.readthedocs.io/en/stable/services/teams/).

## SMTP

- SMTP Server: smtp.gitops.local. Whitelist the network access from kubernetes node to the SMTP server.
