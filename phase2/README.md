# Phase 2: Kafka, JMX Exporter, Airflow & Zabbix

This phase adds a data platform stack and traditional host monitoring to the LGTM observability environment built in Phase 1.

## What's Deployed

| Component | Purpose | Metrics Port |
|-----------|---------|-------------|
| Kafka | Message broker | 7071 (JMX Exporter) |
| ZooKeeper | Kafka coordination | - |
| Airflow | Workflow orchestration | 9102 (StatsD Exporter) |
| Zabbix | Host monitoring | 9224 (Zabbix Exporter) |

## Prerequisites

- Phase 1 LGTM stack running (`kubectl get pods -n lgtm` - all Running)
- Docker Desktop with **12GB RAM** minimum
- Docker Compose

## Quick Start

```powershell
cd C:\mml-lab\mml-training\phase2

# Download JMX Exporter JAR first (not in git)
cd kafka
Invoke-WebRequest -Uri "https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.20.0/jmx_prometheus_javaagent-0.20.0.jar" -OutFile "jmx_prometheus_javaagent-0.20.0.jar"
cd ..

# Start everything
docker-compose up -d
```

## Service URLs

| Service | URL | Credentials |
|---------|-----|-------------|
| Airflow UI | http://localhost:8080 | admin / admin |
| Zabbix UI | http://localhost:8090 | Admin / zabbix |
| Kafka JMX metrics | http://localhost:7071/metrics | - |
| StatsD metrics | http://localhost:9102/metrics | - |
| Zabbix metrics | http://localhost:9224/metrics | - |

## Add Prometheus Scrape Jobs

See `prometheus-phase2.yml` for the scrape job snippets to add to your Phase 1 Prometheus config.

## Directory Structure

```
phase2/
+-- docker-compose.yml          # All services
+-- prometheus-phase2.yml       # Scrape jobs to add to Phase 1
+-- kafka/
|   +-- jmx-exporter-config.yml # JMX Exporter metric rules
|   +-- README.md
+-- airflow/
|   +-- statsd-mapping.yml      # StatsD metric name mappings
|   +-- README.md
+-- zabbix/
    +-- README.md
```

## Clean Up

```powershell
docker-compose down        # Stop services
docker-compose down -v     # Stop + wipe volumes
```
