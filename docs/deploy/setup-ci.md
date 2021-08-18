# Setup CI Tools

## Nexus Configuration

- Sync Nexus in ArgoCD.
- Get default password using following command. Log in and change the password. Also enable anonymous access.

```console
# kubectl exec -it deploy/nexus-nexus-repository-manager -- cat /nexus-data/admin.password
```

- Update credentials for Nexus in Azure Key Vault

## SonarQube Configuration

- Sync SonarQube in ArgoCD.
- Log in. default user admin/admin. Change the password after login.
- Generate a SonarQube token in user profile and update the secret.
- Configure a webhook for Jenkins in SonarQube server in Administration > Configuration > Webhooks.

```console
Name: Jenkins
URL: http://jenkins.gitops-system.svc.cluster.local:8080/sonarqube-webhook/
Secret:
```

- Add plugins in marketplace: PMD

## Jenkins Configuration

- Sync Jenkins in ArgoCD.
- We use Jenkin [Configuration as Code](https://plugins.jenkins.io/configuration-as-code/) plugin for configuring Jenkins so that no manual efforts required. When synchronizing Jenkins from ArgoCD, Jenkins will be configured. Please contact the administrator if you want to add a plugin or change system settings.

## Setup Jenkins Jobs

When Jenkins is setup, a job called `load_all_jobs` is created. This job will populate all jobs defined in `gitops-gitops/jenkins-jobs`. Just build this job so that all other jobs will be generated.
It's maybe necessary to approve the scripts in Manage Jenkins -> Script Approval.

## GitLab Repository Webhook

Configure webhooks for push events in the repositories so that code changes can trigger Jenkins jobs automatically.
