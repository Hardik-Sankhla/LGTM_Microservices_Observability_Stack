# Data Flow Architecture

This document explains how telemetry data flows through the LGTM Stack, from application instrumentation to storage and visualization.

## 🔄 End-to-End Data Flow

### High-Level Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Application   │───►│ OTLP Collector  │───►│   Backends      │
│ (Instrumented)  │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │                        │
                              ▼                        ▼
                       ┌─────────────┐        ┌─────────────────┐
                       │  Processing │        │   Storage       │
                       │             │        │                 │
                       └─────────────┘        └─────────────────┘
                                                │
                                                ▼
                                       ┌─────────────────┐
                                       │   Grafana       │
                                       │ (Visualization) │
                                       └─────────────────┘
```

## 📊 Telemetry Types and Flows

### 1. Metrics Flow

```
Application Metrics
        │
        ▼
OpenTelemetry SDK
- Counter, Histogram, Gauge
- Custom business metrics
- Runtime performance metrics
        │
        ▼
OTLP Protocol (HTTP/gRPC)
- Binary encoding
- Metadata preservation
        │
        ▼
OTLP Collector
- Receives OTLP metrics
- Applies batch processing
- Routes to Prometheus
        │
        ▼
Prometheus
- OTLP ingestion endpoint
- Time-series storage
- Metric processing and aggregation
        │
        ▼
Grafana
- PromQL queries
- Dashboard visualization
- Alerting rules
```

**Key Components**:
- **SDK**: `opentelemetry.metrics`
- **Protocol**: OTLP metrics
- **Storage**: Prometheus TSDB
- **Query**: PromQL
- **Visualization**: Grafana panels

### 2. Traces Flow

```
Application Requests
        │
        ▼
OpenTelemetry SDK
- Span creation
- Context propagation
- Error recording
- Custom attributes
        │
        ▼
OTLP Protocol
- Span batching
- Trace context
- Resource attributes
        │
        ▼
OTLP Collector
- Trace routing
- Batch optimization
- Backend forwarding
        │
        ▼
Tempo
- Trace ingestion
- Storage and indexing
- Search optimization
        │
        ▼
Grafana
- Trace search UI
- Service graph visualization
- Span detail views
```

**Key Components**:
- **SDK**: `opentelemetry.trace`
- **Protocol**: OTLP traces
- **Storage**: Tempo backend
- **Query**: TraceQL
- **Visualization**: Tempo UI in Grafana

### 3. Logs Flow

```
Application Events
        │
        ▼
OpenTelemetry SDK
- Structured logging
- Log correlation
- Error tracking
- Context injection
        │
        ▼
OTLP Protocol
- Log record batching
- Metadata preservation
- Trace correlation
        │
        ▼
OTLP Collector
- Log processing
- Format normalization
- Backend routing
        │
        ▼
Loki
- Log ingestion
- Label indexing
- Storage optimization
        │
        ▼
Grafana
- LogQL queries
- Log exploration
- Trace correlation
```

**Key Components**:
- **SDK**: `opentelemetry.logging`
- **Protocol**: OTLP logs
- **Storage**: Loki log store
- **Query**: LogQL
- **Visualization**: Log panels in Grafana

### 4. Profiles Flow

```
Application Performance
        │
        ▼
OpenTelemetry SDK
- CPU profiling
- Memory profiling
- Custom profiling
- Performance sampling
        │
        ▼
OTLP Protocol
- Profile data encoding
- Metadata attachment
- Sample aggregation
        │
        ▼
OTLP Collector
- Profile routing
- Data validation
- Backend forwarding
        │
        ▼
Pyroscope
- Profile ingestion
- Flame graph generation
- Performance analysis
        │
        ▼
Grafana
- Flame graph visualization
- Profile comparison
- Performance insights
```

**Key Components**:
- **SDK**: Profiling instrumentation
- **Protocol**: OTLP profiles
- **Storage**: Pyroscope backend
- **Visualization**: Flame graphs
- **Analysis**: Performance profiling

## 🔄 Data Processing Pipeline

### Stage 1: Application Instrumentation

```python
# Example: Python application with OpenTelemetry
from opentelemetry import trace, metrics
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.metrics import MeterProvider
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.exporter.otlp.proto.grpc.metric_exporter import OTLPMetricExporter

