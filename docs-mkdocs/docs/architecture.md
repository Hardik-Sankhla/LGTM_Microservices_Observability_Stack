# System Architecture

This document provides a comprehensive overview of the LGTM Stack's architecture, including component interactions, data flows, and design decisions.

## 🏗️ High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    External Applications                    │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │   Go Service    │  │  Java Service   │  │ Python Service  │ │
│  │                 │  │                 │  │                 │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
└─────────────────────┬──────────────────────────────────────────┘
                      │ OTLP Protocol
                      │ (gRPC:4317, HTTP:4318)
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                 OpenTelemetry Collector                     │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Receivers: OTLP, Prometheus                            │ │
│  │ Processors: Batch                                       │ │
│  │ Exporters: OTLP (internal routing)                     │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────┬────────────────────────────────────────┘
                      │ Internal Routing
           ┌──────────┼──────────┐
           │          │          │
    ┌──────▼────┐ ┌───▼────┐ ┌───▼────┐
    │ Prometheus│ │ Tempo  │ │ Loki   │
    │ (Metrics) │ │(Traces)│ │ (Logs) │
    └───────────┘ └────────┘ └────────┘
           │          │          │
           └──────────┼──────────┘
                      │
               ┌──────▼──────┐
               │   Grafana   │
               │ Dashboards  │
               └─────────────┘
                      │
                      ▼
               ┌─────────────┐
               │   Pyroscope │
               │ (Profiles)  │
               └─────────────┘
```

## 🧩 Component Architecture

### OpenTelemetry Collector

#### Role and Responsibilities
- **Central Ingestion Hub**: Single entry point for all telemetry data
- **Protocol Translation**: Converts between different telemetry protocols
- **Data Processing**: Applies transformations, filtering, and enrichment
- **Load Distribution**: Routes data to appropriate backend services
- **Health Monitoring**: Provides health check endpoints

#### Internal Structure
```
OpenTelemetry Collector
├── Receivers
│   ├── OTLP (gRPC/HTTP)
│   └── Prometheus (metrics scraping)
├── Processors
│   └── Batch (efficiency optimization)
├── Exporters
│   ├── OTLP → Prometheus (metrics)
│   ├── OTLP → Tempo (traces)
│   ├── OTLP → Loki (logs)
│   └── OTLP → Pyroscope (profiles)
└── Extensions
    └── Health Check (readiness probes)
```

#### Configuration Details
```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch: {}  # Default batch configuration

exporters:
  otlphttp/metrics:
    endpoint: http://127.0.0.1:9090/api/v1/otlp
  otlphttp/traces:
    endpoint: http://127.0.0.1:4418
  otlphttp/logs:
    endpoint: http://127.0.0.1:3100/otlp
  otlp/profiles:
    endpoint: http://127.0.0.1:4040

service:
  extensions: [health_check]
  pipelines:
    traces: [receivers: [otlp], processors: [batch], exporters: [otlphttp/traces]]
    metrics: [receivers: [otlp, prometheus/collector], processors: [batch], exporters: [otlphttp/metrics]]
    logs: [receivers: [otlp], processors: [batch], exporters: [otlphttp/logs]]
    profiles: [receivers: [otlp], exporters: [otlp/profiles]]
```

### Grafana

#### Architecture Overview
```
Grafana Server
├── HTTP Server (Port 3000)
├── Authentication & Authorization
├── Plugin System
├── Data Source Management
├── Dashboard Engine
├── Alerting Engine
└── Query Caching
```

#### Data Source Integration
- **Prometheus**: Metrics queries via PromQL
- **Tempo**: Trace queries and service graphs
- **Loki**: Log queries via LogQL
- **Pyroscope**: Profiling data visualization

#### Pre-configured Dashboards
- **RED Metrics**: Request rate, Error rate, Duration
- **JVM Metrics**: Java application performance
- **Service Graph**: Service dependency visualization
- **Logs**: Log aggregation and filtering

### Backend Services

#### Prometheus (Metrics)
```
Prometheus Server
├── Time Series Database (TSDB)
├── Service Discovery
├── Rule Engine (Alerting)
├── Query Engine (PromQL)
└── Remote Write/Read API
```

**Key Features**:
- OTLP metrics ingestion
- Native histogram support
- Exemplar support (trace links)
- Multi-dimensional data model

#### Tempo (Traces)
```
Tempo Distributor
├── OTLP Receiver
├── Load Balancing
└── Trace Routing

