# Airflow + StatsD Exporter

## Overview

Airflow emits metrics via StatsD (UDP). The StatsD Exporter bridges to Prometheus.

```
Airflow --> StatsD UDP :9125 --> statsd-exporter --> Prometheus :9102
```

## Ports

| Port | Purpose |
|------|---------|
| 8080 | Airflow Web UI |
| 9125/udp | StatsD input |
| 9102 | Prometheus /metrics output |

## Credentials

- URL: http://localhost:8080
- Username: `admin` / Password: `admin`

## Verify

```powershell
Invoke-WebRequest -Uri "http://localhost:9102/metrics" -UseBasicParsing | Select-Object -ExpandProperty Content
```

## Key Metrics

| Metric | Description |
|--------|-------------|
| `airflow_scheduler_tasks_running` | Tasks actively running |
| `airflow_scheduler_tasks_starving` | Tasks waiting for a slot |
| `airflow_dagrun_duration_success` | Successful DAG run duration |
| `airflow_dagrun_duration_failed` | Failed DAG run duration |
