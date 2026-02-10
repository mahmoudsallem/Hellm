# DevOps App Helm Chart - Installation Instructions

## Prerequisites

1. **Kubernetes Cluster**: A running Kubernetes cluster (Minikube, Kind, EKS, GKE, etc.)
2. **Helm 3**: Installed and configured
3. **kubectl**: Configured to access your cluster
4. **NGINX Ingress Controller**: Installed in the cluster

## Quick Start

### 1. Install the Chart

```bash
# Navigate to the helm directory
cd ./Project/helm

# Install with default values (requires passing image credentials password)
helm install devops-app ./devops-app --set imageCredentials.password=YOUR_DOCKER_PASSWORD

# Or install with custom release name
helm install my-release ./devops-app --set imageCredentials.password=YOUR_DOCKER_PASSWORD
```

### 1b. Image Credentials (Required for Private Registries)

The chart uses Docker Hub credentials for pulling images. **Never commit passwords to git!**

```bash
# Pass password via CLI
helm install devops-app ./devops-app \
  --set imageCredentials.password=YOUR_DOCKER_PASSWORD

# Or for frontend/backend separated charts
helm install frontend ./frontend \
  --set imageCredentials.password=YOUR_DOCKER_PASSWORD

helm install backend ./backend \
  --set imageCredentials.password=YOUR_DOCKER_PASSWORD
```

The credentials are configured in `values.yaml`:
```yaml
imageCredentials:
  enabled: true
  name: regcred
  registry: "https://index.docker.io/v1/"
  username: "mostafakassas55"
  password: ""  # Passed via CLI or ArgoCD
  email: "elkassasmostafa29@gmail.com"
```

### 1c. TLS Configuration

The frontend ingress is configured with TLS. The certificate and key are stored in `values.yaml` as base64-encoded strings.

To generate a new self-signed certificate:
```bash
# Generate certificate for frontend.nti.com
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt \
  -subj "/CN=frontend.nti.com"

# Base64 encode for values.yaml
cat tls.crt | base64 -w 0
cat tls.key | base64 -w 0
```

The TLS configuration in `values.yaml`:
```yaml
frontend:
  tls:
    enabled: true
    secretName: frontend-tls-secret
    crt: "<base64-encoded-certificate>"
    key: "<base64-encoded-key>"
```

### 2. Verify Installation

```bash
# Check all pods are running
kubectl get pods -n frontend-ns
kubectl get pods -n backend-ns
kubectl get pods -n db-ns

# Check services
kubectl get svc -A | grep -E "frontend|backend|postgres|redis"

# Check ingress
kubectl get ingress -A
```

### 3. Access the Application

Add the following to your hosts file (or configure DNS):

**Windows**: `C:\Windows\System32\drivers\etc\hosts`  
**Linux/Mac**: `/etc/hosts`

```
<INGRESS_IP>  frontend.nti.com
<INGRESS_IP>  backend.nti.com
```

Get the ingress IP:
```bash
# For Minikube
minikube ip

# For cloud providers, check the ingress service
kubectl get svc -n ingress-nginx
```

Then open in browser:
- **Frontend**: http://frontend.nti.com
- **Backend API**: http://backend.nti.com

## Customization

### Override Values

Create a custom `my-values.yaml`:

```yaml
frontend:
  replicaCount: 2
  image:
    tag: v5

backend:
  replicaCount: 2

database:
  postgres:
    password: mysecurepassword
  redis:
    password: myredispassword
```

Install with custom values:
```bash
helm install devops-app ./devops-app -f my-values.yaml
```

### Common Overrides

| Parameter | Description | Default |
|-----------|-------------|---------|
| `frontend.replicaCount` | Frontend replicas | `1` |
| `frontend.image.tag` | Frontend image tag | `v4` |
| `backend.replicaCount` | Backend replicas | `1` |
| `backend.image.tag` | Backend image tag | `v1` |
| `database.postgres.password` | Postgres password | `apppass` |
| `database.redis.password` | Redis password | `redispass` |
| `frontend.ingress.host` | Frontend hostname | `frontend.nti.com` |
| `backend.ingress.host` | Backend hostname | `backend.nti.com` |

## Upgrade

```bash
helm upgrade devops-app ./devops-app -f my-values.yaml
```

## Uninstall

```bash
# Uninstall the release
helm uninstall devops-app

# Delete the namespaces (optional - removes all data)
kubectl delete ns frontend-ns backend-ns db-ns
```

## Troubleshooting

### Pods not starting
```bash
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>
```

### Backend waiting for databases
The backend has init containers that wait for Redis and Postgres. Check their logs:
```bash
kubectl logs <backend-pod> -n backend-ns -c wait-for-services
```

### Network policies blocking traffic
If using a CNI that supports NetworkPolicies (Calico, Cilium, etc.), ensure namespaces are labeled:
```bash
kubectl label namespace frontend-ns name=frontend-ns
kubectl label namespace backend-ns name=backend-ns
kubectl label namespace db-ns name=db-ns
```
