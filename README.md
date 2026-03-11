# 🐘 database-k8s

Kubernetes configuration for deploying a **PostgreSQL** database on a K8s cluster. This repo contains all the manifests needed to run a production-ready Postgres instance with persistent storage and secure credentials management.

---

## 📁 File Overview

| File | Purpose |
|------|---------|
| `db-service.yml` | Exposes the PostgreSQL pod inside the cluster via a Kubernetes Service |
| `db-deploy.yml` | Defines the Deployment — replica count, container image, and pod spec |
| `db-auth.yml` | Stores database credentials securely as a Kubernetes Secret |
| `db-storage.yml` | Provisions persistent storage via a PersistentVolumeClaim (PVC) |

---

## 📄 File Breakdown

### `db-service.yml` — Database Service
Defines a Kubernetes **Service** that routes internal cluster traffic to the PostgreSQL pod.
- Sets the target port that Postgres listens on (`5432`)
- Uses label selectors to find and forward traffic to the correct pod
- Keeps the database accessible to other services (e.g. an auth or API service) without exposing it externally

### `db-deploy.yml` — Deployment
Defines the **Deployment** that manages the PostgreSQL pod lifecycle.
- Specifies the number of **replicas** to run
- Pulls the official **PostgreSQL container image** from Docker Hub
- Mounts the persistent volume so data survives pod restarts
- Injects credentials from the Secret (`db-auth.yml`) as environment variables

### `db-auth.yml` — Credentials Secret
A Kubernetes **Secret** that stores sensitive database credentials.
- Holds values like `POSTGRES_USER`, `POSTGRES_PASSWORD`, and `POSTGRES_DB`
- Values are base64-encoded (not encrypted — use Sealed Secrets or Vault in production)
- Referenced by the Deployment so credentials are never hardcoded in the pod spec

### `db-storage.yml` — Persistent Volume Claim
A **PersistentVolumeClaim (PVC)** that requests durable storage from the cluster.
- Defines the storage size needed for the database
- Ensures data is not lost when a pod is deleted or rescheduled
- Mounted into the PostgreSQL container at `/var/lib/postgresql/data`

---

## 🚀 Deploying

Apply the manifests in this order to avoid dependency issues:

```bash
# 1. Create the secret first (Deployment depends on it)
kubectl apply -f db-auth.yml

# 2. Provision storage
kubectl apply -f db-storage.yml

# 3. Deploy the PostgreSQL pod
kubectl apply -f db-deploy.yml

# 4. Expose the database via a Service
kubectl apply -f db-service.yml
```

---

## ✅ Verify the Setup

```bash
# Check that the pod is running
kubectl get pods

# Check the service is created
kubectl get svc

# Check the PVC is bound to storage
kubectl get pvc

# View logs from the Postgres container
kubectl logs deployment/postgres-deployment
```

---

## 🔐 Security Notes

- The `db-auth.yml` Secret uses base64 encoding, which is **not encryption**. For production environments, consider:
  - [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets)
  - [HashiCorp Vault](https://www.vaultproject.io/)
  - External Secrets Operator
- Avoid committing `db-auth.yml` with real credentials to a public repository. Add it to `.gitignore` or use a secrets manager.

---

## 🛠 Prerequisites

- A running Kubernetes cluster (local: [minikube](https://minikube.sigs.k8s.io/) or [kind](https://kind.sigs.k8s.io/), cloud: EKS / GKE / AKS)
- `kubectl` configured and pointing at your cluster
- Sufficient cluster storage to satisfy the PVC request in `db-storage.yml`
