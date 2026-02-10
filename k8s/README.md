# Raw Kubernetes Manifests

This directory contains the raw Kubernetes YAML manifests for the application. These are useful for understanding the underlying resources created by Helm or for manual deployment without Helm.

## 📂 Structure

- **`frontend/`**: Deployments, Services, and Ingress for the Frontend.
- **`backend/`**: Deployments, Services, and ConfigMaps for the Backend.
- **`db/`**: StatefulSets and Services for Redis and PostgreSQL.
- **`network-policies/`**: NetworkPolicy definitions for namespace isolation.

## ⚠️ Note on Usage

These files are **independent** of the Helm chart in the `../helm` directory. Changes made here will **not** be reflected in `helm install` unless you also update the Helm templates.

## 🚀 Manual Deployment

If you prefer `kubectl apply` over Helm:

1. **Create Namespaces**:
   ```bash
   kubectl create ns frontend-ns
   kubectl create ns backend-ns
   kubectl create ns db-ns
   ```

2. **Apply Database Resources**:
   ```bash
   kubectl apply -f k8s/db/
   ```

3. **Apply Backend Resources**:
   ```bash
   kubectl apply -f k8s/backend/
   ```

4. **Apply Frontend Resources**:
   ```bash
   kubectl apply -f k8s/frontend/
   ```

5. **Apply Network Policies**:
   ```bash
   kubectl apply -f k8s/network-policies/
   ```
