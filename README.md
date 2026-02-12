# Hellm Apps - Kubernetes Deployment

A production-ready, multi-environment Kubernetes deployment for a three-tier Task Manager application using Helm charts, with automated CI/CD pipelines.

## Architecture Overview

```
                                    +------------------+
                                    |   GitHub Repo    |
                                    +--------+---------+
                                             |
                         +-------------------+-------------------+
                         |                                       |
                   Push to Dev                            PR to Main
                         |                                       |
                         v                                       v
              +----------+----------+               +------------+------------+
              |    CICD Pipeline    |               | Validate-Main Pipeline  |
              |  (Helm Validation)  |               |   (PR Gate Checks)      |
              +----------+----------+               +------------+------------+
                         |                                       |
                         v                                       v
              +----------+-------------------------------------------+
              |                    Kubernetes Cluster                 |
              |  +---------------+  +---------------+  +------------+ |
              |  | frontend-ns   |  | backend-ns    |  | db-ns      | |
              |  |               |  |               |  |            | |
              |  | React App     |->| Flask API     |->| PostgreSQL | |
              |  | (Port 80)     |  | (Port 5000)   |  | (Port 5432)| |
              |  +---------------+  +---------------+  +------------+ |
              +-------------------------------------------------------+
```

## Application Architecture

```
+-------------------------------------------------------------------+
|                         KUBERNETES CLUSTER                         |
+-------------------------------------------------------------------+
|                                                                    |
|  +------------------+    +------------------+    +---------------+ |
|  |   INGRESS-NGINX  |    |   INGRESS-NGINX  |    |               | |
|  | frontend.nti.com |    | backend.nti.com  |    |               | |
|  +--------+---------+    +--------+---------+    |               | |
|           |                       |              |               | |
|           v                       v              |               | |
|  +------------------+    +------------------+    +---------------+ |
|  |   frontend-ns    |    |   backend-ns     |    |    db-ns      | |
|  |                  |    |                  |    |               | |
|  | +--------------+ |    | +--------------+ |    | +-----------+ | |
|  | |   React App  | |--->| |  Flask API   | |--->| | PostgreSQL| | |
|  | |   Port: 80   | |    | |  Port: 5000  | |    | | Port: 5432| | |
|  | +--------------+ |    | +--------------+ |    | +-----------+ | |
|  |                  |    |                  |    |               | |
|  | - Deployment     |    | - Deployment     |    | - StatefulSet | |
|  | - Service        |    | - Service        |    | - Service     | |
|  | - Ingress        |    | - ConfigMap      |    | - ConfigMap   | |
|  | - HPA            |    | - Secret         |    | - Secret      | |
|  | - NetworkPolicy  |    | - HPA            |    | - PVC (1Gi)   | |
|  +------------------+    | - NetworkPolicy  |    | - NetworkPolicy|
|                          +------------------+    +---------------+ |
+-------------------------------------------------------------------+
```

## CI/CD Pipelines

### CICD Pipeline (Dev Branch)

Triggers on push to `dev` branch when changes are made to `helm/**` or workflow files.

```
+------------------------------------------------------------------+
|                    CICD PIPELINE (dev branch)                     |
+------------------------------------------------------------------+
|                                                                   |
|  +--------------------+                                           |
|  | Stage 1: Validate  |                                           |
|  |    Structure       |                                           |
|  | - Check Chart.yaml |                                           |
|  | - Check values.yaml|                                           |
|  | - Check templates/ |                                           |
|  | - Validate YAML    |                                           |
|  +---------+----------+                                           |
|            |                                                      |
|            v                                                      |
|  +---------+----------+                                           |
|  | Stage 2: Validate  |                                           |
|  |   Docker Images    |                                           |
|  | - Check ECR images |                                           |
|  | - Backend image    |                                           |
|  | - Frontend image   |                                           |
|  | - Database image   |                                           |
|  +---------+----------+                                           |
|            |                                                      |
|            v                                                      |
|  +---------+----------+                                           |
|  | Stage 3: Helm Lint |                                           |
|  | - Lint Dev charts  |                                           |
|  | - Lint Prod charts |                                           |
|  | - Strict mode      |                                           |
|  +---------+----------+                                           |
|            |                                                      |
|            v                                                      |
|  +---------+----------+                                           |
|  | Stage 4: Helm      |                                           |
|  |    Template        |                                           |
|  | - Render templates |                                           |
|  | - Validate output  |                                           |
|  +---------+----------+                                           |
|            |                                                      |
|            v                                                      |
|  +---------+----------+                                           |
|  |   CI Success       |                                           |
|  +--------------------+                                           |
+------------------------------------------------------------------+
```

### Validate-Main Pipeline (PR to Main)

Triggers when a PR is opened or marked ready for review targeting `main` branch.

