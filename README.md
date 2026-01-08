# LGTM Stack - Observability Platform

<div align="center">

![LGTM Stack](docs-mkdocs/docs/assets/images/logo.svg)

**A comprehensive observability stack for modern microservices**

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Documentation](https://img.shields.io/badge/docs-mkdocs-blue)](https://hardik-sankhla.github.io/Hardik-Sankhla-LGTM_Microservices_Observability_Stack/)

</div>

---

## 📖 Overview

The **LGTM Stack** is a production-ready observability platform that combines the best open-source monitoring tools into a unified, Docker-based solution. Built for modern microservices architectures, it provides complete visibility into your applications through metrics, logs, traces, and profiling.

### 🎯 Key Features

- **📊 Complete Observability**: Unified platform for metrics, logs, traces, and profiling
- **🐳 Docker-Native**: Easy deployment with Docker Compose
- **⚡ Production-Ready**: Configured for high availability and performance
- **🔧 Highly Configurable**: Customizable dashboards, alerts, and integrations
- **📚 Comprehensive Documentation**: Detailed guides and examples
- **🚀 Quick Start**: Up and running in minutes

---

## 🏗️ Architecture

The LGTM Stack integrates six powerful observability tools:

| Component | Purpose | Port | Status |
|-----------|---------|------|--------|
| **Grafana** | Visualization & Dashboards | 3000 | ✅ Ready |
| **OpenTelemetry Collector** | Telemetry Pipeline | 4317/4318 | ✅ Ready |
| **Prometheus** | Metrics Collection | 9090 | ✅ Ready |
| **Loki** | Log Aggregation | 3100 | ✅ Ready |
| **Tempo** | Distributed Tracing | 3200 | ✅ Ready |
| **Pyroscope** | Continuous Profiling | 4040 | ✅ Ready |

### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Your Application                          │
│              (Instrumented with OpenTelemetry)              │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│           OpenTelemetry Collector (OTLP)                    │
│         (Receives, Processes, Exports Telemetry)            │
└─────────────────┬───────────────┬───────────────┬───────────┘
                  │               │               │
         ┌────────┘               │               └────────┐
         ▼                        ▼                        ▼
┌─────────────┐         ┌─────────────┐         ┌─────────────┐
│ Prometheus  │         │    Loki     │         │   Tempo     │
│  (Metrics)  │         │   (Logs)    │         │  (Traces)   │
└──────┬──────┘         └──────┬──────┘         └──────┬──────┘
       │                       │                        │
       │               ┌───────┴────────┐               │
       │               │                │               │
       ▼               ▼                ▼               ▼
┌──────────────────────────────────────────────────────────────┐
│                        Grafana                                │
│              (Unified Visualization Layer)                    │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
                     ┌─────────────────┐
                     │   Pyroscope     │
                     │   (Profiling)   │
                     └─────────────────┘
```

---

## 🚀 Quick Start

### Prerequisites

- **Docker** (20.10+) and **Docker Compose** (1.29+)
- **Git**
- **8GB RAM** minimum (16GB recommended)
- **10GB disk space**

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/Hardik-Sankhla/Hardik-Sankhla-LGTM_Microservices_Observability_Stack.git
   cd Hardik-Sankhla-LGTM_Microservices_Observability_Stack
   ```

2. **Start the stack**
   ```bash
   # Linux/Mac
   ./run-lgtm.sh
   
   # Windows PowerShell
   ./run-lgtm.ps1
   
   # Or using Docker Compose directly
   docker-compose up -d
   ```

3. **Access the services**
   
   | Service | URL | Default Credentials |
   |---------|-----|---------------------|
   | **Grafana** | [http://localhost:3000](http://localhost:3000) | admin / admin |
   | **Prometheus** | [http://localhost:9090](http://localhost:9090) | - |
   | **Loki** | [http://localhost:3100](http://localhost:3100) | - |
   | **Tempo** | [http://localhost:3200](http://localhost:3200) | - |
   | **Pyroscope** | [http://localhost:4040](http://localhost:4040) | - |
   | **OTLP gRPC** | `http://localhost:4317` | - |
   | **OTLP HTTP** | `http://localhost:4318` | - |

### Verify Installation

```bash
# Check if all services are running
docker-compose ps

# View logs
docker-compose logs -f
```

---

## 📚 Documentation

### 📖 Complete Documentation

**Visit our comprehensive documentation site**: [https://hardik-sankhla.github.io/Hardik-Sankhla-LGTM_Microservices_Observability_Stack/](https://hardik-sankhla.github.io/Hardik-Sankhla-LGTM_Microservices_Observability_Stack/)

This documentation is built with **MkDocs** and **Material for MkDocs**.

### 📋 Documentation Contents

#### 🎯 Getting Started
- **[Overview](docs-mkdocs/docs/overview.md)** - Introduction to the LGTM Stack
- **[Quick Start](docs-mkdocs/docs/quick-start.md)** - Get up and running in 5 minutes
- **[Installation](docs-mkdocs/docs/installation.md)** - Detailed installation instructions

#### 🏛️ Architecture & Components
- **[System Architecture](docs-mkdocs/docs/architecture.md)** - High-level system design
- **[Components Overview](docs-mkdocs/docs/components/index.md)** - Detailed component documentation
  - [OpenTelemetry Collector](docs-mkdocs/docs/components/otel-collector.md)
  - Grafana, Prometheus, Loki, Tempo, Pyroscope
- **[Data Flow](docs-mkdocs/docs/data-flow.md)** - How telemetry flows through the system

#### ⚙️ Configuration
- **[Configuration Guide](docs-mkdocs/docs/configuration/index.md)** - Configuration options
- **[Environment Variables](docs-mkdocs/docs/configuration/environment-variables.md)** - All available variables

#### 💻 Examples & Tutorials
- **[Language Examples](docs-mkdocs/docs/examples/index.md)** - Instrumented applications
  - Go, Java, Node.js, Python, .NET examples
  - Custom instrumentation patterns

#### 🔧 Operations & Maintenance
- **[Troubleshooting](docs-mkdocs/docs/operations/troubleshooting.md)** - Common issues and solutions
- Performance tuning and optimization
- Backup and recovery strategies

#### 🏭 Production Deployment
- **[Deployment Guide](docs-mkdocs/docs/production/deployment-guide.md)** - Production strategies
- **[Microservices Migration](docs-mkdocs/docs/production/microservices-migration.md)** - Migration guide
- **[Security Hardening](docs-mkdocs/docs/production/security-hardening.md)** - Security best practices

---

## 🔌 Sending Telemetry Data

### OpenTelemetry Integration

The stack works with OpenTelemetry's default configuration:

```bash
# Default OpenTelemetry endpoints (already configured)
export OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
```

### Instrumentation Examples

#### Go
```go
import "go.opentelemetry.io/otel"

// See examples/go/ for complete example
```

#### Python
```python
from opentelemetry import trace

# See examples/python/ for complete example
```

#### Java
```java
import io.opentelemetry.api.trace.Tracer;

// See examples/java/ for complete example
```

#### Node.js
```javascript
const { trace } = require('@opentelemetry/api');

// See examples/nodejs/ for complete example
```

#### .NET
```csharp
using OpenTelemetry.Trace;

// See examples/dotnet/ for complete example
```

For detailed examples, see the [`examples/`](examples/) directory.

---

## 🛠️ Development

### Local Development Setup

1. **Install documentation dependencies**
   ```bash
   cd docs-mkdocs
   pip install -r requirements.txt
   ```

2. **Serve documentation locally**
   ```bash
   cd docs-mkdocs
   mkdocs serve
   ```
   Visit: [http://localhost:8000](http://localhost:8000)

3. **Build documentation**
   ```bash
   cd docs-mkdocs
   mkdocs build
   ```

### Project Structure

```
Hardik-Sankhla-LGTM_Microservices_Observability_Stack/
├── docker/                    # Docker configurations
│   ├── Dockerfile            # Main Docker image
│   ├── grafana-*.json        # Grafana dashboards
│   ├── *-config.yaml         # Component configurations
│   └── run-*.sh              # Component startup scripts
├── docs-mkdocs/              # Professional documentation
│   ├── docs/                 # Markdown documentation
│   ├── mkdocs.yml            # MkDocs configuration
│   └── requirements.txt      # Python dependencies
├── examples/                 # Language-specific examples
│   ├── go/                   # Go example
│   ├── java/                 # Java example
│   ├── nodejs/               # Node.js example
│   ├── python/               # Python example
│   └── dotnet/               # .NET example
├── k8s/                      # Kubernetes manifests
│   └── lgtm.yaml            # K8s deployment
├── docker-compose.yml        # Main stack configuration
├── build-lgtm.*             # Build scripts
├── run-lgtm.*               # Run scripts
└── README.md                # This file
```

---

## ⚙️ Configuration

### Environment Variables

Configure the stack using environment variables in a `.env` file:

```bash
# Enable component logging
ENABLE_LOGS_GRAFANA=true
ENABLE_LOGS_LOKI=true
ENABLE_LOGS_PROMETHEUS=true
ENABLE_LOGS_TEMPO=true
ENABLE_LOGS_PYROSCOPE=true
ENABLE_LOGS_OTELCOL=true
ENABLE_LOGS_ALL=true

# External OTLP endpoint (optional)
OTEL_EXPORTER_OTLP_ENDPOINT=https://your-vendor.com
OTEL_EXPORTER_OTLP_HEADERS=Authorization=Bearer your-token
```

### Kubernetes Deployment

Deploy to Kubernetes:

```bash
# Apply Kubernetes manifests
kubectl apply -f k8s/lgtm.yaml

# Port forward services
kubectl port-forward service/lgtm 3000:3000 4040:4040 4317:4317 4318:4318 9090:9090
```

### Data Persistence

Mount a volume to `/data` for persistent storage:

```yaml
volumes:
  - ./data:/data
```

---

## 🤝 Contributing

Contributions are welcome! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

### How to Contribute

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/amazing-feature`
3. Commit your changes: `git commit -m 'Add amazing feature'`
4. Push to the branch: `git push origin feature/amazing-feature`
5. Open a Pull Request

---

## 📄 License

This project is licensed under the **Apache License 2.0** - see the [LICENSE](LICENSE) file for details.

---

## 👤 Author & Ownership

**Hardik Sankhla**

- 🌐 GitHub: [@Hardik-Sankhla](https://github.com/Hardik-Sankhla)
- 📧 Project Maintainer & Owner
- 💼 All Rights Reserved © 2026

---

## 🙏 Acknowledgments

This project leverages the excellent work of the OpenTelemetry and Grafana open-source communities. Special thanks to:

- **OpenTelemetry**: For the comprehensive observability framework
- **Grafana Labs**: For the visualization and monitoring tools
- **CNCF**: For fostering open-source observability standards

---

## 📞 Support

- **📖 Documentation**: [https://hardik-sankhla.github.io/Hardik-Sankhla-LGTM_Microservices_Observability_Stack/](https://hardik-sankhla.github.io/Hardik-Sankhla-LGTM_Microservices_Observability_Stack/)
- **🐛 Issues**: [GitHub Issues](https://github.com/Hardik-Sankhla/Hardik-Sankhla-LGTM_Microservices_Observability_Stack/issues)
- **💬 Discussions**: [GitHub Discussions](https://github.com/Hardik-Sankhla/Hardik-Sankhla-LGTM_Microservices_Observability_Stack/discussions)

---

## ⭐ Star History

If you find this project helpful, please consider giving it a star ⭐

---

<div align="center">

**Built with ❤️ by Hardik Sankhla**

© 2026 Hardik Sankhla. All Rights Reserved.

[Documentation](https://hardik-sankhla.github.io/Hardik-Sankhla-LGTM_Microservices_Observability_Stack/) • [GitHub](https://github.com/Hardik-Sankhla/Hardik-Sankhla-LGTM_Microservices_Observability_Stack) • [License](LICENSE)

</div>
