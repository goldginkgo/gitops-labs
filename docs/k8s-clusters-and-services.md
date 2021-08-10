# K8S Clusters and Services

We have two kinds of Kubernetes clusters: the admin cluster and workload cluster.
The admin cluster is where ArgoCD and other DevOps tools resides, and the workload cluster is the culster that business applications are deployed.

## Admin Cluster

- ArgoCD
- ApplicationSet Controller
- Argo CD Notifications
- Argo CD Image Updater
- Nginx Ingress Controller
- Azure Key Vault to Kubernetes
- cert-manager
- Jenkins
- Sonarqube
- Nexus
- Trivy
- Prometheus/Grafana
- ...

## Workload Clusters (DEV, TEST, QAS, PROD)

- Azure Key Vault to Kubernetes
- Secrects for docker registry, certificates
- Nginx Ingress Controller
- Filebit
- MLP applications
- Argo Rollouts
- Dynatrace Agent
- Velero
