# Upgrade

## Upgrade ArgoCD

Currently we use v2.0.4 of ArgoCD. If you want to upgrade ArgoCD, please replace the `gitops-labs/argocd/base/install.yaml` file with the latest one.

## Upgrade Jenkins

- Update the Dokcerfile in `jenkins-docker` repository. The Jenkins job `gitops/jenkins-docker` will be executed automatically. Image tag will bump up a minor version, as well as the new git tag in the repository.
- Update the image tag in `gitops-labs/jenkins/values.yaml`.
- [Optional] Update the Helm chart version in `gitops-labs/jenkins/Chart.yaml`.
- Restart the jenkins statefulset in ArgoCD UI.