Tempo Ingester
├── Trace Storage
├── Compaction
└── Retention

Tempo Querier
├── Trace Search
├── Service Graph Generation
└── API Endpoints
```

**Key Features**:
- Distributed trace storage
- Trace search and filtering
- Service dependency graphs
- Integration with metrics (span metrics)

#### Loki (Logs)
```
Loki Distributor
├── OTLP Log Receiver
├── Log Parsing
└── Load Balancing

Loki Ingester
├── Log Storage (TSDB)
├── Indexing
└── Compaction

Loki Querier
├── LogQL Engine
├── Label Filtering
└── Aggregation
```

**Key Features**:
- Log aggregation with metadata
- Label-based indexing
- LogQL query language
- Trace correlation

#### Pyroscope (Profiles)
```
Pyroscope Server
├── Profile Ingestion
├── Storage Engine
├── Query Engine
└── Web UI (Flame Graphs)
```

**Key Features**:
- Continuous profiling
- Flame graph visualization
- Profile querying and filtering
- Performance analysis

## 🔄 Data Flow Architecture

### Telemetry Ingestion Flow

```
Application
    │
    ▼
OpenTelemetry SDK
├── Creates spans (traces)
├── Records metrics
├── Logs structured events
└── Captures performance profiles
    │
    ▼
OTLP Protocol
├── gRPC transport (port 4317)
├── HTTP transport (port 4318)
└── Protocol buffers encoding
    │
    ▼
OpenTelemetry Collector
├── Receives OTLP data
├── Applies batch processing
├── Routes by telemetry type
└── Exports to backends
    │
    ▼
Backend Storage
├── Prometheus ← Metrics
├── Tempo ← Traces
├── Loki ← Logs
└── Pyroscope ← Profiles
```

### Query and Visualization Flow

```
User Request (Browser/API)
    │
    ▼
Grafana Server
├── Authentication
├── Dashboard rendering
└── Query coordination
    │
    ▼
Data Source Queries
├── PromQL → Prometheus
├── TraceQL → Tempo
├── LogQL → Loki
└── Profile queries → Pyroscope
    │
    ▼
Backend Responses
├── Metrics data
├── Trace data
├── Log data
└── Profile data
    │
    ▼
Grafana Processing
├── Data transformation
├── Visualization rendering
└── Dashboard composition
    │
    ▼
User Interface
└── Interactive dashboards
```

## 🌐 Network Architecture

### Internal Networking
- **Localhost Communication**: All services communicate via 127.0.0.1
- **Fixed Ports**: Pre-defined port assignments for service discovery
- **No Service Mesh**: Direct service-to-service communication
- **Health Checks**: HTTP-based readiness probes

### External Interfaces
```
Port Mappings:
├── 3000 → Grafana Web UI
├── 4040 → Pyroscope Web UI
├── 4317 → OTLP gRPC Ingestion
├── 4318 → OTLP HTTP Ingestion
├── 9090 → Prometheus API
├── 3100 → Loki API
└── 3200 → Tempo API
```

### Security Considerations
- **No Authentication**: Open access to all endpoints
- **No Encryption**: Plain HTTP communication
- **Localhost Only**: No external network exposure by default
- **Development Focus**: Security not prioritized for demo purposes

## 💾 Storage Architecture

### Data Persistence Strategy
- **Filesystem Storage**: All data stored in `/data` directory
- **Service-Specific Directories**:
  - `/data/grafana`: Dashboards, datasources, users
  - `/data/prometheus`: Time-series metrics
  - `/data/loki`: Log data and indices
  - `/data/tempo`: Trace data and indices
  - `/data/pyroscope`: Profiling data

### Storage Characteristics

#### Prometheus Storage
- **Format**: Custom time-series database
- **Retention**: Configurable (default: 15 days)
- **Compression**: Built-in compression
- **Performance**: Optimized for time-range queries

#### Tempo Storage
- **Format**: Parquet files in object storage simulation
- **Indexing**: Trace ID and service-based indexing
- **Retention**: Configurable retention policies
- **Search**: Full-text search capabilities

#### Loki Storage
- **Format**: TSDB-based log storage
- **Indexing**: Label-based indexing
- **Compression**: LZ4 compression
- **Query**: LogQL for complex queries

#### Pyroscope Storage
- **Format**: Custom profiling database
- **Indexing**: Application and function-based indexing
- **Compression**: Efficient storage of profiling data
- **Visualization**: Flame graph generation

## ⚙️ Configuration Architecture

### Configuration Sources
1. **Container Image**: Baked-in default configurations
2. **Environment Variables**: Runtime overrides
3. **Volume Mounts**: External configuration files
4. **Dynamic Updates**: Limited runtime reconfiguration

### Configuration Hierarchy
```
Environment Variables (highest priority)
    │
    ▼
