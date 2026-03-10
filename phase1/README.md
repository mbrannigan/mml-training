# Phase 1: Core LGTM Stack

Deploys Grafana, Prometheus, Mimir, Loki, Promtail, and Node Exporter to a kind Kubernetes cluster in the `lgtm` namespace.

---

## Prerequisites

- Docker Desktop running with kind cluster active
- `kubectl` configured and pointing at the kind cluster
- At least **8GB RAM** allocated to Docker Desktop

Verify your cluster is ready:
```powershell
kubectl get nodes
# Should show 1 control-plane + 3 workers, all Ready
```

---

## Deploy

```powershell
cd C:\mml-lab\mml-training\phase1

kubectl apply -f namespace.yaml
kubectl apply -f configmaps/
kubectl apply -f .
```

### Verify

```powershell
kubectl get pods -n lgtm -w
```

Expected pods (all `1/1 Running`):
```
grafana-xxx           1/1  Running
prometheus-xxx        1/1  Running
mimir-xxx             1/1  Running
loki-xxx              1/1  Running
promtail-xxx          1/1  Running   # DaemonSet - one per node
node-exporter-xxx     1/1  Running   # DaemonSet - one per node
```

---

## Access the UIs

> ⚠️ **kind does not route NodePort traffic to localhost.** Use `kubectl port-forward`.

```powershell
Start-Job -ScriptBlock { kubectl port-forward -n lgtm deployment/grafana 3000:3000 }
Start-Job -ScriptBlock { kubectl port-forward -n lgtm deployment/prometheus 9090:9090 }
```

| Service | URL | Credentials |
|---------|-----|--------------|
| Grafana | http://localhost:3000 | admin / admin |
| Prometheus | http://localhost:9090 | — |

Loki and Mimir are ClusterIP only (no UI).

**Stop port forwards when done:**
```powershell
Get-Job | Stop-Job; Get-Job | Remove-Job
```

---

## Configure Grafana Data Sources

In Grafana → Connections → Add new connection:

**Prometheus:**
- Type: `Prometheus`
- URL: `http://prometheus:9090`
- Save & Test

**Mimir:**
- Type: `Prometheus`
- Name: `Mimir`
- URL: `http://mimir:9009/prometheus`
- Custom HTTP Headers: `X-Scope-OrgID` = `anonymous`
- Save & Test

**Loki:**
- Type: `Loki`
- URL: `http://loki:3100`
- Save & Test

---

## Verify Metrics & Logs

In Grafana → Explore → Prometheus (Code mode):

```promql
# All scrape targets up
up

# Pod CPU usage
sum by (pod) (rate(container_cpu_usage_seconds_total{namespace="lgtm"}[5m]))

# Node CPU utilization
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

In Grafana → Explore → Loki:

```logql
# All lgtm namespace logs
{namespace="lgtm"}

# Errors only
{namespace="lgtm"} |= "error"
```

---

## Troubleshooting

**Pod stuck in Pending or CrashLoopBackOff:**
```powershell
kubectl describe pod -n lgtm <pod-name>
kubectl logs -n lgtm <pod-name> --previous
```

**No logs in Loki:** Check Promtail is tailing log files:
```powershell
kubectl logs -n lgtm -l app=promtail --tail=50
# Look for: msg="tail routine: started"
```
See [DEBUG-LOKI.md](./DEBUG-LOKI.md) for detailed Loki troubleshooting.

**Grafana data source "Bad Gateway":** Confirm service names match — use short names (`http://prometheus:9090`), not FQDN. Grafana resolves service names within the `lgtm` namespace automatically.

**Port-forward drops:** Normal when pods restart. Re-run the `Start-Job` commands.

---

## Configuration Notes

- **Prometheus:** Scrapes all services in `lgtm` namespace, remote-writes to Mimir, 1-hour local retention
- **Mimir:** Single-tenant mode (`X-Scope-OrgID: anonymous`), filesystem storage via `emptyDir`
- **Loki:** 7-day log retention, filesystem storage via `emptyDir`
- **Promtail:** DaemonSet, auto-collects logs from all pods, adds `namespace`/`pod`/`container` labels
- **Node Exporter:** DaemonSet, exposes host-level CPU/memory/disk/network metrics

> ⚠️ All storage uses `emptyDir` — data is lost on pod restart. Fine for lab use.

---

## Cleanup

```powershell
kubectl delete namespace lgtm
```

Deletes all Phase 1 and Phase 2 resources (they share the `lgtm` namespace).

---

## Next Steps

Once all pods are `1/1 Running`, proceed to [Phase 2](../phase2/README.md) to deploy Kafka, Airflow, and Zabbix.
