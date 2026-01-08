# Environment Variables Reference

This document provides a comprehensive reference for all environment variables supported by the LGTM Stack.

## 📋 Variable Categories

### 🔐 Security & Authentication
| Variable | Default | Description | Example |
|----------|---------|-------------|---------|
| `GF_SECURITY_ADMIN_USER` | `admin` | Grafana admin username | `myadmin` |
| `GF_SECURITY_ADMIN_PASSWORD` | `admin` | Grafana admin password | `securepass123` |
| `GF_AUTH_ANONYMOUS_ENABLED` | `true` | Enable anonymous access | `false` |
| `GF_AUTH_BASIC_ENABLED` | `true` | Enable basic authentication | `true` |

### 📊 Grafana Configuration
| Variable | Default | Description | Example |
|----------|---------|-------------|---------|
| `GF_SERVER_HTTP_PORT` | `3000` | Grafana web server port | `3001` |
| `GF_PLUGINS_PREINSTALL` | - | Comma-separated plugin list | `grafana-piechart-panel,grafana-worldmap-panel` |
| `GF_LOG_LEVEL` | `info` | Logging level | `debug` |
| `GF_PATHS_DATA` | `/data/grafana` | Data directory path | `/custom/grafana` |

### 🔍 Logging Configuration
| Variable | Default | Description | Example |
|----------|---------|-------------|---------|
| `ENABLE_LOGS_ALL` | `false` | Enable all component logs | `true` |
| `ENABLE_LOGS_GRAFANA` | `false` | Enable Grafana logs | `true` |
| `ENABLE_LOGS_LOKI` | `false` | Enable Loki logs | `true` |
| `ENABLE_LOGS_PROMETHEUS` | `false` | Enable Prometheus logs | `true` |
| `ENABLE_LOGS_TEMPO` | `false` | Enable Tempo logs | `true` |
| `ENABLE_LOGS_PYROSCOPE` | `false` | Enable Pyroscope logs | `true` |
| `ENABLE_LOGS_OTELCOL` | `false` | Enable OTLP Collector logs | `true` |

### 🌐 External Data Export
| Variable | Default | Description | Example |
|----------|---------|-------------|---------|
| `OTEL_EXPORTER_OTLP_ENDPOINT` | - | External OTLP endpoint | `https://api.honeycomb.io:443` |
| `OTEL_EXPORTER_OTLP_HEADERS` | - | Headers for external export | `x-honeycomb-team=abc123,x-honeycomb-dataset=myapp` |
| `OTEL_EXPORTER_OTLP_PROTOCOL` | `http/protobuf` | Export protocol | `grpc` |

### 🔌 Port Configuration
| Variable | Default | Description | Example |
|----------|---------|-------------|---------|
| `GRAFANA_PORT` | `3000` | Grafana web UI port | `3001` |
| `PROMETHEUS_PORT` | `9090` | Prometheus API port | `9091` |
| `TEMPO_PORT` | `3200` | Tempo API port | `3201` |
| `LOKI_PORT` | `3100` | Loki API port | `3101` |
| `PYROSCOPE_PORT` | `4040` | Pyroscope API port | `4041` |
| `OTLP_GRPC_PORT` | `4317` | OTLP gRPC port | `4319` |
| `OTLP_HTTP_PORT` | `4318` | OTLP HTTP port | `4320` |

### 💾 Data Path Configuration
| Variable | Default | Description | Example |
|----------|---------|-------------|---------|
| `GRAFANA_DATA_PATH` | `/data/grafana` | Grafana data directory | `/persistent/grafana` |
| `PROMETHEUS_DATA_PATH` | `/data/prometheus` | Prometheus data directory | `/persistent/prometheus` |
| `LOKI_DATA_PATH` | `/data/loki` | Loki data directory | `/persistent/loki` |
| `TEMPO_DATA_PATH` | `/data/tempo` | Tempo data directory | `/persistent/tempo` |
| `PYROSCOPE_DATA_PATH` | `/data/pyroscope` | Pyroscope data directory | `/persistent/pyroscope` |

