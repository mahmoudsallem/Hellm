# Minikube Deployment Guide

Complete guide for deploying the Task Manager application (React frontend + Flask backend + PostgreSQL) to Minikube.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Build and Push Docker Images](#build-and-push-docker-images)
- [Deployment Methods](#deployment-methods)
  - [Method 1: Using K8s Manifests](#method-1-using-k8s-manifests)
  - [Method 2: Using Helm (devops-app)](#method-2-using-helm-devops-app)
- [Accessing the Application](#accessing-the-application)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Required Tools
- **Minikube**: v1.30.0 or later
- **kubectl**: v1.27.0 or later
- **helm**: v3.12.0 or later (for Helm deployment)
- **Docker**: v24.0.0 or later
- **Docker Hub account**: For pushing images

### Installation
```powershell
# Install Minikube (Windows)
winget install Kubernetes.minikube

# Install kubectl
winget install Kubernetes.kubectl

# Install Helm
winget install Helm.Helm

# Verify installations
minikube version
kubectl version --client
helm version
docker --version
```

---

## Quick Start

```powershell
# 1. Start Minikube
minikube start --cpus=4 --memory=4096

# 2. Enable Ingress addon
minikube addons enable ingress

# 3. Build and push Docker images (see section below)

# 4. Deploy using Helm (recommended)
helm install devops-app d:\nti-final-project\helm\devops-app

# 5. Wait for pods to be ready
kubectl get pods -A -w

# 6. Get Minikube IP
minikube ip

# 7. Add to hosts file (run as Administrator)
Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value "`n$(minikube ip) frontend.nti.com backend.nti.com"

# 8. Access application
Start-Process "http://frontend.nti.com"
```

---

## Build and Push Docker Images

**IMPORTANT**: You need to build and push the images to Docker Hub before deploying.

### 1. Login to Docker Hub
```powershell
docker login
# Enter username: mostafakassas55
# Enter password: <your-password>
```

### 2. Build Frontend Image
```powershell
cd d:\nti-final-project\frontend

# Build the image
docker build -t mostafakassas55/nti_frontend:v1 .

# Push to Docker Hub
docker push mostafakassas55/nti_frontend:v1
```

### 3. Build Backend Image
```powershell
cd d:\nti-final-project\backend

# Build the image
docker build -t mostafakassas55/nti_backend:v1 .

# Push to Docker Hub
docker push mostafakassas55/nti_backend:v1
```

### 4. Verify Images
```powershell
# Check images exist locally
docker images | Select-String "nti_"

# Verify they're on Docker Hub
Start-Process "https://hub.docker.com/r/mostafakassas55/nti_frontend/tags"
Start-Process "https://hub.docker.com/r/mostafakassas55/nti_backend/tags"
```

---

## Deployment Methods

### Method 1: Using K8s Manifests

Deploy using raw Kubernetes manifest files from the `k8s` folder.

#### Step 1: Start Minikube and Enable Ingress
```powershell
# Start Minikube
minikube start --cpus=4 --memory=4096

# Enable ingress addon
minikube addons enable ingress

# Verify ingress controller is running
kubectl get pods -n ingress-nginx
```

#### Step 2: Create Namespaces
```powershell
# Create all three namespaces
kubectl create namespace frontend-ns
kubectl create namespace backend-ns
kubectl create namespace db-ns

# Label namespaces (required for network policies)
kubectl label namespace frontend-ns name=frontend-ns
kubectl label namespace backend-ns name=backend-ns
kubectl label namespace db-ns name=db-ns

# Verify namespaces
kubectl get namespaces --show-labels
```

####Step 3: Deploy Database (PostgreSQL)
```powershell
# Deploy PostgreSQL
kubectl apply -f d:\nti-final-project\k8s\db\

# Wait for PostgreSQL to be ready
kubectl wait --for=condition=ready pod -l app=postgres -n db-ns --timeout=180s

# Verify PostgreSQL is running
kubectl get pods -n db-ns
kubectl get pvc -n db-ns  # Check persistent volume claim
```

#### Step 4: Deploy Backend
```powershell
# Deploy backend application
kubectl apply -f d:\nti-final-project\k8s\backend\

# Wait for backend to be ready
kubectl wait --for=condition=ready pod -l app=backend -n backend-ns --timeout=180s

# Check backend logs
kubectl logs -f deployment/backend -n backend-ns
```

#### Step 5: Deploy Frontend
```powershell
# Deploy frontend application
kubectl apply -f d:\nti-final-project\k8s\frontend\

# Wait for frontend to be ready
kubectl wait --for=condition=ready pod -l app=frontend -n frontend-ns --timeout=120s

# Verify frontend is running
kubectl get pods -n frontend-ns
```

#### Step 6: Deploy Network Policies
```powershell
# Apply network policies
kubectl apply -f d:\nti-final-project\k8s\network-policies\

# Verify network policies
kubectl get networkpolicies -A
```

#### Step 7: Verify Deployment
```powershell
# Get all resources
kubectl get all -A | Select-String "frontend-ns|backend-ns|db-ns"

# Check ingress
kubectl get ingress -A

# Get Minikube IP
minikube ip
```

#### Step 8: Update Hosts File
Run PowerShell as Administrator:
```powershell
# Get Minikube IP
$minikubeIP = minikube ip

# Add entries to hosts file
Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value "`n$minikubeIP frontend.nti.com backend.nti.com"

# Verify
Get-Content C:\Windows\System32\drivers\etc\hosts | Select-String "nti.com"
```

---

### Method 2: Using Helm (devops-app)

Deploy using the unified Helm chart which manages all components.

#### Step 1: Start Minikube and Enable Ingress
```powershell
# Start Minikube
minikube start --cpus=4 --memory=4096

# Enable ingress addon
minikube addons enable ingress
```

#### Step 2 Deploy with Helm
```powershell
# Install the Helm chart (creates namespaces automatically)
helm install devops-app d:\nti-final-project\helm\devops-app

# Watch deployment progress
kubectl get pods -A -w
```

#### Step 3: Verify Deployment
```powershell
# Check Helm release
helm list

# Get all resources
kubectl get all -A | Select-String "frontend-ns|backend-ns|db-ns"

# Check pods are running
kubectl get pods -n frontend-ns
kubectl get pods -n backend-ns
kubectl get pods -n db-ns
```

#### Step 4: Update Hosts File
Run PowerShell as Administrator:
```powershell
# Get Minikube IP
$minikubeIP = minikube ip

# Add entries to hosts file
Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value "`n$minikubeIP frontend.nti.com backend.nti.com"
```

#### Step 5: Access Application
```powershell
# Open frontend in browser
Start-Process "http://frontend.nti.com"

# Test backend API
Invoke-WebRequest -Uri "http://backend.nti.com/health"
```

---

### Method 3: Using Separate Helm Charts

Deploy using individual Helm charts for granular control.

#### Step 1: Start Minikube and Enable Ingress
```powershell
minikube start --cpus=4 --memory=4096
minikube addons enable ingress
```

#### Step 2: Deploy Components
Deploy in the following order to ensure dependencies are met:

```powershell
# 1. Deploy Database (must be first)
helm install db d:\nti-final-project\helm\db

# Wait for PostgreSQL to be ready
kubectl wait --for=condition=ready pod -l app=postgres -n db-ns --timeout=180s

# 2. Deploy Backend
helm install backend d:\nti-final-project\helm\backend

# Wait for Backend to be ready
kubectl wait --for=condition=ready pod -l app=backend -n backend-ns --timeout=180s

# 3. Deploy Frontend
helm install frontend d:\nti-final-project\helm\frontend
```

#### Step 3: Verify Deployment
```powershell
# List all releases
helm list

# Verify pods
kubectl get pods -A | Select-String "frontend-ns|backend-ns|db-ns"
```

#### Step 4: Update Hosts File
Run PowerShell as Administrator:
```powershell
# Get Minikube IP
$minikubeIP = minikube ip

# Add entries to hosts file
Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value "`n$minikubeIP frontend.nti.com backend.nti.com"
```

---

## Accessing the Application

After deployment, you can access the application through these URLs:

### Frontend (Task Manager UI)
- **URL**: http://frontend.nti.com
- **What it does**: Provides the web interface to create, view, update, and delete tasks

### Backend API
- **Base URL**: http://backend.nti.com
- **Health Check**: http://backend.nti.com/health
- **API Documentation**: http://backend.nti.com/api/docs
- **Tasks Endpoint**: http://backend.nti.com/api/tasks

### Testing the Frontend
1. Open http://frontend.nti.com in your browser
2. You should see the Task Manager interface
3. Try creating a new task:
   - Enter a task title
   - Add a description (optional)
   - Click "Add Task"
4. The task should appear in the list below
5. You can edit, complete, or delete tasks

### Testing the Backend API
```powershell
# Test health endpoint
Invoke-WebRequest -Uri "http://backend.nti.com/health" | Select-Object -ExpandProperty Content

# Get all tasks
Invoke-WebRequest -Uri "http://backend.nti.com/api/tasks" | Select-Object -ExpandProperty Content

# Create a new task
$body = @{
    title = "Test Task"
    description = "Created via API"
} | ConvertTo-Json

Invoke-WebRequest -Uri "http://backend.nti.com/api/tasks" -Method POST -Body $body -ContentType "application/json"
```

---

## Verification

### 1 Check Pod Status
```powershell
# All pods should be in Running status
kubectl get pods -A | Select-String "frontend-ns|backend-ns|db-ns"
```

### 2. Check Services
```powershell
# Verify services are created
kubectl get svc -n frontend-ns
kubectl get svc -n backend-ns
kubectl get svc -n db-ns
```

### 3. Check Ingress
```powershell
# Verify ingress resources
kubectl get ingress -A

# Describe ingress for details
kubectl describe ingress frontend-ingress -n frontend-ns
kubectl describe ingress backend-ingress -n backend-ns
```

### 4. Check Persistent Volumes
```powershell
# Verify PostgreSQL has persistent storage
kubectl get pvc -n db-ns
kubectl get pv
```

### 5. Test Database Connection
```powershell
# Test from backend pod
kubectl exec -it deployment/backend -n backend-ns -- python -c "
from sqlalchemy import create_engine, text
import os
host = os.getenv('POSTGRES_HOST')
port = os.getenv('POSTGRES_PORT')
db = os.getenv('POSTGRES_DB')
user = os.getenv('POSTGRES_USER')
password = os.getenv('POSTGRES_PASSWORD')
url = f'postgresql://{user}:{password}@{host}:{port}/{db}'
engine = create_engine(url)
with engine.connect() as conn:
    result = conn.execute(text('SELECT 1'))
    print('Database connection successful!')
"
```

### 6. Test Network Policies
```powershell
# Test that frontend can reach backend (should succeed)
kubectl run test-curl --image=curlimages/curl:latest --rm -it --restart=Never -n frontend-ns -- curl -s http://backend.backend-ns.svc.cluster.local:5000/health

# Test that external pod cannot reach database (should fail/timeout)
kubectl run test-curl2 --image=curlimages/curl:latest --rm -it --restart=Never -- nc -zv postgres-0.postgres.db-ns.svc.cluster.local 5432
```

---

## Troubleshooting

### Pods Not Starting

**Problem**: Pods are stuck in Pending or ImagePullBackOff

**Solutions**:
```powershell
# Check pod details
kubectl describe pod <pod-name> -n <namespace>

# Check if images are accessible
docker pull mostafakassas55/nti_frontend:v1
docker pull mostafakassas55/nti_backend:v1

# If images are private, make them public on Docker Hub
```

### Backend Can't Connect to Database

**Problem**: Backend logs show database connection errors

**Solutions**:
```powershell
# Check PostgreSQL is running
kubectl get pods -n db-ns

# Check PostgreSQL logs
kubectl logs -f statefulset/postgres -n db-ns

# Verify secrets are created correctly
kubectl get secret app-secret -n backend-ns -o yaml
kubectl get secret db-secret -n db-ns -o yaml

# Check environment variables in backend pod
kubectl exec deployment/backend -n backend-ns -- env | grep POSTGRES
```

### Ingress Not Working

**Problem**: Cannot access frontend/backend via hostnames

**Solutions**:
```powershell
# Check ingress controller is running
kubectl get pods -n ingress-nginx

# If not enabled, enable it
minikube addons enable ingress

# Check ingress resources
kubectl get ingress -A

# Verify hosts file has correct entries
Get-Content C:\Windows\System32\drivers\etc\hosts | Select-String "nti.com"

# Verify Minikube IP
minikube ip

# Test with Minikube IP directly
Start-Process "http://$(minikube ip)"
```

###Database Data Not Persisting

**Problem**: Tasks disappear when PostgreSQL pod restarts

**Solutions**:
```powershell
# Check PVC exists
kubectl get pvc -n db-ns

# Check PV is bound
kubectl get pv

# Describe PVC for details
kubectl describe pvc postgres-data-postgres-0 -n db-ns

# If using Minikube, ensure persistent storage is enabled
minikube addons list | Select-String "storage"
```

### Network Policy Blocking Traffic

**Problem**: Frontend cannot reach backend or backend cannot reach database

**Solutions**:
```powershell
# Check namespaces have correct labels
kubectl get namespaces --show-labels

# Add missing labels
kubectl label namespace frontend-ns name=frontend-ns
kubectl label namespace backend-ns name=backend-ns
kubectl label namespace db-ns name=db-ns

# Temporarily disable network policies to test
kubectl delete networkpolicies --all -A

# Re-apply after testing
kubectl apply -f d:\nti-final-project\k8s\network-policies\
```

### Helm Installation Issues

**Problem**: Helm install fails with validation errors

**Solutions**:
```powershell
# Lint the Helm chart first
helm lint d:\nti-final-project\helm\devops-app

# Dry-run to see what would be created
helm install devops-app d:\nti-final-project\helm\devops-app --dry-run --debug

# If already installed, upgrade instead
helm upgrade devops-app d:\nti-final-project\helm\devops-app

# Or uninstall and reinstall
helm uninstall devops-app
kubectl delete namespace frontend-ns backend-ns db-ns
helm install devops-app d:\nti-final-project\helm\devops-app
```

### Viewing Logs

```powershell
# Frontend logs
kubectl logs -f deployment/frontend -n frontend-ns

# Backend logs
kubectl logs -f deployment/backend -n backend-ns

# PostgreSQL logs
kubectl logs -f statefulset/postgres -n db-ns

# Init container logs (if pod fails to start)
kubectl logs <pod-name> -n <namespace> -c wait-for-postgres
```

### Clean Up and Restart

```powershell
# Using K8s manifests
kubectl delete -f d:\nti-final-project\k8s\frontend\
kubectl delete -f d:\nti-final-project\k8s\backend\
kubectl delete -f d:\nti-final-project\k8s\db\
kubectl delete -f d:\nti-final-project\k8s\network-policies\
kubectl delete namespace frontend-ns backend-ns db-ns

# Using Helm
helm uninstall devops-app
kubectl delete namespace frontend-ns backend-ns db-ns

# Delete all PVs/PVCs
kubectl delete pvc --all -A
kubectl delete pv--all

# Restart Minikube
minikube stop
minikube delete
minikube start --cpus=4 --memory=4096
```

---

## Additional Notes

### Security Considerations
- All secrets are base64 encoded in Kubernetes
- Non-root users are configured in all Docker containers
- Network policies restrict traffic between pods
- PostgreSQL data is persisted in a PersistentVolume

### Resource Requirements
- **Minimum**: 2 CPUs, 2GB RAM
- **Recommended**: 4 CPUs, 4GB RAM
- **Disk Space**: ~2GB for images and persistent volumes

### Scaling
```powershell
# Scale frontend replicas
kubectl scale deployment frontend -n frontend-ns --replicas=3

# Scale backend replicas
kubectl scale deployment backend -n backend-ns --replicas=2

# HPA will auto-scale if CPU exceeds 50%
kubectl get hpa -A
```

### Updating Images
```powershell
# After building new images

# For K8s manifests
kubectl set image deployment/frontend frontend=mostafakassas55/nti_frontend:v2 -n frontend-ns
kubectl set image deployment/backend backend=mostafakassas55/nti_backend:v2 -n backend-ns

# For Helm
helm upgrade devops-app d:\nti-final-project\helm\devops-app --set frontend.image.tag=v2 --set backend.image.tag=v2
```

---

## Support

For issues or questions:
1. Check the troubleshooting section above
2. Review pod logs: `kubectl logs -f <pod-name> -n <namespace>`
3. Check Kubernetes events: `kubectl get events -n <namespace> --sort-by='.lastTimestamp'`
4. Verify resource status: `kubectl get all -A`
