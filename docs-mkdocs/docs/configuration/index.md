# Configuration Guide

This guide covers all configuration options for customizing the LGTM Stack to fit your specific needs and environment.

## 📋 Configuration Overview

The LGTM Stack supports multiple configuration methods:

### 1. Environment Variables
- **Scope**: Runtime configuration
- **Priority**: Highest (overrides config files)
- **Use Case**: Dynamic configuration, secrets

### 2. Configuration Files
- **Scope**: Component-specific settings
- **Priority**: Medium
- **Use Case**: Complex configurations, defaults

### 3. Volume Mounts
- **Scope**: External configuration files
- **Priority**: Low
- **Use Case**: Persistent custom configurations

### 4. Build-time Configuration
- **Scope**: Container image customization
- **Priority**: Lowest
- **Use Case**: Custom builds, fixed configurations

## 🔧 Environment Variables

### Core Stack Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `GF_SECURITY_ADMIN_USER` | `admin` | Grafana admin username |
| `GF_SECURITY_ADMIN_PASSWORD` | `admin` | Grafana admin password |
| `GF_PLUGINS_PREINSTALL` | - | Comma-separated list of Grafana plugins |
| `ENABLE_LOGS_ALL` | `false` | Enable verbose logging for all components |
| `ENABLE_LOGS_GRAFANA` | `false` | Enable Grafana logging |
| `ENABLE_LOGS_LOKI` | `false` | Enable Loki logging |
| `ENABLE_LOGS_PROMETHEUS` | `false` | Enable Prometheus logging |
| `ENABLE_LOGS_TEMPO` | `false` | Enable Tempo logging |
| `ENABLE_LOGS_PYROSCOPE` | `false` | Enable Pyroscope logging |
| `ENABLE_LOGS_OTELCOL` | `false` | Enable OTLP Collector logging |

### External Data Export

| Variable | Default | Description |
|----------|---------|-------------|
| `OTEL_EXPORTER_OTLP_ENDPOINT` | - | External OTLP endpoint URL |
| `OTEL_EXPORTER_OTLP_HEADERS` | - | Headers for external OTLP export |

### Port Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `GRAFANA_PORT` | `3000` | Grafana web UI port |
| `PROMETHEUS_PORT` | `9090` | Prometheus API port |
| `TEMPO_PORT` | `3200` | Tempo API port |
| `LOKI_PORT` | `3100` | Loki API port |
| `PYROSCOPE_PORT` | `4040` | Pyroscope API port |
| `OTLP_GRPC_PORT` | `4317` | OTLP gRPC port |
| `OTLP_HTTP_PORT` | `4318` | OTLP HTTP port |

### Data Persistence

| Variable | Default | Description |
|----------|---------|-------------|
| `GRAFANA_DATA_PATH` | `/data/grafana` | Grafana data directory |
| `PROMETHEUS_DATA_PATH` | `/data/prometheus` | Prometheus data directory |
| `LOKI_DATA_PATH` | `/data/loki` | Loki data directory |
| `TEMPO_DATA_PATH` | `/data/tempo` | Tempo data directory |
| `PYROSCOPE_DATA_PATH` | `/data/pyroscope` | Pyroscope data directory |

## 📁 Configuration Files

### Grafana Configuration

#### Datasources (`grafana-datasources.yaml`)
```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    uid: prometheus
    url: http://127.0.0.1:9090
    editable: true

  - name: Tempo
    type: tempo
    uid: tempo
    url: http://127.0.0.1:3200
    editable: true

  - name: Loki
    type: loki
    uid: loki
    url: http://127.0.0.1:3100
    editable: true

  - name: Pyroscope
    type: grafana-pyroscope-datasource
    uid: pyroscope
    url: http://127.0.0.1:4040
    editable: true
```

#### Dashboards (`grafana-dashboards.yaml`)
```yaml
apiVersion: 1

providers:
  - name: 'default'
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /otel-lgtm/grafana/conf/provisioning/dashboards
```

### Prometheus Configuration

#### Core Config (`prometheus.yaml`)
```yaml
---
global:
  scrape_native_histograms: true
otlp:
  keep_identifying_resource_attributes: true
  promote_resource_attributes:
    - service.instance.id
    - service.name
    - service.namespace
    - service.version
    - cloud.availability_zone
    - cloud.region
    - container.name
    - deployment.environment
    - deployment.environment.name
    - k8s.cluster.name
    - k8s.container.name
    - k8s.cronjob.name
    - k8s.daemonset.name
    - k8s.deployment.name
    - k8s.job.name
    - k8s.namespace.name
    - k8s.node.name
    - k8s.pod.name
    - k8s.replicaset.name
    - k8s.statefulset.name
    - host.name
    - postgresql.database.name
    - postgresql.schema.name
    - postgresql.table.name
    - postgresql.index.name
    - database
    - kafka.cluster.alias
storage:
  tsdb:
    out_of_order_time_window: 10m
```