# Initialize tracing
trace.set_tracer_provider(TracerProvider())
trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(OTLPSpanExporter(endpoint="localhost:4317"))
)

# Initialize metrics
metrics.set_meter_provider(MeterProvider())
meter = metrics.get_meter("my-app")
counter = meter.create_counter("requests_total")

# Instrument application
with tracer.start_as_span("business_operation") as span:
    span.set_attribute("user.id", user_id)
    counter.add(1, {"method": "GET", "endpoint": "/api/users"})
```

### Stage 2: OTLP Transport

#### Protocol Structure
```protobuf
// OTLP Trace Message
message ExportTraceServiceRequest {
  repeated ResourceSpans resource_spans = 1;
}

message ResourceSpans {
  Resource resource = 1;
  repeated ScopeSpans scope_spans = 2;
}

message ScopeSpans {
  InstrumentationScope scope = 1;
  repeated Span spans = 2;
}
```

#### Transport Options
- **gRPC** (port 4317): Efficient, bidirectional, binary
- **HTTP** (port 4318): REST API, JSON support, CORS enabled

### Stage 3: Collector Processing

#### Data Enrichment
```yaml
processors:
  batch:
    timeout: 1s
    send_batch_size: 1024

  # Add resource attributes
  resource:
    attributes:
      - key: service.version
        value: "1.0.0"
        action: insert
```

#### Routing Logic
- **Metrics** → Prometheus OTLP endpoint
- **Traces** → Tempo OTLP endpoint
- **Logs** → Loki OTLP endpoint
- **Profiles** → Pyroscope OTLP endpoint

### Stage 4: Backend Storage

#### Prometheus Metrics Storage
```yaml
# Prometheus receives OTLP metrics
POST /api/v1/otlp/v1/metrics
Content-Type: application/x-protobuf

# Converts to Prometheus format
requests_total{method="GET",endpoint="/api/users"} 42
```

#### Tempo Trace Storage
```yaml
# Tempo receives OTLP traces
POST /otlp/v1/traces
Content-Type: application/x-protobuf

# Stores in Tempo format
trace_id: "a1b2c3d4..."
spans: [...]
```

#### Loki Log Storage
```yaml
# Loki receives OTLP logs
POST /otlp/v1/logs
Content-Type: application/x-protobuf

# Stores with labels
{job="my-app", level="info"} log line here
```

### Stage 5: Query and Visualization

#### Grafana Data Source Queries

**Prometheus Query**:
```promql
# Request rate by endpoint
rate(http_requests_total[5m])
```

**Tempo Query**:
```traceql
# Find traces with errors
{status = error}
```

**Loki Query**:
```logql
# Logs with specific pattern
{job="my-app"} |= "ERROR" | json
```

## 🔗 Data Correlation

### Trace-to-Metrics Correlation
```
Trace Span
├── Duration: 250ms
├── Tags: {service.name="web", operation="http_request"}
└── Metrics: request_duration_seconds{quantile="0.95"} = 0.25
```

### Trace-to-Logs Correlation
```
Trace ID: a1b2c3d4-e5f6-7890-abcd-ef1234567890
├── Span: "process_request"
├── Log: {trace_id="a1b2c3d4-e5f6-7890-abcd-ef1234567890"} Processing request
└── Log: {trace_id="a1b2c3d4-e5f6-7890-abcd-ef1234567890"} Request completed
```

### Metrics-to-Traces Correlation (Exemplars)
```
Metric: http_request_duration_seconds{quantile="0.95"} = 0.250
Exemplar:
├── Trace ID: a1b2c3d4-e5f6-7890-abcd-ef1234567890
├── Span ID: b2c3d4e5-f6g7-8901-bcde-f23456789012
└── Value: 0.250
```

## ⚡ Real-Time Data Flow

### Ingestion Latency
- **Target**: <100ms end-to-end
- **OTLP Transport**: <10ms
- **Collector Processing**: <5ms
- **Backend Ingestion**: <50ms
- **Indexing**: <50ms

### Query Latency
- **Metrics**: <1s for simple queries
- **Traces**: <5s for trace searches
- **Logs**: <2s for recent logs
- **Dashboards**: <3s for panel loads

### Data Freshness
- **Metrics**: Real-time (scrape intervals)
- **Traces**: Near real-time (<1s)
- **Logs**: Real-time streaming
- **Profiles**: Batch-based (configurable)

## 🔄 Data Retention and Lifecycle

### Storage Tiers
```
Hot Storage (Recent Data)
├── Prometheus: 15 days (configurable)
├── Tempo: 30 days (configurable)
├── Loki: 30 days (configurable)
└── Pyroscope: 30 days (configurable)

