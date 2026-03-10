# Zabbix

## Overview

Zabbix provides traditional agent-based host monitoring. The Zabbix Prometheus Exporter bridges metrics into Prometheus format.

```
zabbix-agent --> zabbix-server --> zabbix-exporter --> Prometheus :9224
```

## Services

| Container | Purpose |
|-----------|---------|
| zabbix-db | PostgreSQL backend |
| zabbix-server | Core Zabbix server |
| zabbix-web | Web UI (Nginx) |
| zabbix-agent | Monitors the lab host |
| zabbix-exporter | Exposes metrics to Prometheus |

## Ports

| Port | Purpose |
|------|---------|
| 8090 | Zabbix Web UI |
| 10051 | Zabbix server |
| 9224 | Prometheus /metrics output |

## Credentials

- URL: http://localhost:8090
- Username: `Admin` / Password: `zabbix`

> Allow 2-3 minutes on first start for the database to initialize.

## Verify

```powershell
Invoke-WebRequest -Uri "http://localhost:9224/metrics" -UseBasicParsing | Select-Object -ExpandProperty Content
```
