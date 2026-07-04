# AI-Gatekeeper GitOps

This repository contains the Kubernetes infrastructure and deployment manifests for the AI-Gatekeeper application. It is managed using **Kustomize** and designed to be deployed via a GitOps workflow (using **ArgoCD**).

## Architecture
- **GitOps Engine:** ArgoCD (`infrastructure/controllers/argocd`)
- **Ingress Controller:** Ingress-Nginx (`infrastructure/controllers/ingress-nginx`)
- **TLS Automation:** Cert-Manager (`infrastructure/security/cert-manager`)
- **Secrets Management:** Sealed Secrets (`infrastructure/security/sealed-secrets`)

## How to Run It Locally

If you want to spin this cluster up on your local machine (e.g. Docker Desktop), follow these steps:

### 1. 🔐 Secrets Configuration
Because this repository uses **Sealed Secrets**, the `sealed-secret.yaml` file in this repository is encrypted and permanently tied to a specific "Master Key" (private key) that resides in the cluster.

If you ever completely wipe your cluster, that active master key is destroyed, and the cluster will generate a new one upon reboot. To ensure `sealed-secret.yaml` continues to work, you must restore the original master key from your backup **before** deploying the rest of the cluster.

1. First, deploy **only** the Sealed Secrets controller to create its namespace and CRDs:
   ```bash
   kustomize build --enable-helm infrastructure/security/sealed-secrets/base | kubectl apply -f -
   ```
2. Locate your backed-up master key (e.g., `master-key.backup.yaml`). **Never commit this file to Git.**
3. Apply the backup key to the cluster manually:
   ```bash
   kubectl apply -f master-key.backup.yaml
   ```
4. Restart the `sealed-secrets` controller so it picks up the restored key:
   ```bash
   kubectl rollout restart deployment -n sealed-secrets sealed-secrets
   ```

*(Note: If you are setting this up for the first time on a brand new project, you will need to generate a new secret from `secret.template.yaml`, encrypt it with `kubeseal`, and back up your cluster's new master key).*

### 2. 🚀 Deploy the Cluster
Now that the secrets controller is running with the correct master key, you can deploy the entire production stack:
```bash
kustomize build --enable-helm clusters/production | kubectl apply -f -
```
> **Note:** The first time you run this command on a fresh cluster, you will likely see errors about `resource mapping not found` for Custom Resources like `Application` or `ClusterIssuer`. This is normal! It just means Kubernetes tried to create those resources before the ArgoCD and Cert-Manager controllers finished booting up. **Simply wait 30 seconds and run the exact same command a second time**, and everything will succeed.

*(Alternatively, you can deploy the ArgoCD Root Application in `clusters/production/root-application.yaml` to let ArgoCD take over).*

### 3. 🌐 Local Domain Mapping
Our ingress controller listens for `local.ai-gatekeeper.com`. To access the application locally, map this domain to your localhost.

Edit your `/etc/hosts` file (`sudo nano /etc/hosts`) and add:
```
127.0.0.1 local.ai-gatekeeper.com
```

### 4. 🧪 Access the Application
Open your browser and navigate to:
`https://local.ai-gatekeeper.com`

*(Note: Because we are using a Self-Signed certificate for local development, your browser will warn you that the connection is not private. Click "Advanced" and "Proceed".)*