## 🔧 Usage Examples

### Basic Development Setup
```bash
# Enable debug logging for troubleshooting
export ENABLE_LOGS_ALL=true

# Use different ports to avoid conflicts
export GRAFANA_PORT=3001
export PROMETHEUS_PORT=9091

# Run the stack
./run-lgtm.sh
```

### Secure Production Setup
```bash
# Secure admin credentials
export GF_SECURITY_ADMIN_USER=observability_admin
export GF_SECURITY_ADMIN_PASSWORD=$(openssl rand -base64 32)

# Disable anonymous access
export GF_AUTH_ANONYMOUS_ENABLED=false

# Enable component logging
export ENABLE_LOGS_OTELCOL=true
export ENABLE_LOGS_PROMETHEUS=true

# Run with security
./run-lgtm.sh
```

### External Observability Integration
```bash
# Send data to Grafana Cloud
export OTEL_EXPORTER_OTLP_ENDPOINT=https://otlp-gateway-prod-us-east-0.grafana.net/otlp
export OTEL_EXPORTER_OTLP_HEADERS="authorization=Bearer glc_..."

# Send data to Honeycomb
export OTEL_EXPORTER_OTLP_ENDPOINT=https://api.honeycomb.io:443
export OTEL_EXPORTER_OTLP_HEADERS="x-honeycomb-team=your-api-key,x-honeycomb-dataset=lgtm-stack"

# Send data to DataDog
export OTEL_EXPORTER_OTLP_ENDPOINT=https://trace.agent.datadoghq.com
export OTEL_EXPORTER_OTLP_HEADERS="dd-api-key=your-datadog-api-key"

./run-lgtm.sh
```

### Custom Data Paths
```bash
# Use external persistent volumes
export GRAFANA_DATA_PATH=/mnt/persistent/grafana
export PROMETHEUS_DATA_PATH=/mnt/persistent/prometheus
export LOKI_DATA_PATH=/mnt/persistent/loki
export TEMPO_DATA_PATH=/mnt/persistent/tempo
export PYROSCOPE_DATA_PATH=/mnt/persistent/pyroscope

# Run with custom paths
docker run -v /host/data:/mnt/persistent \
  -e GRAFANA_DATA_PATH=/mnt/persistent/grafana \
  -e PROMETHEUS_DATA_PATH=/mnt/persistent/prometheus \
  grafana/otel-lgtm:latest
```

### Grafana Plugin Installation
```bash
# Install additional plugins
export GF_PLUGINS_PREINSTALL=grafana-piechart-panel,grafana-worldmap-panel,grafana-clock-panel

# Custom plugin from URL
export GF_PLUGINS_PREINSTALL=grafana-piechart-panel,https://github.com/ryantxu/ajax-panel/releases/download/v0.2.0/ryantxu-ajax-panel-0.2.0.zip

./run-lgtm.sh
```

## 🔍 Variable Precedence

Environment variables take precedence over configuration files:

```
Environment Variable (highest priority)
    │
    ▼
Configuration File Settings
    │
    ▼
Container Default Values (lowest priority)
```

## ✅ Validation

### Required Variables
- No variables are strictly required (all have defaults)
- External export variables are optional

### Format Validation
```bash
# Port validation (must be numeric)
export GRAFANA_PORT=3000  # ✅ Valid
export GRAFANA_PORT=abc   # ❌ Invalid

# URL validation
export OTEL_EXPORTER_OTLP_ENDPOINT=https://valid.url:443  # ✅ Valid
export OTEL_EXPORTER_OTLP_ENDPOINT=invalid-url           # ❌ Invalid
```

### Runtime Validation
The container will validate variables at startup:

```bash
# Check logs for validation errors
docker logs lgtm | grep -i "error\|invalid\|failed"

# Common validation messages
# - "invalid port number"
# - "invalid URL format"
# - "permission denied"
# - "directory not writable"
```

