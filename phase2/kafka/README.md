# Kafka + JMX Exporter

## Overview

Kafka runs as a single-broker instance with ZooKeeper. The JMX Exporter runs as a Java agent inside the Kafka JVM, exposing broker metrics on port `7071`.

## Ports

| Port | Purpose |
|------|---------|
| 9092 | Kafka broker |
| 7071 | JMX Exporter /metrics |
| 9999 | Raw JMX (internal) |

## Download JMX Exporter JAR

The JAR is not committed to git. Download before starting:

```powershell
cd phase2/kafka
Invoke-WebRequest -Uri "https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.20.0/jmx_prometheus_javaagent-0.20.0.jar" -OutFile "jmx_prometheus_javaagent-0.20.0.jar"
```

## Verify

```powershell
Invoke-WebRequest -Uri "http://localhost:7071/metrics" -UseBasicParsing | Select-Object -ExpandProperty Content
```

## Key Metrics

| Metric | Description |
|--------|-------------|
| `kafka_server_brokertopicmetrics_messagesin_total` | Messages in per second |
| `kafka_server_replicamanager_underreplicatedpartitions` | Should be 0 |
| `kafka_consumer_lag` | Consumer lag per partition |
| `jvm_heap_memory_used_bytes` | JVM heap usage |
