# Numeraid — IAM Database Kubernetes Manifests

This repository contains Kubernetes manifests to deploy the Identity & Access Management (IAM) database (Microsoft SQL Server) for Numeraid. It includes a base set of manifests and an Argo CD Application manifest to deploy the base to a cluster.

Quick summary

- Purpose: Deploy a MSSQL instance for IAM data in a Kubernetes cluster.
- Argo CD compatible: `argocd/application.yml` creates an Argo CD Application pointing at `base/`.

Prerequisites

- A Kubernetes cluster and `kubectl` configured to talk to it.
- (Optional) An Argo CD installation to manage GitOps deployments.

Deploying

1) Deploy with Argo CD

- Import the Argo CD application into your Argo CD instance (either via UI or `kubectl`):

```bash
# Apply into the cluster where Argo CD is running
kubectl apply -f argocd/application.yml
```

Argo CD will create the `numeraid-data` namespace and sync the resources under `base/`.

1) Deploy manually with kubectl

```bash
# Create namespace and resources directly
kubectl apply -f base/namespace.yml
kubectl apply -f base/secret.yml   # edit secret before applying (see notes)
kubectl apply -f base/pvc.yml
kubectl apply -f base/deployment.yml
kubectl apply -f base/service.yml
```

Important configuration notes

- Secrets: `base/secret.yml` includes a template password that is intentionally non-production. Replace it with a strong password before applying, or generate the secret out-of-band using:

```bash
kubectl create secret generic iam-mssql-server-secret --from-literal=MSSQL_SA_PASSWORD='<your-strong-password>' -n numeraid-data
```

- Storage: `base/pvc.yml` requests `10Gi`. Confirm your cluster's StorageClass can satisfy the claim or edit the file to specify a `storageClassName` for your environment.
- Image: The deployment uses `ghcr.io/numeraid/identity-and-access-management-database-container:latest`. Ensure the image is accessible from your cluster (private registry auth may be required).
- Startup behaviour: the deployment includes a SQL-based `startupProbe`, `readinessProbe`, and `livenessProbe` so the database doesn't get marked ready before it can accept connections.

Files and purpose

- `argocd/application.yml`: Argo CD Application resource that points to `base/` in this repo and deploys into `numeraid-data`.
- `base/namespace.yml`: Namespace definition for the IAM DB resources.
- `base/secret.yml`: Kubernetes Secret for `MSSQL_SA_PASSWORD` (replace before use).
- `base/pvc.yml`: PersistentVolumeClaim used by the MSSQL pod for data persistence.
- `base/deployment.yml`: Deployment for the MSSQL container with resource requests/limits and probes.
- `base/service.yml`: ClusterIP service exposing port 1433 inside the cluster.

Next steps / recommendations

- Replace the placeholder password in `base/secret.yml` or create the secret out-of-band.
- Review resource requests/limits in `base/deployment.yml` to match your environment.
- Consider adding readiness/liveness or backup jobs if you plan to run production data workloads.

Support
If you want me to:

- add a kustomize overlay for environment-specific configs, or
- add a Helm chart or CI pipeline for automatic releases,
tell me which option you prefer and I will implement it.
