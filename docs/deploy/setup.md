# Setup GitOps Core Components

## Provision a Kubernetes cluster

The cluster should be able to create following resources.

- LoadBalancer service (used by ingress controller)
- PV (used by trivy)

## Install sealed-secrets

Install sealed-secrets in the Kubernetes cluster.

```
git clone https://github.com/goldginkgo/gitops-labs.git
cd gitops-labs
k apply -f namespaces
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm dependency build sealed-secrets
helm install sealed-secrets sealed-secrets
```

Intall kubeseal client in you local Ubuntu desktop.

```
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.16.0/kubeseal-linux-amd64 -O kubeseal
sudo install -m 755 kubeseal /usr/local/bin/kubeseal
kubeseal --fetch-cert --controller-name=sealed-secrets --controller-namespace=gitops-system > seal-pub-cert.pem  # cert expires in 30 days
```

Example of how to use sealed-secrets

```
kubectl create secret generic my-secret --from-literal=key1=supersecret --from-literal=key2=topsecret --dry-run=client -o yaml | kubeseal --cert seal-pub-cert.pem -o yaml
```

## Create secrets in the Cluster

```
k apply -f secrets
```

## Deploy ArgoCD and other tools

```
kustomize build argocd | kubectl apply -f -
k apply -f app-of-apps.yaml
```
