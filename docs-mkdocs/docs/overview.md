# LGTM Stack Overview

## What is LGTM Stack?

The **Grafana Docker OTel LGTM Stack** is a comprehensive, self-contained observability platform packaged as a single Docker container. It provides a complete telemetry pipeline for collecting, processing, storing, and visualizing application metrics, logs, traces, and profiles.

"LGTM" stands for **L**ogs, **G**rafana, **T**empo, and **M**etrics - representing the core components of the stack.

## 🎯 Purpose & Use Cases

### Primary Purpose
- **OpenTelemetry Learning & Experimentation**: Quick setup for developers learning observability concepts
- **Development & Testing**: Local observability backend for application development
- **Demo & Proof-of-Concepts**: Instant observability stack for presentations and evaluations
- **Tool Evaluation**: Easy comparison and testing of observability tools

### Target Audience
- **Application Developers**: Learning OpenTelemetry instrumentation
- **DevOps Engineers**: Evaluating observability solutions
- **Platform Teams**: Building observability platforms
- **Educators**: Teaching observability concepts
- **Solution Architects**: Designing observability architectures

## 🏗️ Architecture Overview

```
┌─────────────────┐    OTLP/HTTP (4318)    ┌─────────────────────┐
│   Application   │ ────────────────────► │ OpenTelemetry       │
│   (Instrumented)│    OTLP/gRPC (4317)    │ Collector           │
└─────────────────┘                        └─────────────────────┘
                                                │
                                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Internal Data Flow                          │
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │  Prometheus │  │    Tempo    │  │    Loki    │  │ Pyroscope   │ │
│  │  (Metrics)  │  │  (Traces)   │  │   (Logs)   │  │ (Profiles)  │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │
│         │              │              │              │              │
└─────────┼──────────────┼──────────────┼──────────────┼──────────────┘
          │              │              │              │
          └──────────────┼──────────────┼──────────────┼──────────────┘
                         │              │              │
                    ┌─────────────┐     │              │
                    │   Grafana   │ ◄───┼──────────────┼──────────────┘
                    │ (Dashboards)│     │              │
                    └─────────────┘     └──────────────┘
```

## 🧩 Core Components

### OpenTelemetry Collector
- **Role**: Central ingestion and routing hub
- **Protocols**: OTLP (gRPC/HTTP), Prometheus, Jaeger
- **Features**: Data processing, batching, filtering, routing

### Grafana
- **Role**: Visualization and exploration platform
- **Features**: Dashboards, alerting, data source integration
- **Port**: 3000 (default credentials: admin/admin)

### Prometheus
- **Role**: Metrics collection and storage
- **Features**: Time-series database, PromQL query language
- **Port**: 9090

### Tempo
- **Role**: Distributed tracing backend
- **Features**: Trace storage, search, service graphs
- **Port**: 3200

### Loki
- **Role**: Log aggregation system
- **Features**: Log storage, LogQL queries, trace correlation
- **Port**: 3100

### Pyroscope
- **Role**: Continuous profiling platform
- **Features**: Flame graphs, performance analysis
- **Port**: 4040

## 🚀 Key Features

### Zero-Configuration Setup
- Single command startup
- Pre-configured data sources
- Built-in dashboards
- Automatic service discovery

### Multi-Language Support
- Go, Java, Node.js, Python, .NET examples
- Auto-instrumentation libraries
- Language-specific best practices

### Production-Grade Backends
- Industry-standard observability tools
- Enterprise features enabled
- Scalable storage backends

### Developer Experience
- Cross-platform scripts (Linux/macOS/Windows)
- Health checks and startup coordination
- Debug logging options
- Volume persistence support

## 📊 Telemetry Pipeline

### Data Ingestion
1. Applications send telemetry via **OTLP** protocol
2. Collector receives data on ports **4317** (gRPC) / **4318** (HTTP)
3. Data is processed and routed to appropriate backends

### Data Storage
- **Metrics** → Prometheus (time-series database)
- **Traces** → Tempo (distributed tracing)
- **Logs** → Loki (log aggregation)
- **Profiles** → Pyroscope (continuous profiling)