### Tempo Configuration

#### Core Config (`tempo-config.yaml`)
```yaml
server:
  http_listen_port: 3200
  grpc_listen_port: 9096

distributor:
  receivers:
    otlp:
      protocols:
        grpc:
          endpoint: "127.0.0.1:4417"
        http:
          endpoint: "127.0.0.1:4418"

ingester:
  trace_idle_period: 1s
  max_block_duration: 1s
  flush_check_period: 1s

storage:
  trace:
    backend: local
    wal:
      path: /data/tempo/wal
    local:
      path: /data/tempo/blocks

metrics_generator:
  processor:
    local_blocks:
      filter_server_spans: false
    span_metrics:
      dimensions:
        - service_name
        - operation
        - status_code
  traces_storage:
    path: /data/tempo/generator/traces
  storage:
    path: /data/tempo/generator/wal
    remote_write:
      - url: http://127.0.0.1:9090/api/v1/write
        send_exemplars: true
```

### Loki Configuration

#### Core Config (`loki-config.yaml`)
```yaml
auth_enabled: false

server:
  http_listen_port: 3100

common:
  instance_addr: 127.0.0.1
  path_prefix: /data/loki
  storage:
    filesystem:
      chunks_directory: /data/loki/chunks
      rules_directory: /data/loki/rules
  replication_factor: 1
  ring:
    kvstore:
      store: inmemory

schema_config:
  configs:
    - from: 2020-10-24
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h

ruler:
  alertmanager_url: http://127.0.0.1:9093

pattern_ingester:
  lifecycler:
    min_ready_duration: 1s

ingester:
  lifecycler:
    min_ready_duration: 1s

frontend:
  scheduler_dns_lookup_period: 1s
  address: 127.0.0.1

query_scheduler:
  use_scheduler_ring: false
```

### Pyroscope Configuration

#### Core Config (`pyroscope-config.yaml`)
```yaml
server:
  grpc_listen_port: 9097

metastore:
  address: 127.0.0.1
  min_ready_duration: 1s

distributor:
  ring:
    kvstore:
      store: inmemory

ingester:
  lifecycler:
    address: 127.0.0.1
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
    min_ready_duration: 1s

pyroscopedb:
  data_path: "/data/pyroscope"
```

## 🔄 Custom Configuration Examples

### Development Environment
```bash
# Enable debug logging
export ENABLE_LOGS_ALL=true

# Use different ports to avoid conflicts
export GRAFANA_PORT=3001
export PROMETHEUS_PORT=9091

# Custom admin credentials
export GF_SECURITY_ADMIN_USER=developer
export GF_SECURITY_ADMIN_PASSWORD=dev123

# Run with custom config
./run-lgtm.sh
```

### Production-like Setup
```bash
# Secure admin credentials
export GF_SECURITY_ADMIN_USER=admin
export GF_SECURITY_ADMIN_PASSWORD=$(openssl rand -base64 32)

# Enable specific component logging
export ENABLE_LOGS_OTELCOL=true
export ENABLE_LOGS_PROMETHEUS=true

# External data export
export OTEL_EXPORTER_OTLP_ENDPOINT=https://observability.example.com:4317
export OTEL_EXPORTER_OTLP_HEADERS="authorization=Bearer ${API_TOKEN}"

# Run with production config
./run-lgtm.sh
```

### Custom Data Paths
```bash
# Use external volumes
export GRAFANA_DATA_PATH=/external/grafana
export PROMETHEUS_DATA_PATH=/external/prometheus

# Run with volume mounts
docker run -v /host/data:/external \
  -e GRAFANA_DATA_PATH=/external/grafana \
  -e PROMETHEUS_DATA_PATH=/external/prometheus \
  grafana/otel-lgtm:latest
```

## 🔧 Advanced Configuration

### Custom OTLP Collector Configuration

#### Adding Custom Processors
```yaml
processors:
  batch:
    timeout: 5s
    send_batch_size: 1024

  # Add service name to all telemetry
  resource:
    attributes:
      - key: service.namespace
        value: "lgtm-stack"
        action: insert

  # Filter out health check spans
  filter:
    spans:
      exclude:
        match_type: strict
        span_names: ["/health", "/ready"]
```

#### Custom Exporters
```yaml
exporters:
  # Additional Prometheus instance
  otlphttp/metrics-secondary:
    endpoint: http://secondary-prometheus:9090/api/v1/otlp
    tls:
      insecure: true

  # Debug exporter for development
  debug/traces:
    verbosity: detailed
    sampling_initial: 5
    sampling_thereafter: 200

pipelines:
  traces:
    exporters: [otlphttp/traces, debug/traces]
  metrics:
    exporters: [otlphttp/metrics, otlphttp/metrics-secondary]
```

### Custom Grafana Plugins

