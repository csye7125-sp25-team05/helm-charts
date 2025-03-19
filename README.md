# helm-charts

This Helm chart deploys an API server and a PostgreSQL database in a Kubernetes cluster, each in separate namespaces (`api-server` and `api-db`), with Network Policies enforcing a "Default Deny" approach and restricted access between components.

## Overview

- **API Server**: Deployed in the `api-server` namespace, it connects to the PostgreSQL database for data operations.
- **PostgreSQL**: Deployed in the `api-db` namespace as a StatefulSet with persistent storage.
- **Network Policies**: Enforces a "Default Deny" policy for all ingress traffic, with an exception allowing the API server to communicate with PostgreSQL on port 5432.

## Prerequisites

- **Kubernetes Cluster**: Version 1.19 or higher.
- **Helm**: Version 3.x installed.
- **Network Plugin**: A CNI plugin that supports Network Policies (e.g., Calico, Cilium, Weave).
- **Storage Class**: A default or specified storage class for PostgreSQL persistent volumes.

## Chart Structure

```
helm-charts/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── namespace-api-db.yaml
│   ├── namespace-api-server.yaml
│   ├── db-deploy.yaml
│   ├── postgres-service.yaml
│   ├── api-server-deployment.yaml
│   ├── default-deny-api-db.yaml
│   ├── default-deny-api-server.yaml
│   ├── postgres-network-policy.yaml
│   └── secrets.yaml (optional)
├── README.md
```

## Installation

1. **Clone the Repository** (if applicable):
   ```bash
   git clone https://github.com/csye7125-sp25-team05/helm-charts
   cd helm-charts
   ```

2. **Install the Chart**:
   Deploy the chart with default values:
   ```bash
   helm install helm-charts .
   ```

3. **Customize Installation** (Optional):
   - Use a custom `values.yaml` file:
     ```bash
     helm install helm-charts . -f custom-values.yaml
     ```
   - Override specific values inline:
     ```bash
     helm install helm-charts . --set apiServer.replicas=3
     ```

4. **Verify Deployment**:
   Check the namespaces, deployments, and policies:
   ```bash
   kubectl get ns
   kubectl get statefulset -n api-db
   kubectl get deployment -n api-server
   kubectl get networkpolicy -n api-db
   kubectl get networkpolicy -n api-server
   ```

## Configuration

The `values.yaml` file contains configurable parameters. Key options include:

| Parameter                  | Description                                      | Default             |
|----------------------------|--------------------------------------------------|---------------------|
| `namespaces.apiDb`         | Namespace for PostgreSQL deployment             | `api-db`            |
| `namespaces.apiServer`     | Namespace for API server deployment             | `api-server`        |
| `postgres.replicas`        | Number of PostgreSQL replicas                   | `1`                 |
| `postgres.image`           | PostgreSQL container image                      | `postgres:latest`   |
| `postgres.storageSize`     | Size of persistent volume for PostgreSQL        | `10Gi`              |
| `apiServer.appName`        | Name of the API server application              | `api-server`        |
| `apiServer.replicas`       | Number of API server replicas                   | `2`                 |
| `apiServer.image`          | API server container image                      | `my-api-server:latest` |
| `apiServer.flywayImage`    | Flyway image for database migrations            | `flyway/flyway:latest` |
| `secrets.postgres.db`      | PostgreSQL database name                        | `api-server`        |
| `secrets.postgres.user`    | PostgreSQL username                             | `postgres`          |
| `secrets.postgres.password`| PostgreSQL password                             | `mysecretpassword`  |

### Example Custom `values.yaml`
```yaml
apiServer:
  replicas: 3
  image: my-custom-api:1.0.0
postgres:
  storageSize: 20Gi
secrets:
  postgres:
    password: supersecret
```

## Usage

- **Accessing the API Server**: After deployment, expose the API server with a Service or Ingress (not included in this chart) to access its endpoints (e.g., `/health`).
- **Database Connectivity**: The API server connects to PostgreSQL using the fully qualified service name: `postgres-service.api-db.svc.cluster.local:5432`.

## Network Policies

- **Default Deny**: All ingress traffic to pods in both namespaces is blocked unless explicitly allowed.
- **Postgres Access**: Only pods labeled `app: api-server` in the `api-server` namespace can connect to PostgreSQL in the `api-db` namespace on port 5432.

## Secrets Management

The chart includes an optional `secrets.yaml` template to deploy a `postgres-secrets` Secret in both namespaces. If you prefer external management:
1. Remove `templates/secrets.yaml` from the chart.
2. Create the Secret manually before deployment:
   ```bash
   kubectl create secret generic postgres-secrets \
     --namespace api-db \
     --from-literal=POSTGRES_DB=api-server \
     --from-literal=POSTGRES_USER=postgres \
     --from-literal=POSTGRES_PASSWORD=mysecretpassword
   kubectl create secret generic postgres-secrets \
     --namespace api-server \
     --from-literal=POSTGRES_DB=api-server \
     --from-literal=POSTGRES_USER=postgres \
     --from-literal=POSTGRES_PASSWORD=mysecretpassword
   ```

## Upgrading the Chart

To update the deployment:
```bash
helm upgrade helm-charts .
```

## Uninstalling

To remove the deployment:
```bash
helm uninstall helm-charts
```

*Note*: This does not delete the namespaces or persistent volumes unless explicitly cleaned up with `kubectl`.

## Troubleshooting

- **Pods Not Starting**: Check pod logs with `kubectl logs <pod-name> -n <namespace>`.
- **Network Policy Issues**: Verify policies with `kubectl describe networkpolicy -n <namespace>`.
- **Database Connectivity**: Ensure the API server can resolve `postgres-service.api-db.svc.cluster.local` (test with a temporary pod if needed).
