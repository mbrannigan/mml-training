# Phase 2: Kubernetes Manifests

All Phase 2 components deploy into the existing `lgtm` namespace alongside the Phase 1 LGTM stack.

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
cd C:\mml-lab\mml-training\phase2\k8s

# Deploy ConfigMaps first
kubectl apply -f configmaps/

# Deploy services (order matters - zookeeper before kafka, db before zabbix-server)
kubectl apply -f zookeeper.yaml
kubectl apply -f kafka.yaml
kubectl apply -f statsd-exporter.yaml
kubectl apply -f airflow.yaml
kubectl apply -f zabbix.yaml
```

Or deploy everything at once:
```powershell
kubectl apply -f configmaps/
kubectl apply -f .
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
zabbix-agent-xxx     1/1  Running   # DaemonSet - one per node
zabbix-db-xxx        1/1  Running
zabbix-server-xxx    1/1  Running
zabbix-web-xxx       1/1  Running
```

## Service URLs (NodePort)

| Service | URL |
|---------|-----|
| Airflow UI | http://localhost:30808 |
| Zabbix UI | http://localhost:30809 |

## Add Prometheus Scrape Jobs

See `configmaps/prometheus-scrape-jobs.yaml` for the scrape job definitions.

Add the three new jobs to the existing Phase 1 Prometheus ConfigMap:
```powershell
# Edit the Phase 1 prometheus config and add the phase2 jobs
kubectl edit configmap prometheus-config -n lgtm
```

Then restart Prometheus to pick up the changes:
```powershell
kubectl rollout restart deployment/prometheus -n lgtm
```

## Verify Metrics

In Grafana Explore (Prometheus data source, Code mode):

```
# Kafka broker up
up{job="kafka"}

# Kafka messages in
rate(kafka_server_brokertopicmetrics_messagesin_total[5m])

# Airflow scheduler running
up{job="airflow"}

# Airflow tasks running
airflow_scheduler_tasks_running
```

## Teardown

```powershell
kubectl delete -f .
kubectl delete -f configmaps/
```

> Note: This does NOT affect Phase 1 components - they share the `lgtm` namespace but are managed by separate manifests.
