# OpenTelemetry Collector

The OpenTelemetry Collector is the central telemetry ingestion and routing component in the LGTM Stack. It receives telemetry data from instrumented applications and routes it to the appropriate backend services.

## 🎯 Role and Responsibilities

### Primary Functions
- **Telemetry Ingestion**: Single entry point for all OTLP data
- **Protocol Translation**: Converts between different telemetry protocols
- **Data Processing**: Applies transformations and filtering
- **Load Distribution**: Routes data to appropriate storage backends
- **Health Monitoring**: Provides operational health checks

### Architecture Overview
```
OpenTelemetry Collector
├── Receivers (Data Input)
│   ├── OTLP gRPC (port 4317)
│   ├── OTLP HTTP (port 4318)
│   └── Prometheus (self-metrics)
├── Processors (Data Transformation)
│   └── Batch (efficiency optimization)
├── Exporters (Data Output)
│   ├── OTLP → Prometheus (metrics)
│   ├── OTLP → Tempo (traces)
│   ├── OTLP → Loki (logs)
│   └── OTLP → Pyroscope (profiles)
└── Extensions (Operational Support)
    └── Health Check (port 13133)
```

## ⚙️ Configuration

### Primary Configuration (`otelcol-config.yaml`)

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318
        cors:
          allowed_origins:
            - http://*
  prometheus/collector:
    config:
      scrape_configs:
        - job_name: "opentelemetry-collector"
          scrape_interval: 1s
          static_configs:
            - targets: ["127.0.0.1:8888"]

extensions:
  health_check:
    endpoint: 0.0.0.0:13133
    path: "/ready"

processors:
  batch:

exporters:
  otlphttp/metrics:
    endpoint: http://127.0.0.1:9090/api/v1/otlp
    tls:
      insecure: true
  otlphttp/traces:
    endpoint: http://127.0.0.1:4418
    tls:
      insecure: true
  otlphttp/logs:
    endpoint: http://127.0.0.1:3100/otlp
    tls:
      insecure: true
  otlp/profiles:
    endpoint: http://127.0.0.1:4040
    tls:
      insecure: true
  debug/metrics:
    verbosity: detailed
  debug/traces:
    verbosity: detailed
  debug/logs:
    verbosity: detailed

service:
  extensions: [health_check]
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlphttp/traces]
    metrics:
      receivers: [otlp, prometheus/collector]
      processors: [batch]
      exporters: [otlphttp/metrics]
    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlphttp/logs]
    profiles:
      receivers: [otlp]
      exporters: [otlp/profiles]
```

### Export Configuration (`otelcol-config-export-http.yaml`)

```yaml
service:
  pipelines:
    traces:
      exporters: [otlphttp/traces, otlphttp/external]
    metrics:
      exporters: [otlphttp/metrics, otlphttp/external]
    logs:
      exporters: [otlphttp/logs, otlphttp/external]

exporters:
  otlphttp/external:
    endpoint: ${env:OTEL_EXPORTER_OTLP_ENDPOINT}
    headers:
      authorization: ${env:OTEL_EXPORTER_OTLP_HEADERS}
```

## 🔌 Receivers

### OTLP gRPC Receiver
- **Port**: 4317
- **Protocol**: gRPC with Protocol Buffers
- **Purpose**: High-performance telemetry ingestion
- **Features**:
  - Bidirectional streaming
  - Efficient binary encoding
  - Built-in compression

### OTLP HTTP Receiver
- **Port**: 4318
- **Protocol**: HTTP with JSON/Protobuf
- **Purpose**: Compatibility and debugging
- **Features**:
  - REST API endpoints
  - CORS support for web applications
  - Human-readable JSON format

### Prometheus Receiver
- **Purpose**: Self-monitoring metrics collection
- **Target**: Collector's own metrics (port 8888)
- **Features**:
  - Standard Prometheus scraping
  - Automatic service discovery
  - Metric relabeling

## 🔄 Processors

### Batch Processor
- **Purpose**: Improve efficiency and reduce network overhead
- **Configuration**:
  ```yaml
  processors:
    batch:
      timeout: 1s
      send_batch_size: 1024
      send_batch_max_size: 2048
  ```
- **Behavior**:
  - Groups telemetry data into batches
  - Reduces number of network calls
  - Improves backend performance

## 📤 Exporters

### Backend Routing
- **Prometheus**: `otlphttp/metrics` → `http://127.0.0.1:9090/api/v1/otlp`
- **Tempo**: `otlphttp/traces` → `http://127.0.0.1:4418`
- **Loki**: `otlphttp/logs` → `http://127.0.0.1:3100/otlp`
- **Pyroscope**: `otlp/profiles` → `http://127.0.0.1:4040`

### External Export
- **Trigger**: `OTEL_EXPORTER_OTLP_ENDPOINT` environment variable
- **Purpose**: Forward data to external observability backends
- **Configuration**: Uses `otelcol-config-export-http.yaml`

### Debug Exporters
- **Purpose**: Development and troubleshooting
- **Output**: Detailed telemetry data to logs
- **Usage**: Enable with `debug/*` exporters in pipelines

## 🌐 API Endpoints

### Health Check
```bash
# Health status
GET http://localhost:13133/ready
# Response: 200 OK when healthy
```

### Metrics Endpoint
```bash
# Collector self-metrics
GET http://localhost:8888/metrics
# Format: Prometheus exposition format
```

### OTLP Ingestion
```bash
# gRPC endpoint
grpc://localhost:4317

# HTTP endpoints
POST http://localhost:4318/v1/traces
POST http://localhost:4318/v1/metrics
POST http://localhost:4318/v1/logs
```