### Data Visualization
- **Grafana** provides unified view across all telemetry types
- Pre-built dashboards for each data type
- Custom dashboard creation
- Alerting and notification capabilities

## 🔄 Data Flow Example

```
Application Request
    │
    ▼
OpenTelemetry SDK
- Creates spans
- Records metrics
- Logs events
- Captures profiles
    │
    ▼
OTLP Protocol
- gRPC/HTTP transport
- Binary/JSON encoding
    │
    ▼
OpenTelemetry Collector
- Receives OTLP data
- Applies processing
- Routes to backends
    │
    ▼
Backend Storage
- Prometheus: Metrics
- Tempo: Traces
- Loki: Logs
- Pyroscope: Profiles
    │
    ▼
Grafana Visualization
- Unified dashboards
- Cross-telemetry correlation
- Alerting & notifications
```

## ⚠️ Important Limitations

### Not Production-Ready
This stack is explicitly designed for **development, demo, and testing** environments. Key limitations include:

- **Single Container**: All services run in one container (no isolation)
- **No High Availability**: Single replicas, no redundancy
- **Filesystem Storage**: No persistent, scalable storage
- **No Security**: No authentication, encryption, or access control
- **Resource Limits**: No CPU/memory constraints
- **No Monitoring**: No observability of the observability stack

### Production Recommendation
> For production use, consider [Grafana Cloud Application Observability](https://grafana.com/products/cloud/application-observability/) or deploy individual components as separate services.

## 🛠️ Quick Start Commands

### Linux/macOS
```bash
# Clone repository
git clone https://github.com/grafana/docker-otel-lgtm.git
cd docker-otel-lgtm

# Run the stack
./run-lgtm.sh

# Run example application
./run-example.sh

# Generate traffic
./generate-traffic.sh
```

### Windows (PowerShell)
```powershell
# Clone repository
git clone https://github.com/grafana/docker-otel-lgtm.git
cd docker-otel-lgtm

# Run the stack
.\run-lgtm.ps1

# Run example application
.\run-example.cmd

# Generate traffic
.\generate-traffic.cmd
```

### Using Docker Directly
```bash
# Pull and run
docker run -p 3000:3000 -p 4317:4317 -p 4318:4318 \
  grafana/otel-lgtm:latest
```

## 🌟 What Makes LGTM Special

### Developer-Centric Design
- **Fast Iteration**: Sub-second startup times
- **Local Development**: No cloud dependencies
- **Language Agnostic**: Works with any OpenTelemetry-supported language
- **Educational**: Clear examples and documentation

### Complete Observability Coverage
- **Four Pillars**: Metrics, logs, traces, profiles
- **Correlation**: Links between all telemetry types
- **Rich Context**: Service graphs, flame graphs, trace views
- **Real-Time**: Live data updates and dashboards

### Community & Ecosystem
- **Open Source**: Apache 2.0 licensed
- **Grafana Labs**: Backed by observability experts
- **Active Community**: Regular updates and improvements
- **Industry Standards**: Uses CNCF and OpenTelemetry projects

## 📈 Evolution & Roadmap

### Current Status (v0.11.16)
- Stable, production-quality components
- Comprehensive example applications
- Automated testing and releases
- Multi-architecture support (AMD64/ARM64)

### Future Enhancements
- Enhanced security features
- Additional language examples
- Performance optimizations
- Extended integration options

## 🤝 Community & Support

### Getting Help
- **Documentation**: Comprehensive guides and tutorials
- **GitHub Issues**: Bug reports and feature requests
- **GitHub Discussions**: Community questions and discussions
- **Grafana Community**: Broader observability community

### Contributing
- **Open Source**: Welcome contributions from the community
- **Development**: Local development setup provided
- **Testing**: Automated testing with OATS framework
- **Code Quality**: Linting, formatting, and security scanning

---

*Ready to get started? Check out the [Quick Start Guide](quick-start.md) to launch your first observability stack!* 🚀