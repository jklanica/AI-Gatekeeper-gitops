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
Because this repository uses **Sealed Secrets**, the `sealed-secret.yaml` file currently in this repository is encrypted specifically for the author's cluster and **will not work on your machine**. 

Before deploying, you must generate your own encrypted secrets:

1. Copy the secret template:
   ```bash
   cp apps/ai-gatekeeper/base/config/secret.template.yaml apps/ai-gatekeeper/base/config/my-secret.yaml
   ```
2. Open `my-secret.yaml` and fill in your actual passwords.
3. Install the Sealed Secrets controller into your cluster (so you can get the public key):
   ```bash
   kubectl apply -k infrastructure/security/sealed-secrets/base --enable-helm
   ```
4. Encrypt your secret using the `kubeseal` CLI tool:
   ```bash
   kubeseal -f apps/ai-gatekeeper/base/config/my-secret.yaml -w apps/ai-gatekeeper/base/config/sealed-secret.yaml
   ```
5. **IMPORTANT:** Delete your plaintext secret so you don't accidentally commit it!
   ```bash
   rm apps/ai-gatekeeper/base/config/my-secret.yaml
   ```

### 2. 🚀 Deploy the Cluster
You can deploy the entire production stack manually:
```bash
kubectl apply -k clusters/production --enable-helm
```
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
