# Components Documentation

This section provides detailed documentation for each component in the LGTM Stack, including configuration, APIs, and operational details.

## 📋 Component Overview

| Component | Purpose | Port | Technology |
|-----------|---------|------|------------|
| **[OpenTelemetry Collector](otel-collector.md)** | Telemetry ingestion & routing | 4317/4318 | Go |
| **[Grafana](grafana.md)** | Visualization & dashboards | 3000 | Go/React |
| **[Prometheus](prometheus.md)** | Metrics collection & storage | 9090 | Go |
| **[Tempo](tempo.md)** | Distributed tracing | 3200 | Go |
| **[Loki](loki.md)** | Log aggregation | 3100 | Go |
| **[Pyroscope](pyroscope.md)** | Continuous profiling | 4040 | Go |

## 🔄 Component Interactions

```
┌─────────────────┐
│ Applications    │
│ (Instrumented)  │
└─────────┬───────┘
          │ OTLP
          ▼
┌─────────────────┐    ┌─────────────────┐
│ OTLP Collector  │───►│   Grafana       │
│                 │    │ (Visualization) │
└─────────┬───────┘    └─────────────────┘
          │
    ┌─────┼─────┐
    │     │     │
    ▼     ▼     ▼
┌─────┐ ┌─────┐ ┌─────┐
│Prom │ │Tempo│ │Loki │
│ethe │ │     │ │     │
│us   │ │     │ │     │
└─────┘ └─────┘ └─────┘
    │     │     │
    └─────┼─────┘
          │
          ▼
    ┌─────────────┐
    │  Pyroscope  │
    │ (Profiling) │
    └─────────────┘
```

## 🏗️ Component Architecture

### OpenTelemetry Collector
**Role**: Central telemetry ingestion and routing hub
- **Receivers**: OTLP (gRPC/HTTP), Prometheus scraping
- **Processors**: Batch processing, filtering
- **Exporters**: Routes data to appropriate backends
- **Extensions**: Health checks, service discovery

### Grafana
**Role**: Unified visualization platform
- **Data Sources**: Prometheus, Tempo, Loki, Pyroscope
- **Dashboards**: Pre-built and custom visualizations
- **Alerting**: Rule-based notifications
- **Plugins**: Extensible visualization capabilities

### Prometheus
**Role**: Time-series metrics database
- **Storage**: Efficient time-series storage
- **Query**: PromQL query language
- **Alerting**: Rule evaluation engine
- **Service Discovery**: Dynamic target discovery

### Tempo
**Role**: Distributed tracing backend
- **Ingestion**: OTLP trace data
- **Storage**: Trace storage and indexing
- **Search**: Full-text trace search
- **Analysis**: Service graphs, span metrics

### Loki
**Role**: Log aggregation system
- **Ingestion**: OTLP log data
- **Indexing**: Label-based indexing
- **Query**: LogQL query language
- **Integration**: Trace correlation

### Pyroscope
**Role**: Continuous profiling platform
- **Ingestion**: OTLP profiling data
- **Storage**: Profile storage and indexing
- **Visualization**: Flame graphs
- **Analysis**: Performance profiling

## ⚙️ Configuration Patterns

### Environment Variables
All components support configuration via environment variables:

```bash
# Component-specific settings
GRAFANA_PORT=3000
PROMETHEUS_RETENTION=15d
TEMPO_TRACE_IDLE_PERIOD=1s

# Cross-component settings
ENABLE_LOGS_ALL=true
DATA_PATH=/data
```

### Configuration Files
Components use YAML configuration files mounted as volumes:

```yaml
# Example: Prometheus configuration
global:
  scrape_interval: 15s
otlp:
  keep_identifying_resource_attributes: true
```

### Runtime Configuration
Limited runtime reconfiguration through APIs:

```bash
# Grafana datasource updates
curl -X POST http://localhost:3000/api/datasources \
  -H "Content-Type: application/json" \
  -d '{"name":"Prometheus","type":"prometheus","url":"http://localhost:9090"}'
```

## 🔍 Monitoring Components

### Health Endpoints
Each component provides health check endpoints:

| Component | Health Endpoint | Response |
|-----------|----------------|----------|
| OTLP Collector | `GET /ready` | `200 OK` |
| Grafana | `GET /api/health` | `{"database":"ok"}` |
| Prometheus | `GET /api/v1/status/runtimeinfo` | Status info |
| Tempo | `GET /ready` | `200 OK` |
| Loki | `GET /ready` | `200 OK` |
| Pyroscope | `GET /ready` | `200 OK` |