Volume-mounted config files
    │
    ▼
Container default configurations (lowest priority)
```

### Key Configuration Areas
- **Service Ports**: Fixed port assignments
- **Data Paths**: Configurable storage locations
- **Resource Limits**: No limits (development focus)
- **Authentication**: Minimal (admin/admin)
- **External Integration**: OTLP export capability

## 🔧 Operational Architecture

### Startup Sequence
```
Container Start
    │
    ▼
Parallel Service Startup
├── Grafana
├── Loki
├── OTLP Collector
├── Prometheus
├── Tempo
└── Pyroscope
    │
    ▼
Health Check Loop
├── Check all services ready
├── Report startup times
└── Create /tmp/ready flag
    │
    ▼
Services Available
└── Accept external connections
```

### Health Monitoring
- **Service Health**: Individual `/ready` endpoints
- **Dependency Checks**: Inter-service connectivity
- **Startup Coordination**: All services must be ready
- **Failure Handling**: Container exits on critical failures

### Logging Architecture
- **Service Logs**: Individual service logging
- **Container Logs**: Docker logging integration
- **Debug Mode**: Optional verbose logging
- **Log Correlation**: Limited cross-service correlation

## 🚀 Scalability Considerations

### Current Limitations
- **Single Container**: No horizontal scaling
- **Resource Contention**: Shared CPU/memory
- **Storage Limits**: Filesystem constraints
- **Network Limits**: Localhost-only communication

### Scaling Strategies (For Production)
- **Service Separation**: Individual containers per service
- **Load Balancing**: Multiple instances behind load balancers
- **Storage Scaling**: Distributed storage systems
- **Service Mesh**: Inter-service communication layer

## 🔒 Security Architecture

### Current Security Posture
- **Development Focus**: Security not prioritized
- **No Authentication**: Open access to all services
- **No Encryption**: Plain HTTP communication
- **Default Credentials**: Well-known admin/admin

### Security Gaps
- **Network Security**: No firewall rules
- **Access Control**: No RBAC or authorization
- **Data Protection**: No encryption at rest
- **Audit Logging**: Limited security event logging

## 📊 Performance Characteristics

### Resource Usage
- **Memory**: ~2-4GB baseline, scales with data volume
- **CPU**: Low baseline, spikes during queries
- **Disk I/O**: High during data ingestion and compaction
- **Network**: Moderate internal traffic

### Performance Bottlenecks
- **Single Container**: Resource contention
- **Filesystem Storage**: I/O limitations
- **No Caching**: Cold starts for queries
- **Synchronous Processing**: Blocking operations

## 🔄 Extension Points

### Configuration Extensions
- **Environment Variables**: Runtime customization
- **Volume Mounts**: External configuration
- **Custom Images**: Modified container builds

### Integration Extensions
- **External OTLP Export**: Send data to external systems
- **Grafana Plugins**: Additional visualization capabilities
- **Custom Dashboards**: Extended monitoring views

### Development Extensions
- **Custom Examples**: Additional language examples
- **Testing Frameworks**: Extended test coverage
- **CI/CD Integration**: Automated deployment pipelines

---

This architecture document provides the foundation for understanding the LGTM Stack's design. For implementation details, see the [Components](components/index.md) documentation. For production deployment considerations, refer to the [Production Readiness](production/readiness.md) guide. 🏗️