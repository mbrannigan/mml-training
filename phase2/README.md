# Phase 2: Kafka, JMX Exporter, Airflow & Zabbix

This phase adds a data platform stack and traditional host monitoring to the LGTM observability environment built in Phase 1. All components deploy into the existing `lgtm` namespace in Kubernetes.

## Components

| Manifest | Services Deployed | Metrics Port |
|----------|------------------|--------------|
| `zookeeper.yaml` | ZooKeeper | - |
| `kafka.yaml` | Kafka + JMX Exporter (init container) | 7071 |
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

Expected new pods:
```
airflow-xxx          1/1  Running
kafka-xxx            1/1  Running
statsd-exporter-xxx  1/1  Running
zookeeper-xxx        1/1  Running
zabbix-agent-xxx     1/1  Running   # DaemonSet
zabbix-db-xxx        1/1  Running
zabbix-server-xxx    1/1  Running
zabbix-web-xxx       1/1  Running
```

## Access the UIs (kind cluster)

> ⚠️ **kind does not route NodePort traffic to `localhost`.** Use `kubectl port-forward` to access UIs.

```powershell
# Zabbix Web UI
Start-Job -ScriptBlock { kubectl port-forward -n lgtm deployment/zabbix-web 8080:8080 }

# Airflow UI (use 8081 to avoid conflict with Zabbix on 8080)
Start-Job -ScriptBlock { kubectl port-forward -n lgtm deployment/airflow 8081:8080 }
```

| Service | URL | Credentials |
|---------|-----|-------------|
| Zabbix UI | http://localhost:8080 | Admin / zabbix |
| Airflow UI | http://localhost:8081 | admin / admin |

> 💡 Note the capital "A" in Zabbix's `Admin` — it's case-sensitive.

## Add Prometheus Scrape Jobs

See `configmaps/prometheus-scrape-jobs.yaml` for the scrape job definitions. Add them to the Phase 1 Prometheus ConfigMap:

```powershell
kubectl edit configmap prometheus-config -n lgtm
# Add the three scrape_configs jobs from configmaps/prometheus-scrape-jobs.yaml

kubectl rollout restart deployment/prometheus -n lgtm
```

## Verify Metrics

```promql
# Kafka broker up
up{job="kafka"}

# Airflow scheduler
up{job="airflow"}

# Airflow tasks running
airflow_scheduler_tasks_running
```

## Directory Structure

```
phase2/
+-- README.md
+-- namespace.yaml
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