## 📊 Telemetry Data Flow

### Ingestion Process
```
Application → OTLP SDK → OTLP Protocol → Collector Receiver → Processor → Exporter → Backend
```

### Data Transformation
1. **Receive**: Accept OTLP data via gRPC or HTTP
2. **Decode**: Parse protocol buffers or JSON
3. **Process**: Apply batching and filtering
4. **Route**: Send to appropriate backend based on telemetry type
5. **Export**: Forward data using backend-specific protocols

### Error Handling
- **Invalid Data**: Logged and dropped
- **Backend Unavailable**: Retry with exponential backoff
- **Network Issues**: Buffer data and retry
- **Configuration Errors**: Fail fast on startup

## 🔧 Operational Details

### Startup Process
```bash
# Started by run-all.sh
./run-otelcol.sh

# Configuration validation
otelcol-contrib --config docker/otelcol-config.yaml --dry-run

# Service startup
# 1. Load configuration
# 2. Initialize receivers
# 3. Start processors
# 4. Connect exporters
# 5. Start health check endpoint
```

### Resource Usage
- **Memory**: ~50-100MB baseline
- **CPU**: Low baseline, scales with throughput
- **Network**: Moderate internal traffic
- **Disk**: Minimal (configuration only)

### Monitoring
```bash
# Health check
curl http://localhost:13133/ready

# Metrics
curl http://localhost:8888/metrics | grep otelcol_

# Logs
docker logs lgtm | grep otelcol
```

## 🐛 Troubleshooting

### Common Issues

#### Collector Not Starting
```bash
# Check configuration syntax
otelcol-contrib --config docker/otelcol-config.yaml --dry-run

# Check port availability
netstat -tulpn | grep :4317

# Check logs
docker logs lgtm | grep -A 5 -B 5 "otelcol"
```

#### Data Not Reaching Backends
```bash
# Test OTLP ingestion
curl -X POST http://localhost:4318/v1/traces \
  -H "Content-Type: application/json" \
  -d '{"resourceSpans":[]}'

# Check backend health
curl http://localhost:9090/api/v1/status/runtimeinfo

# Verify exporter configuration
docker exec lgtm cat /otel-lgtm/docker/otelcol-config.yaml
```

#### High Resource Usage
```bash
# Check batch processor settings
# Increase batch size or timeout

# Monitor metrics
curl http://localhost:8888/metrics | grep batch

# Check for stuck connections
netstat -tulpn | grep :4317
```

### Debug Mode
```bash
# Enable debug exporters
# Edit otelcol-config.yaml to include debug exporters in pipelines

# View detailed logs
docker logs -f lgtm | grep -E "(otelcol|debug)"
```

## 🔄 External Integration

### Sending Data to External Systems
```bash
# Set environment variables
export OTEL_EXPORTER_OTLP_ENDPOINT=https://my-otel-collector.com:4317
export OTEL_EXPORTER_OTLP_HEADERS="authorization=Bearer token123"

# Restart container
docker restart lgtm
```

### Supported External Backends
- **Grafana Cloud**: OTLP endpoint with authentication
- **Honeycomb**: OTLP ingestion
- **DataDog**: OTLP support
- **New Relic**: OTLP ingestion
- **Custom Collectors**: Any OTLP-compatible system

## 📈 Performance Tuning

### Batch Processor Optimization
```yaml
processors:
  batch:
    timeout: 5s          # Wait up to 5 seconds for batch
    send_batch_size: 512 # Send batches of 512 items
    send_batch_max_size: 2048  # Maximum batch size
```

### Memory Management
- **Batch Size**: Larger batches reduce memory pressure
- **Timeout**: Shorter timeouts reduce latency
- **Concurrency**: Single-threaded processing

### Network Optimization
- **Keep-Alive**: Persistent connections to backends
- **Compression**: Built-in compression for OTLP
- **Connection Pooling**: Reused connections

## 🔒 Security Considerations

### Current Security (Development)
- **No Authentication**: Open OTLP endpoints
- **No Encryption**: Plain HTTP/gRPC
- **No Access Control**: Accept all incoming data

### Production Security
- **TLS Termination**: External load balancer
- **Authentication**: API keys or OAuth
- **Rate Limiting**: Prevent abuse
- **Data Validation**: Input sanitization

## 📚 Advanced Configuration

### Custom Processors
```yaml
processors:
  attributes:
    actions:
      - key: service.name
        action: insert
        value: "lgtm-stack"
```

### Multiple Exporters
```yaml
exporters:
  otlphttp/primary:
    endpoint: http://primary-backend:4317
  otlphttp/secondary:
    endpoint: http://secondary-backend:4317

pipelines:
  traces:
    exporters: [otlphttp/primary, otlphttp/secondary]
```

### Conditional Routing
```yaml
processors:
  routing:
    from_attribute: service.name
    table:
      - value: "important-service"
        exporters: [otlphttp/high-priority]
      - value: "default"
        exporters: [otlphttp/standard]
```

## 🔗 Related Components

### Dependent Services
- **Prometheus**: Receives metrics from collector
- **Tempo**: Receives traces from collector
- **Loki**: Receives logs from collector
- **Pyroscope**: Receives profiles from collector

### Integration Points
- **Grafana**: Queries collector metrics
- **Applications**: Send telemetry to collector
- **External Systems**: Can receive forwarded data

---

The OpenTelemetry Collector is the heart of the LGTM Stack, providing reliable telemetry ingestion and routing. For configuration examples, see the [Configuration Guide](../configuration/index.md). For troubleshooting, refer to the [Troubleshooting Guide](../operations/troubleshooting.md). 🔄