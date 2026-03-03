# MML Training - Self-Hosted Observability Stack

A comprehensive hands-on lab environment for learning modern observability with the LGTM stack (Loki, Grafana, Tempo, Mimir) and related technologies.

## 🎯 Overview

This repository contains all the code, configuration files, and scripts needed to build a production-grade observability stack from scratch. Through progressive phases, you'll learn to deploy, configure, and operate each component while monitoring real applications.

## 🏗️ Architecture

The training follows a phased approach, building from core components to a complete observability platform:

- **Grafana** - Visualization and dashboarding
- **Prometheus** - Metrics collection and short-term storage
- **Mimir** - Long-term metrics storage with unlimited retention
- **Loki** - Log aggregation and querying
- **Tempo** - Distributed tracing
- **Promtail** - Log shipping agent

## 📚 Training Phases

### Phase 1: Core LGTM Stack
**Status:** ✅ Complete

Set up the foundational observability stack with Docker Compose:
- Grafana for visualization
- Prometheus for metrics scraping
- Mimir for long-term metrics storage
- Loki for log aggregation
- Promtail for log collection

**What you'll learn:**
- Docker Compose orchestration
- Prometheus scrape configuration
- Remote write to Mimir
- Log collection from Docker containers
- PromQL and LogQL basics

📂 **Files:** [`/phase1/`](./phase1/)

---

### Phase 2: Extended Monitoring
**Status:** 🔄 In Progress

Add monitoring for complex systems:
- Kafka (metrics via JMX Exporter)
- Apache Airflow
- PostgreSQL
- Redis

**What you'll learn:**
- JMX metrics export
- Application-specific exporters
- Custom Prometheus scrape configs
- Multi-service monitoring

📂 **Files:** [`/phase2/`](./phase2/)

---

### Phase 3: Kubernetes Integration
**Status:** 📋 Planned

Deploy the stack on Kubernetes:
- Helm charts for LGTM components
- ServiceMonitors and PodMonitors
- Node exporter for cluster metrics
- cAdvisor for container metrics

**What you'll learn:**
- Kubernetes monitoring patterns
- Helm chart configuration
- Service discovery in K8s
- Resource monitoring

📂 **Files:** [`/phase3/`](./phase3/)

---

### Phase 4: Distributed Tracing
**Status:** 📋 Planned

Add Tempo for distributed tracing:
- Tempo deployment
- OpenTelemetry instrumentation
- Trace correlation with logs and metrics
- End-to-end request tracing

**What you'll learn:**
- Distributed tracing concepts
- OpenTelemetry integration
- TraceQL query language
- Correlating traces with metrics and logs

📂 **Files:** [`/phase4/`](./phase4/)

---

## 🚀 Quick Start

### Prerequisites

- **Windows 10/11** with WSL2 (or Linux/macOS)
- **Docker Desktop** (4.x or later)
- **8GB+ RAM** allocated to Docker
- **Git** for cloning the repository

### Getting Started

1. **Clone the repository:**
```bash
git clone https://github.com/mbrannigan/mml-training.git
cd mml-training
```

2. **Start with Phase 1:**
```bash
cd phase1
docker-compose up -d
```

3. **Access Grafana:**
- URL: http://localhost:3000
- Username: `admin`
- Password: `admin`

4. **Follow the documentation:**
See the [Phase 1 Setup Guide](./docs/phase1-setup.md) for detailed instructions.

---

## 📖 Documentation

All training materials and guides are available in the [`/docs/`](./docs/) directory:

- [Phase 1: Core LGTM Stack Setup](./docs/phase1-setup.md)
- [Understanding Your LGTM Stack](./docs/understanding-lgtm.md)
- [Component Deep Dives](./docs/component-deep-dives/)
  - [Prometheus Deep Dive](./docs/component-deep-dives/prometheus.md)
  - [Mimir Deep Dive](./docs/component-deep-dives/mimir.md)
  - [Loki Deep Dive](./docs/component-deep-dives/loki.md)
  - [Grafana Deep Dive](./docs/component-deep-dives/grafana.md)
- [Query Language Guides](./docs/query-languages/)
  - [PromQL Mastery](./docs/query-languages/promql.md)
  - [LogQL Mastery](./docs/query-languages/logql.md)
  - [TraceQL Mastery](./docs/query-languages/traceql.md)
- [Useful Resources](./docs/resources.md)

---

## 🗂️ Repository Structure

```
mml-training/
├── README.md
├── docs/                          # All training documentation
│   ├── phase1-setup.md
│   ├── understanding-lgtm.md
│   ├── component-deep-dives/
│   └── query-languages/
├── phase1/                        # Phase 1: Core LGTM Stack
│   ├── docker-compose.yml
│   ├── prometheus.yaml
│   ├── mimir.yaml
│   ├── loki.yaml
│   ├── promtail.yaml
│   └── grafana-provisioning/
├── phase2/                        # Phase 2: Extended Monitoring
│   ├── docker-compose.yml
│   ├── kafka/
│   ├── airflow/
│   └── exporters/
├── phase3/                        # Phase 3: Kubernetes
│   ├── helm/
│   ├── manifests/
│   └── monitoring/
├── phase4/                        # Phase 4: Tracing
│   ├── tempo.yaml
│   ├── otel-collector/
│   └── sample-apps/
└── scripts/                       # Utility scripts
    ├── setup.sh
    ├── cleanup.sh
    └── validate.sh
```

---

## 🎓 Learning Path

1. **Start with fundamentals** - Complete Phase 1 to understand the core stack
2. **Explore query languages** - Master PromQL and LogQL through the guides
3. **Dive deep** - Read component deep dives for architectural understanding
4. **Expand capabilities** - Progress through Phases 2-4
5. **Experiment** - Modify configs, create dashboards, build alerts

---

## 🛠️ Troubleshooting

### Common Issues

**Docker containers won't start:**
- Check port conflicts: `netstat -ano | findstr "3000 9090"`
- Increase Docker memory allocation (Settings → Resources)
- Verify config file syntax

**Grafana can't connect to data sources:**
- Ensure all containers are on the same network: `docker network inspect phase1_lgtm`
- Check container names match config: `docker ps`
- Review logs: `docker-compose logs <service-name>`

**Logs not appearing in Loki:**
- Enable Docker API (Windows): Settings → General → "Expose daemon on tcp://localhost:2375"
- Check Promtail logs: `docker-compose logs promtail`
- Verify Promtail can reach Docker API

More troubleshooting: [Troubleshooting Guide](./docs/troubleshooting.md)

---

## 🤝 Contributing

This is a training repository, but contributions are welcome!

- **Found a bug?** Open an issue
- **Have an improvement?** Submit a pull request
- **Want to add a phase?** Let's discuss in issues first

---

## 📝 License

MIT License - See [LICENSE](./LICENSE) for details

---

## 🔗 Additional Resources

- [Grafana Documentation](https://grafana.com/docs/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Mimir Documentation](https://grafana.com/docs/mimir/)
- [Grafana Loki Documentation](https://grafana.com/docs/loki/)
- [Grafana Tempo Documentation](https://grafana.com/docs/tempo/)

---

## ⭐ Acknowledgments

Built for hands-on learning and practical experience with production-grade observability tools.

---

**Happy Learning! 🚀**
