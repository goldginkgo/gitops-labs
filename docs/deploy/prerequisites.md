# Prerequisites

## Local Desktop

- Install Git/kubectl/Kustomize/ArgoCD CLI locally.

## Provision a Kubernetes cluster

The cluster should be able to create following resources.

- LoadBalancer service (used by ingress controller)
- PV (used by trivy, Jenkins, SonarQube, ...)

## DNS

`10.224.14.210` is the example IP address for the internal Load Balancer for the admin cluster ingress controller.
Add the mapping to hosts file if necessary.

| DNS                    |  IP Address   |
| :--------------------- | :-----------: |
| argocd.gitops.local    | 10.224.14.210 |
| jenkins.gitops.local   | 10.224.14.210 |
| sonarqube.gitops.local | 10.224.14.210 |
| nexus.gitops.local     | 10.224.14.210 |

## Microsoft Teams

- Microsoft Teams channel for notification. Refer to [Microsoft Teams channel](https://argocd-notifications.readthedocs.io/en/stable/services/teams/).

## SMTP

- SMTP Server: smtp.gitops.local. Whitelist the network access from kubernetes node to the SMTP server.