#### Installing Plugins
```bash
# Via environment variable
export GF_PLUGINS_PREINSTALL=grafana-piechart-panel,grafana-worldmap-panel

# Via configuration file
# Add to grafana-datasources.yaml or create custom config
```

#### Plugin Configuration
```yaml
# Custom plugin settings in Grafana config
[plugins]
allow_loading_unsigned_plugins = my-custom-plugin
```

### Custom Dashboards

#### Provisioning Custom Dashboards
```yaml
# grafana-dashboards.yaml
apiVersion: 1

providers:
  - name: 'custom'
    type: file
    disableDeletion: false
    updateIntervalSeconds: 30
    allowUiUpdates: true
    options:
      path: /custom-dashboards
```

#### Dashboard JSON Structure
```json
{
  "dashboard": {
    "title": "Custom Observability Dashboard",
    "tags": ["custom", "observability"],
    "timezone": "browser",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "datasource": "Prometheus"
          }
        ]
      }
    ]
  }
}
```

## 🔒 Security Configuration

### Authentication
```bash
# Grafana authentication
export GF_SECURITY_ADMIN_USER=secure_admin
export GF_SECURITY_ADMIN_PASSWORD=$(openssl rand -hex 32)

# Disable anonymous access
export GF_AUTH_ANONYMOUS_ENABLED=false

# Enable OAuth (example: GitHub)
export GF_AUTH_GITHUB_ENABLED=true
export GF_AUTH_GITHUB_CLIENT_ID=your_client_id
export GF_AUTH_GITHUB_CLIENT_SECRET=your_client_secret
```

### Network Security
```bash
# Bind to specific interfaces (not implemented in current version)
# Would require custom container build

# Use reverse proxy for TLS termination
# Configure external nginx/traefik/caddy
```

### Data Encryption
```bash
# External encryption (not built-in)
# Use encrypted volumes or external KMS

# TLS for external communications
# Configure external load balancer with TLS
```

## 📊 Monitoring Configuration

### Enable Component Metrics
```bash
# Enable detailed logging
export ENABLE_LOGS_ALL=true

# Custom log levels
export GRAFANA_LOG_LEVEL=debug
export PROMETHEUS_LOG_LEVEL=info
```

### Health Check Configuration
```bash
# Custom health check endpoints
# (Currently fixed in container)

# External monitoring
# Configure external monitoring systems
```

## 🚀 Performance Tuning

### Resource Limits
```bash
# Docker resource limits (external)
docker run --memory=4g --cpus=2 grafana/otel-lgtm:latest

# Component-specific tuning
export PROMETHEUS_MEMORY_LIMIT=2g
export TEMPO_MEMORY_LIMIT=1g
```

### Batch Processing
```yaml
# OTLP Collector batch tuning
processors:
  batch:
    timeout: 10s
    send_batch_size: 2048
    send_batch_max_size: 4096
```

### Query Optimization
```yaml
# Prometheus query settings
query:
  timeout: 2m
  max_samples: 50000000

# Tempo query settings
querier:
  max_concurrent_queries: 20
  timeout: 30s
```

## 🔄 Configuration Management

### Version Control
```bash
# Store configurations in git
git add docker/otelcol-config.yaml
git add docker/prometheus.yaml
git commit -m "Update observability configuration"
```

### Environment-Specific Configs
```bash
# Development
export ENV=dev
export LOG_LEVEL=debug

# Staging
export ENV=staging
export LOG_LEVEL=info

# Production
export ENV=prod
export LOG_LEVEL=warn
```

### Configuration Validation
```bash
# Validate OTLP Collector config
otelcol-contrib --config docker/otelcol-config.yaml --dry-run

# Validate Prometheus config
promtool check config docker/prometheus.yaml

# Validate Tempo config
tempo --config.file=docker/tempo-config.yaml --dry-run
```

## 🐛 Troubleshooting Configuration

### Common Issues

#### Configuration Not Applied
```bash
# Check environment variables
env | grep -E "(GF_|ENABLE_|OTEL_)"

# Check configuration file syntax
cat docker/otelcol-config.yaml | python3 -c "import yaml; yaml.safe_load(open(0))"

# Restart container
docker restart lgtm
```

#### Port Conflicts
```bash
# Check port usage
netstat -tulpn | grep :3000

# Use different ports
export GRAFANA_PORT=3001
export PROMETHEUS_PORT=9091
```

#### Permission Issues
```bash
# Check data directory permissions
ls -la /data/

# Fix permissions
chmod -R 755 /data/grafana
chown -R 472:472 /data/grafana
```

---

Configuration is key to customizing the LGTM Stack for your needs. Start with environment variables for simple changes, then move to configuration files for complex setups. For examples, see the [Examples Documentation](../examples/index.md). For troubleshooting, refer to the [Troubleshooting Guide](../operations/troubleshooting.md). ⚙️