# Kubernetes Manifests for LGTM Stack

This directory contains Kubernetes manifests to deploy the LGTM (Loki, Grafana, Tempo, Mimir) observability stack on Kubernetes.

## Overview

The Kubernetes deployment mirrors the Docker Compose setup but adapts it for native Kubernetes operation:

- **Namespace**: All resources are deployed in the `lgtm` namespace
- **ConfigMaps**: Configuration files are stored as ConfigMaps
- **Services**: Each component has a Kubernetes Service for service discovery
- **Deployments**: Grafana, Prometheus, Mimir, and Loki run as Deployments
- **DaemonSet**: Promtail runs as a DaemonSet (one pod per node) to collect logs

## Differences from Docker Compose

### Service Discovery
- **Docker Compose**: Uses service names (e.g., `http://prometheus:9090`)
- **Kubernetes**: Uses DNS within namespace (e.g., `http://prometheus:9090` or `http://prometheus.lgtm.svc.cluster.local:9090`)

### Promtail Log Collection
- **Docker Compose**: Connects to Docker API at `tcp://host.docker.internal:2375`
- **Kubernetes**: Runs as DaemonSet, mounts node's `/var/log` and `/var/lib/docker/containers`

### Access to Services (kind cluster)

> ⚠️ **kind does not route NodePort traffic to `localhost`.** Use `kubectl port-forward` to access UIs.

```powershell
# Run each in a separate PowerShell background job
Start-Job -ScriptBlock { kubectl port-forward -n lgtm deployment/grafana 3000:3000 }
Start-Job -ScriptBlock { kubectl port-forward -n lgtm deployment/prometheus 9090:9090 }
```

| Service | URL | Credentials |
|---------|-----|--------------|
| Grafana | http://localhost:3000 | admin / admin |
| Prometheus | http://localhost:9090 | — |

Loki and Mimir are ClusterIP only (internal access, no UI).

### RBAC Permissions
Prometheus and Promtail require ServiceAccounts with ClusterRole permissions to:
- Discover Kubernetes nodes, pods, services, and endpoints
- Access metrics endpoints

## Deployment Instructions

### 1. Create the namespace and ConfigMaps
```powershell
kubectl apply -f namespace.yaml
kubectl apply -f configmaps/
```

### 2. Deploy the services
```powershell
kubectl apply -f grafana.yaml
kubectl apply -f prometheus.yaml
kubectl apply -f mimir.yaml
kubectl apply -f loki.yaml
kubectl apply -f promtail.yaml
```

Or deploy everything at once:
```powershell
kubectl apply -f namespace.yaml
kubectl apply -f configmaps/
kubectl apply -f .
```

### 3. Verify deployment
```powershell
# Check all pods are running
kubectl get pods -n lgtm

# Check services
kubectl get svc -n lgtm

# Check logs if any pod has issues
kubectl logs -n lgtm <pod-name>
```

### 4. Access the UIs

See port-forward commands above. Once forwarding is active:
- **Grafana**: http://localhost:3000 (username: `admin`, password: `admin`)
- **Prometheus**: http://localhost:9090

## Configuration Notes

### Prometheus
- Scrapes itself, Grafana, Loki, and Mimir
- Auto-discovers Kubernetes pods with `prometheus.io/scrape: "true"` annotation
- Sends metrics to Mimir for long-term storage with `X-Scope-OrgID: anonymous` header
- 1-hour local retention

### Mimir
- Single-tenant mode using `X-Scope-OrgID: anonymous`
- Filesystem storage (using emptyDir - data lost on pod restart)
- For production, replace emptyDir with PersistentVolumeClaim

### Loki
- 7-day log retention
- Filesystem storage (using emptyDir - data lost on pod restart)
- For production, replace emptyDir with PersistentVolumeClaim

### Promtail
- Runs as DaemonSet (one pod per node)
- Collects logs from all Kubernetes pods
- Automatically adds labels: namespace, pod, container

## Storage Considerations

**Current setup uses `emptyDir` volumes** which means:
- ✅ Fast and simple for testing
- ❌ Data is lost when pod restarts
- ❌ Not suitable for production

**For production**, replace `emptyDir` with `PersistentVolumeClaim`:

```yaml
volumes:
  - name: prometheus-storage
    persistentVolumeClaim:
      claimName: prometheus-pvc
```

## Cleanup

To remove the entire stack:
```powershell
kubectl delete namespace lgtm
```

This will delete all resources in the lgtm namespace.

## Next Steps

- Configure Grafana data sources
- Deploy Phase 2 (Kafka, Airflow, Zabbix) from `../phase2/`
- Add persistent volumes for production use
- Configure ingress for external access
- Set up alerts and dashboards
