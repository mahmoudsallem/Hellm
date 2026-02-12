# DevOps Application Helm Chart

A comprehensive Helm chart for deploying the secure DevOps application on Kubernetes. This chart manages the deployment of the Frontend, Backend, Database layers, and associated security resources.

## 📂 Chart Structure

- **templates/**: Contains the Kubernetes manifest templates.
- **values.yaml**: The default configuration values for the chart.
- **Chart.yaml**: Metadata about the chart.

## ⚙️ Configuration

The following table lists the configurable parameters of the `devops-app` chart and their default values (as seen in `values.yaml`).

| Parameter | Description | Default |
|-----------|-------------|---------|
| `namespaces.frontend` | Namespace for Frontend resources | `frontend-ns` |
| `namespaces.backend` | Namespace for Backend resources | `backend-ns` |
| `namespaces.db` | Namespace for Database resources | `db-ns` |
| `frontend.image.repository` | Frontend Docker Image | `mostafakassas55/mfrontend` |
| `frontend.ingress.enabled` | Enable Ingress resource | `true` |
| `frontend.tls.enabled` | Enable TLS for Ingress | `true` |
| `backend.image.repository` | Backend Docker Image | `mostafakassas55/mbackend` |
| `imageCredentials.enabled` | Enable Private Registry Pull Secrets | `true` |
| `networkPolicies.enabled` | Enable Kubernetes Network Policies | `true` |

## 🚀 Deployment Guide

### Authentication Secrets
Before deploying, ensure you have set the correct credentials for the image pull secret if `imageCredentials.enabled` is `true`.

**Using CLI:**
```bash
helm install devops-app ./devops-app \
  --set imageCredentials.password="YOUR_ACTUAL_PASSWORD" \
  --set imageCredentials.enabled=true
```

**Using a separate Values file:**
Create a `secrets.yaml` (do NOT commit this to git):
```yaml
imageCredentials:
  password: "super-secure-password"
```
Then run:
```bash
helm install devops-app ./devops-app -f values.yaml -f secrets.yaml
```

### Upgrading the Release
To upgrade an existing release:
```bash
helm upgrade devops-app ./devops-app
```

### Uninstallation
To completely remove the application:
```bash
helm uninstall devops-app
```

## 🔐 Security Configuration

### TLS Certificates
The Helm chart expects the TLS certificate and key to be provided in `values.yaml` (as base64 encoded strings) or injected via an external secret management system like Vault.

- **`frontend.tls.crt`**: The base64 encoded certificate.
- **`frontend.tls.key`**: The base64 private key.

### Network Policies
Network policies are enabled by default to enforce the "Zero Trust" model inside the cluster.
- **Frontend** can only be accessed via the Ingress Controller.
- **Backend** can only be accessed by the Frontend pods.
- **Databases** can only be accessed by the Backend pods.

---
*Last tested: 2026-02-12*