### Metrics Endpoints
Components expose their own metrics:

```bash
# Prometheus self-metrics
curl http://localhost:9090/metrics

# OTLP Collector metrics
curl http://localhost:8888/metrics
```

### Log Access
Component logs are available through Docker:

```bash
# View all component logs
docker logs lgtm

# Follow logs in real-time
docker logs -f lgtm
```

## 🔧 Operational Commands

### Startup Sequence
Components start in a specific order managed by `run-all.sh`:

1. **Grafana** - Visualization platform
2. **Loki** - Log aggregation
3. **OTLP Collector** - Telemetry ingestion
4. **Prometheus** - Metrics storage
5. **Tempo** - Trace storage
6. **Pyroscope** - Profiling storage

### Service Dependencies
```
OTLP Collector
├── Prometheus (metrics export)
├── Tempo (trace export)
├── Loki (log export)
└── Pyroscope (profile export)

Grafana
├── Prometheus (metrics queries)
├── Tempo (trace queries)
├── Loki (log queries)
└── Pyroscope (profile queries)
```

### Resource Requirements

| Component | Memory | CPU | Disk |
|-----------|--------|-----|------|
| OTLP Collector | 100MB | 0.1 | 10MB |
| Grafana | 200MB | 0.2 | 100MB |
| Prometheus | 500MB | 0.3 | 2GB+ |
| Tempo | 300MB | 0.2 | 1GB+ |
| Loki | 200MB | 0.2 | 1GB+ |
| Pyroscope | 150MB | 0.1 | 500MB+ |

## 🚨 Troubleshooting Components

### Common Issues

#### Component Won't Start
```bash
# Check component logs
docker logs lgtm | grep -A 10 -B 10 "component_name"

# Check port conflicts
netstat -tulpn | grep :port_number

# Check resource availability
docker stats lgtm
```

#### Data Not Appearing
```bash
# Verify data ingestion
curl http://localhost:4318/v1/traces -X POST -d '{}' -H "Content-Type: application/json"

# Check component health
curl http://localhost:component_port/ready

# Verify data storage
docker exec lgtm ls -la /data/component_name/
```

#### Performance Issues
```bash
# Monitor resource usage
docker stats lgtm

# Check component metrics
curl http://localhost:component_port/metrics

# Analyze query performance
# (Component-specific debugging)
```

## 🔄 Component Updates

### Version Management
Components are version-pinned in the Dockerfile:

```dockerfile
ARG GRAFANA_VERSION=v12.3.1
ARG PROMETHEUS_VERSION=v3.8.0
ARG TEMPO_VERSION=v2.9.0
ARG LOKI_VERSION=v3.6.3
ARG PYROSCOPE_VERSION=v1.16.0
ARG OPENTELEMETRY_COLLECTOR_VERSION=v0.139.0
```

### Update Process
1. **Version Updates**: Renovate bot updates versions
2. **Testing**: Automated acceptance tests
3. **Release**: Tagged releases with changelogs
4. **Container Build**: Automated Docker image builds

### Compatibility Matrix
- **OTLP Collector**: Compatible with OTLP 1.0+
- **Grafana**: Compatible with all backend versions
- **Prometheus**: Compatible with OTLP metrics
- **Tempo**: Compatible with OTLP traces
- **Loki**: Compatible with OTLP logs
- **Pyroscope**: Compatible with OTLP profiles

## 📚 Component Documentation Links

### Official Documentation
- [OpenTelemetry Collector](https://opentelemetry.io/docs/collector/)
- [Grafana](https://grafana.com/docs/grafana/latest/)
- [Prometheus](https://prometheus.io/docs/)
- [Tempo](https://grafana.com/docs/tempo/latest/)
- [Loki](https://grafana.com/docs/loki/latest/)
- [Pyroscope](https://grafana.com/docs/pyroscope/latest/)

### Configuration References
- [OTLP Collector Config](https://opentelemetry.io/docs/collector/configuration/)
- [Grafana Config](https://grafana.com/docs/grafana/latest/setup-grafana/configure-docker/)
- [Prometheus Config](https://prometheus.io/docs/prometheus/latest/configuration/)
- [Tempo Config](https://grafana.com/docs/tempo/latest/configuration/)
- [Loki Config](https://grafana.com/docs/loki/latest/configuration/)
- [Pyroscope Config](https://grafana.com/docs/pyroscope/latest/configure/)

---

For detailed information about each component, select from the list above. Each component page includes configuration options, API references, troubleshooting guides, and operational details. 🧩