## 🐛 Troubleshooting

### Variables Not Applied
```bash
# Check variable is set
echo $GF_SECURITY_ADMIN_PASSWORD

# Check variable in container
docker exec lgtm env | grep GF_

# Restart container after variable changes
docker restart lgtm
```

### Port Conflicts
```bash
# Check what's using the port
lsof -i :3000

# Use different port
export GRAFANA_PORT=3001
docker run -p 3001:3000 grafana/otel-lgtm:latest
```

### Permission Issues
```bash
# Check data directory permissions
ls -la /data/grafana

# Fix permissions
sudo chown -R 472:472 /data/grafana
sudo chmod -R 755 /data/grafana
```

### External Export Issues
```bash
# Test connectivity
curl -I $OTEL_EXPORTER_OTLP_ENDPOINT

# Check headers format
echo $OTEL_EXPORTER_OTLP_HEADERS

# Verify authentication
curl -H "$OTEL_EXPORTER_OTLP_HEADERS" $OTEL_EXPORTER_OTLP_ENDPOINT/v1/traces
```

## 📚 Advanced Usage

### Variable Expansion
```bash
# Use other variables in values
export BASE_DOMAIN=example.com
export OTEL_EXPORTER_OTLP_ENDPOINT=https://otel.$BASE_DOMAIN:443

# Use command substitution
export GF_SECURITY_ADMIN_PASSWORD=$(openssl rand -base64 32)
export API_KEY=$(cat /secrets/api-key)
```

### Conditional Variables
```bash
# Environment-specific configuration
if [ "$ENV" = "production" ]; then
  export ENABLE_LOGS_ALL=false
  export GF_SECURITY_ADMIN_PASSWORD=$PROD_PASSWORD
else
  export ENABLE_LOGS_ALL=true
  export GF_SECURITY_ADMIN_PASSWORD=admin
fi
```

### Variable Files
```bash
# Load variables from file
set -a
source .env
set +a

# Or use docker --env-file
docker run --env-file .env grafana/otel-lgtm:latest
```

## 🔐 Security Best Practices

### Secret Management
```bash
# Never hardcode secrets
# export GF_SECURITY_ADMIN_PASSWORD=secret123  # ❌ Bad

# Use secure generation
export GF_SECURITY_ADMIN_PASSWORD=$(openssl rand -base64 32)  # ✅ Good

# Use external secret management
export GF_SECURITY_ADMIN_PASSWORD=$(vault kv get -field=password secret/grafana)  # ✅ Better
```

### Access Control
```bash
# Disable anonymous access in production
export GF_AUTH_ANONYMOUS_ENABLED=false

# Use strong passwords
export GF_SECURITY_ADMIN_PASSWORD=$(openssl rand -base64 32)

# Enable audit logging
export ENABLE_LOGS_GRAFANA=true
```

### Network Security
```bash
# Don't expose sensitive ports externally
# Use reverse proxy or firewall rules

# Use HTTPS for external access
# Configure TLS termination externally
```

## 📊 Monitoring Variables

### Debug Variables
```bash
# Enable detailed logging
export ENABLE_LOGS_ALL=true

# Component-specific debug
export GRAFANA_LOG_LEVEL=debug
export PROMETHEUS_LOG_LEVEL=debug
```

### Performance Variables
```bash
# Adjust resource usage
export PROMETHEUS_MEMORY_LIMIT=2g
export TEMPO_MEMORY_LIMIT=1g

# Tune batch processing
export OTELCOL_BATCH_TIMEOUT=10s
export OTELCOL_BATCH_SIZE=2048
```

---

Environment variables provide flexible configuration for the LGTM Stack. Start with the basic examples, then explore advanced usage for production deployments. For configuration file options, see the [Configuration Guide](index.md). For security considerations, refer to the [Security Guide](../production/security.md). 🔧