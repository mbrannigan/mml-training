# Phase 1: Core LGTM Stack

Foundational observability stack with Grafana, Prometheus, Mimir, Loki, and Promtail running in Docker Compose.

## What's Included

- **Grafana** - Visualization and dashboarding (port 3000)
- **Prometheus** - Metrics scraping (port 9090)
- **Mimir** - Long-term metrics storage (port 9009)
- **Loki** - Log aggregation (port 3100)
- **Promtail** - Log collection from Docker containers

## Quick Start

### 1. Enable Docker API (Windows Only)

**Important:** Promtail needs access to the Docker API to collect container logs.

1. Open **Docker Desktop**
2. Go to **Settings** → **General**
3. Enable: **"Expose daemon on tcp://localhost:2375 without TLS"**
4. Click **Apply & Restart**

⚠️ This is safe for local development only.

### 2. Start the Stack

```bash
cd phase1
docker-compose up -d
```

### 3. Verify Containers are Running

```bash
docker-compose ps
```

You should see all 5 services running.

### 4. Access the UIs

- **Grafana**: http://localhost:3000 (admin/admin)
- **Prometheus**: http://localhost:9090
- **Loki**: http://localhost:3100/ready

### 5. Configure Grafana Data Sources

In Grafana, go to **Connections** → **Add new connection**

**Add Prometheus:**
- URL: `http://prometheus:9090`
- Click **Save & Test**

**Add Mimir:**
- Search for **Prometheus**
- Name: `Mimir`
- URL: `http://mimir:9009/prometheus`
- Under **Custom HTTP Headers**:
  - Header: `X-Scope-OrgID`
  - Value: `anonymous`
- Click **Save & Test**

**Add Loki:**
- URL: `http://loki:3100`
- Click **Save & Test**

### 6. Test Queries

**In Prometheus/Mimir (switch to Code mode):**
```promql
up
```
Should show all services with `up=1`

**In Loki:**
```logql
{job="docker"}
```
Should show logs from all containers

## Configuration Files

- `docker-compose.yml` - Service definitions
- `prometheus.yaml` - Prometheus scrape config with remote_write to Mimir
- `mimir.yaml` - Mimir storage configuration
- `loki.yaml` - Loki log storage configuration
- `promtail.yaml` - Log collection from Docker containers

## Key Features

✅ Prometheus → Mimir remote_write (with X-Scope-OrgID header)
✅ Docker container log collection via Promtail
✅ All services monitored (Prometheus, Grafana, Loki, Mimir)
✅ 1-hour metrics retention in Prometheus, unlimited in Mimir
✅ 7-day log retention in Loki

## Troubleshooting

**Logs not appearing in Loki?**
- Verify Docker API is enabled (Step 1)
- Check Promtail logs: `docker-compose logs promtail`

**Mimir authentication errors?**
- The `X-Scope-OrgID: anonymous` header is required
- Already configured in `prometheus.yaml` remote_write section
- Also needed when querying Mimir from Grafana

**Grafana can't connect to data sources?**
- Verify all containers are on the `lgtm` network
- Use service names (`prometheus`, `loki`, `mimir`) not `localhost`

## Clean Up

```bash
docker-compose down -v
```

The `-v` flag removes volumes (all data will be lost).

---

**Next:** Phase 2 adds Kafka, Airflow, and JMX monitoring
