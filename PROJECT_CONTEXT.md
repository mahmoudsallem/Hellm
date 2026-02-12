# Project Context for Claude Code

## Folder Location
```
/media/mahmoud/life/NTI/test/Hellm_apps
```

## Project Structure
```
Hellm_apps/
├── helm/
│   ├── Dev/
│   │   ├── backend/      # Backend API Helm chart
│   │   ├── frontend/     # Frontend Helm chart (currently deploys API)
│   │   └── db/           # PostgreSQL database Helm chart
│   ├── Prod/
│   │   ├── backend/
│   │   ├── frontend/
│   │   └── db/
│   └── README.md
├── k8s/                  # Raw Kubernetes manifests (alternative to Helm)
└── DEPLOYMENT.md
```

## AWS ECR Configuration
- **Region:** us-east-1
- **Account ID:** 288378107043
- **Repositories:**
  - `final/app` - Flask API (Task Manager) - Currently used as "frontend"
  - `final/backend` - Flask API (has libpq.so.5 issue - needs rebuild)

### ECR Authentication
```bash
# Generate ECR token (expires every 12 hours)
ECR_TOKEN=$(aws ecr get-login-password --region us-east-1)

# Deploy with Helm
helm upgrade --install <release> ./helm/Dev/<chart> --set imageCredentials.password="$ECR_TOKEN"
```

## Current Issues

### 1. Backend Image Missing libpq
The `final/backend` image crashes with:
```
ImportError: libpq.so.5: cannot open shared object file: No such file or directory
```
**Fix:** Add to Dockerfile:
```dockerfile
RUN apt-get update && apt-get install -y libpq5
```
Or change `psycopg2` to `psycopg2-binary` in requirements.txt

### 2. No Real Frontend
The `final/app` image is a Flask API backend, not a frontend with HTML.
- Serves API at `/api/tasks`
- Swagger docs at `/api/docs`
- Health check at `/health`

**Need:** Build and push an actual frontend (React/Vue/Angular) to ECR

## Kubernetes Namespaces
- `frontend-ns` - Frontend service
- `backend-ns` - Backend API
- `db-ns` - PostgreSQL database

## Helm Deployment Commands
```bash
# Deploy all services
helm upgrade --install db ./helm/Dev/db
helm upgrade --install backend ./helm/Dev/backend --set imageCredentials.password="$ECR_TOKEN"
helm upgrade --install frontend ./helm/Dev/frontend --set imageCredentials.password="$ECR_TOKEN"

# Check status
kubectl get pods -A | grep -E "frontend|backend|postgres"
```

## Git Branch
- Working branch: `dev`
- Remote: `origin` (GitHub)

## Port Configuration
| Service | Container Port | Service Port |
|---------|---------------|--------------|
| frontend | 5000 | 5000 |
| backend | 5000 | 5000 |
| postgres | 5432 | 5432 |
