# Roadmap

## ArgoCD

- Blue Green/Canary deployment
- Update Image back to Gitlab according to https://blog.argoproj.io/closing-ci-cd-loop-using-argoproj-a78a50a98fe8
- HA deployment of Argo CD, Configuration tuning for large environments
- Monitoring/Logging Optimization
- Restrict IP to access Azure Key Vault

## Jenkins

- Jenkins shared library
- Validate Kubernetes manifests with [kube-score](https://github.com/zegl/kube-score), [conftest](https://github.com/open-policy-agent/conftest) or [polaris](https://github.com/FairwindsOps/polaris).
- SSH using or [SSH Pipeline Steps](https://plugins.jenkins.io/ssh-steps) or [Publish Over SSH](https://plugins.jenkins.io/publish-over-ssh/)
- Multi-branch pipeline
- Backup
- Cache base images for kaniko

## SonarQube

- LDAP support

## Nexus

- LDAP support
- Detailed configrations
- Nexux as Helm Chart Proxy