```
+------------------------------------------------------------------+
|              VALIDATE-MAIN PIPELINE (PR to main)                  |
+------------------------------------------------------------------+
|                                                                   |
|  +--------------------+                                           |
|  | Stage 1: Validate  |                                           |
|  |   Docker Images    |                                           |
|  | - AWS ECR auth     |                                           |
|  | - Verify images    |                                           |
|  +---------+----------+                                           |
|            |                                                      |
|            v                                                      |
|  +---------+----------+                                           |
|  | Stage 2: Helm      |                                           |
|  |   Validation       |                                           |
|  | - Lint all charts  |                                           |
|  | - Template tests   |                                           |
|  +---------+----------+                                           |
|            |                                                      |
|            v                                                      |
|  +---------+----------+                                           |
|  |     PR Ready       |                                           |
|  | - Ready to merge   |                                           |
|  +--------------------+                                           |
+------------------------------------------------------------------+
```

## Project Structure

```
Hellm_apps/
├── .github/
│   └── workflows/
│       ├── CICD.yaml              # Dev branch CI/CD pipeline
│       └── validate-main.yaml     # PR validation pipeline
├── helm/
│   ├── Dev/                       # Development environment
│   │   ├── backend/               # Flask API chart
│   │   ├── db/                    # PostgreSQL chart
│   │   └── frontend/              # React app chart
│   ├── Prod/                      # Production environment
│   │   ├── backend/
│   │   ├── db/
│   │   └── frontend/
│   ├── README.md
│   └── INSTRUCTIONS.md
├── k8s/                           # Raw Kubernetes manifests
│   ├── frontend/
│   │   backend/
│   ├── db/
│   └── network-policies/
├── DEPLOYMENT.md                  # Deployment guide
└── README.md                      # This file
```

## Components

| Component | Technology | Port | Image Source |
|-----------|------------|------|--------------|
| Frontend | React | 80 | Amazon ECR |
| Backend | Flask/Python | 5000 | Amazon ECR |
| Database | PostgreSQL 15 | 5432 | Docker Hub |

## Environments

| Environment | Namespace | Image Tags | Purpose |
|-------------|-----------|------------|---------|
| Dev | *-ns | Commit hash | Development & testing |
| Prod | *-ns | latest | Production deployment |

## Network Policies

```
+-------------+       +-------------+       +-------------+
|   Ingress   | ----> |  Frontend   | ----> |   Backend   |
|   (NGINX)   |       |  (React)    |       |   (Flask)   |
+-------------+       +-------------+       +------+------+
                                                   |
                                                   v
                                            +------+------+
                                            |  Database   |
                                            | (PostgreSQL)|
                                            +-------------+

Traffic Flow:
- Ingress NGINX -> Frontend (allowed)
- Frontend -> Backend (allowed)
- Backend -> Database (allowed)
- All other traffic (blocked)
```

## Quick Start

### Prerequisites

- Kubernetes cluster (Minikube, EKS, etc.)
- Helm v3.13.0+
- kubectl configured
- NGINX Ingress Controller

### Deploy to Development

```bash
# Deploy all components
helm install backend helm/Dev/backend -n backend-ns --create-namespace
helm install db helm/Dev/db -n db-ns --create-namespace
helm install frontend helm/Dev/frontend -n frontend-ns --create-namespace
```

### Deploy to Production

```bash
# Deploy all components
helm install backend helm/Prod/backend -n backend-ns --create-namespace
helm install db helm/Prod/db -n db-ns --create-namespace
helm install frontend helm/Prod/frontend -n frontend-ns --create-namespace
```

## CI/CD Requirements

### GitHub Secrets

Configure these secrets in your GitHub repository under the `ECR_REPOSITORY` environment:

| Secret | Description |
|--------|-------------|
| `AWS_ACCESS_KEY_ID` | AWS access key for ECR |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key for ECR |

### Pipeline Triggers

| Pipeline | Trigger | Branch |
|----------|---------|--------|
| CICD | Push | `dev` |
| Validate-Main | PR opened/ready | `main` |

## Scaling

All components support Horizontal Pod Autoscaling (HPA):

- **Min Replicas**: 1
- **Max Replicas**: 10
- **Target CPU**: 50%

## Resources

| Resource | Request | Limit |
|----------|---------|-------|
| CPU | 100m | 200m |
| Memory | 128Mi | 256Mi |

## Documentation

- [Deployment Guide](DEPLOYMENT.md) - Detailed deployment instructions
- [Helm Charts](helm/README.md) - Helm configuration details
- [Helm Instructions](helm/INSTRUCTIONS.md) - Step-by-step guide
- [K8s Manifests](k8s/README.md) - Raw manifest deployment

## License

This project is for educational and demonstration purposes.
