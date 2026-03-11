# Phase 2: Kafka, JMX Exporter, Airflow & Zabbix

This phase adds a data platform stack and traditional host monitoring to the LGTM observability environment built in Phase 1. All components deploy into the existing `lgtm` namespace in Kubernetes.

## Components

| Manifest | Services Deployed | Metrics Port |
|----------|------------------|--------------|
| `zookeeper.yaml` | ZooKeeper | - |
| `kafka.yaml` | Kafka + JMX Exporter | 9092 (broker), 7071 (metrics) |
| `statsd-exporter.yaml` | StatsD Exporter | 9102 |
| `airflow.yaml` | Airflow standalone | 8080 (UI) |
| `zabbix.yaml` | Zabbix DB, Server, Web, Agent (DaemonSet) | 8080 (UI) |

## Prerequisites

Phase 1 must be running:
```powershell
kubectl get pods -n lgtm
# All pods should be Running before proceeding
```

## Deploy

```powershell
cd C:\mml-lab\mml-training\phase2

# Deploy ConfigMaps first
kubectl apply -f configmaps/

# Deploy services (order matters)
kubectl apply -f zookeeper.yaml
kubectl apply -f kafka.yaml
kubectl apply -f statsd-exporter.yaml
kubectl apply -f airflow.yaml
kubectl apply -f zabbix.yaml
```

Or all at once:
```powershell
kubectl apply -f configmaps/ && kubectl apply -f .
```

## Verify Pods

```powershell
kubectl get pods -n lgtm
```

Expected new pods (all `1/1 Running`):
```
airflow-xxx          1/1  Running
kafka-xxx            1/1  Running   # goes through Init:0/1 first (JMX jar download)
statsd-exporter-xxx  1/1  Running
zookeeper-xxx        1/1  Running
zabbix-agent-xxx     1/1  Running   # DaemonSet
zabbix-db-xxx        1/1  Running
zabbix-server-xxx    1/1  Running
zabbix-web-xxx       1/1  Running
```

> Kafka pod will show `Init:0/1` for ~30 seconds while the JMX Exporter JAR downloads. This is normal.

## Access the UIs (kind cluster)

> ⚠️ **kind does not route NodePort traffic to `localhost`.** Use `kubectl port-forward`.

```powershell
# Zabbix Web UI
Start-Job -ScriptBlock { kubectl port-forward -n lgtm deployment/zabbix-web 8080:8080 }

# Airflow UI (use 8081 to avoid conflict with Zabbix on 8080)
Start-Job -ScriptBlock { kubectl port-forward -n lgtm deployment/airflow 8081:8080 }
```

| Service | URL | Credentials |
|---------|-----|-------------|
| Zabbix UI | http://localhost:8080 | Admin / zabbix (capital A) |
| Airflow UI | http://localhost:8081 | admin / admin \* |

\* Airflow `standalone` takes 2–3 minutes to initialize on first start. Wait for `1/1 Running` then wait
for the ready signal before logging in:
```powershell
kubectl logs -n lgtm deployment/airflow --follow
# Wait for: standalone | Airflow is ready
```

If `admin`/`admin` is still rejected, look up the generated password:
```powershell
kubectl logs -n lgtm deployment/airflow | Select-String -Pattern "password"
# standalone | Login with username: admin  password: Ab3xK9mP
```

## Add Prometheus Scrape Jobs

The Phase 2 scrape jobs (kafka, airflow, zabbix-web) are already included in
`phase1/configmaps/prometheus-config.yaml`. Apply and restart Prometheus to activate them:

```powershell
kubectl apply -f ../phase1/configmaps/prometheus-config.yaml
kubectl rollout restart deployment/prometheus -n lgtm
```

## Verify Metrics

Run these queries in Prometheus (http://localhost:9090) to confirm each component is working.
All queries below should return data immediately after deploy — no DAG runs or manual steps needed.

### Scrape targets up

Check the Prometheus targets page first: **Status → Targets**
- `kafka` → UP
- `airflow` → UP

### Kafka — confirms JMX Exporter is running inside the Kafka JVM

```promql
jvm_heap_memory_used_bytes{job="kafka"}
```

Expected: a number in the hundreds of millions (bytes). If this returns no data, the JMX agent
did not attach — check `kubectl logs -n lgtm deployment/kafka`.

### Airflow — confirms StatsD bridge is receiving scheduler heartbeats

```promql
airflow_scheduler_heartbeat{job="airflow"}
```

Expected: a counter that increments every few seconds. This is always emitted by the scheduler
regardless of whether any DAGs are running.

### Zabbix — confirm agent is running (pod-level check only)

```promql
kube_pod_status_ready{namespace="lgtm", pod=~"zabbix.*"}
```

> Note: Zabbix does not expose Prometheus metrics natively. The above confirms pods are healthy
> via kube-state-metrics. The `zabbix-web` scrape job will show DOWN in Prometheus targets — this
> is expected and can be ignored.

## Directory Structure

```
phase2/
+-- README.md
+-- zookeeper.yaml
+-- kafka.yaml
+-- statsd-exporter.yaml
+-- airflow.yaml
+-- zabbix.yaml
+-- configmaps/
    +-- kafka-jmx-config.yaml
    +-- statsd-mapping-config.yaml
    +-- prometheus-scrape-jobs.yaml
```

## Teardown

```powershell
kubectl delete -f .
kubectl delete -f configmaps/
```

> This does NOT affect Phase 1 — components share the `lgtm` namespace but are managed by separate manifests.

## Next Steps

Once all pods are `1/1 Running`, proceed to [Phase 3](../phase3/README.md) to deploy AWS services via LocalStack.
