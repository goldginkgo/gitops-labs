# K8S Clusters and Services

We have two kinds of Kubernetes clusters: the admin cluster and workload clusters.
The admin cluster is where ArgoCD and other DevOps tools resides, and the workload cluster is the culster where business applications are deployed.

## Admin Cluster

- ArgoCD
- ApplicationSet Controller
- Argo CD Notifications
- Argo CD Image Updater
- Nginx Ingress Controller
- sealed-secrets
- cert-manager
- Jenkins
- Sonarqube
- Nexus
- Trivy
- Prometheus/Grafana
- ...

## Workload Clusters (DEV, TEST, QA, PROD)

- sealed-secrets
- Secrects for docker registry, certificates
- Nginx Ingress Controller
- Filebit
- Business applications
- Argo Rollouts
- Dynatrace Agent
- Velero