Warm Storage (Historical Data)
└── Optional: External object storage

Cold Storage (Archive)
└── Optional: Long-term retention
```

### Data Compaction
- **Prometheus**: Block compaction every 2 hours
- **Tempo**: Trace compaction and indexing
- **Loki**: Log chunk compaction
- **Pyroscope**: Profile data compaction

### Cleanup Processes
- **Automatic**: Background cleanup jobs
- **Configurable**: Retention policies
- **Manual**: Administrative deletion APIs

## 🚨 Error Handling and Resilience

### Collector Failure Scenarios
```
Network Partition
├── Collector ↔ Backend: Retry with backoff
├── Collector ↔ Application: Buffer and retry
└── Data Loss: Minimal (buffered)

Backend Unavailable
├── Retry Logic: Exponential backoff
├── Circuit Breaker: Prevent cascade failures
└── Fallback: Local buffering
```

### Data Quality Issues
```
Invalid Data
├── Validation: Schema validation
├── Sanitization: Data cleaning
├── Rejection: Invalid data dropped
└── Logging: Error reporting
```

### Backpressure Handling
```
High Load
├── Batch Sizing: Adaptive batching
├── Rate Limiting: Prevent overload
├── Queue Management: Bounded queues
└── Shedding: Drop low-priority data
```

## 📊 Monitoring Data Flow

### Flow Metrics
```promql
# Collector throughput
rate(otelcol_exporter_sent_spans_total[5m])

# Backend ingestion rate
rate(prometheus_remote_write_samples_total[5m])

# Query performance
histogram_quantile(0.95, rate(grafana_query_duration_seconds_bucket[5m]))
```

### Health Checks
- **Collector**: `/ready` endpoint
- **Backends**: Service-specific health checks
- **Grafana**: `/api/health` endpoint

### Alerting Rules
```yaml
# Data flow alerts
- alert: CollectorDown
  expr: up{job="otelcol"} == 0
  for: 5m

- alert: HighIngestionLatency
  expr: histogram_quantile(0.95, rate(otelcol_processor_batch_duration_bucket[5m])) > 10
  for: 5m
```

## 🔧 Troubleshooting Data Flow

### Common Issues

#### Data Not Appearing
```bash
# Check OTLP ingestion
curl -X POST http://localhost:4318/v1/traces \
  -H "Content-Type: application/json" \
  -d '{"resourceSpans":[]}'

# Verify backend health
curl http://localhost:9090/api/v1/status/runtimeinfo

# Check Grafana data sources
curl http://localhost:3000/api/datasources
```

#### High Latency
```bash
# Monitor collector metrics
curl http://localhost:8888/metrics | grep batch

# Check backend performance
curl http://localhost:9090/api/v1/query?query=up

# Analyze network latency
ping localhost
```

#### Data Loss
```bash
# Check collector logs
docker logs lgtm | grep -i error

# Verify disk space
df -h /data

# Check retention settings
curl http://localhost:9090/api/v1/status/config
```

## 🚀 Performance Optimization

### Collector Tuning
```yaml
processors:
  batch:
    timeout: 5s
    send_batch_size: 2048
    send_batch_max_size: 4096
```

### Backend Optimization
- **Prometheus**: Increase memory limits
- **Tempo**: Adjust block sizes
- **Loki**: Configure chunk sizes
- **Grafana**: Enable query caching

### Network Optimization
- **Keep-Alive**: Persistent connections
- **Compression**: Enable OTLP compression
- **Connection Pooling**: Reuse connections

---

Understanding data flow is crucial for effective observability. This document explains how telemetry moves from applications to visualization. For component-specific details, see the [Components Documentation](components/index.md). For configuration examples, refer to the [Configuration Guide](configuration/index.md). 🔄📊