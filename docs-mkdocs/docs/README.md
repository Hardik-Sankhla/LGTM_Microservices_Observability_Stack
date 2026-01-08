# LGTM Stack Documentation

Welcome to the comprehensive documentation for the Grafana Docker OTel LGTM Stack. This documentation provides complete information about the project, from basic usage to advanced production deployment strategies.

## Documentation Structure

### 📖 Getting Started
- **[Overview](overview.md)** - Project introduction and purpose
- **[Quick Start](quick-start.md)** - Get up and running in 5 minutes
- **[Installation](installation.md)** - Detailed installation instructions

### 🏗️ Architecture & Design
- **[System Architecture](architecture.md)** - High-level system design
- **[Components](components/index.md)** - Detailed component documentation
  - [OpenTelemetry Collector](components/otel-collector.md)
  - [Grafana](components/grafana.md)
  - [Prometheus](components/prometheus.md)
  - [Tempo](components/tempo.md)
  - [Loki](components/loki.md)
  - [Pyroscope](components/pyroscope.md)
- **[Data Flow](data-flow.md)** - How telemetry data flows through the system

### ⚙️ Configuration & Customization
- **[Configuration Guide](configuration/index.md)** - Configuration options
- **[Environment Variables](configuration/environment-variables.md)** - All available environment variables
- **[Custom Dashboards](configuration/dashboards.md)** - Adding custom Grafana dashboards
- **[Plugins](configuration/plugins.md)** - Installing Grafana plugins

### 🚀 Examples & Tutorials
- **[Language Examples](examples/index.md)** - Instrumented applications by language
  - [Go Example](examples/go.md)
  - [Java Example](examples/java.md)
  - [Node.js Example](examples/nodejs.md)
  - [Python Example](examples/python.md)
  - [.NET Example](examples/dotnet.md)
- **[Advanced Examples](examples/advanced.md)** - Complex instrumentation scenarios

### 🔧 Operations & Maintenance
- **[Monitoring](operations/monitoring.md)** - Monitoring the monitoring stack
- **[Troubleshooting](operations/troubleshooting.md)** - Common issues and solutions
- **[Performance Tuning](operations/performance.md)** - Optimization and scaling
- **[Backup & Recovery](operations/backup-recovery.md)** - Data persistence strategies

### 🏭 Production Deployment
- **[Deployment Guide](production/deployment-guide.md)** - Production deployment strategies
- **[Microservices Migration](production/microservices-migration.md)** - Converting to microservices architecture
- **[Security Hardening](production/security-hardening.md)** - Production security measures
- **[Production Readiness](production/readiness.md)** - Production gap analysis
- **[Kubernetes Deployment](production/kubernetes.md)** - K8s manifests and Helm charts
- **[CI/CD Integration](production/cicd.md)** - Build and deployment pipelines

### 🔌 API & Integration
- **[API Reference](api/reference.md)** - Complete API documentation for all services
- **[Integration Guide](api/integration.md)** - Integrating with external systems
- **[Webhooks](api/webhooks.md)** - Event-driven integrations

### 🤝 Contributing & Development
- **[Contributing Guide](contributing/index.md)** - How to contribute
- **[Development Setup](contributing/development.md)** - Development environment
- **[Testing](contributing/testing.md)** - Testing strategies and frameworks
- **[Release Process](contributing/release.md)** - Versioning and releases

### 📚 Reference
- **[Glossary](reference/glossary.md)** - Terminology and definitions
- **[FAQs](reference/faq.md)** - Frequently asked questions
- **[Changelog](reference/changelog.md)** - Version history and changes
- **[Migration Guide](reference/migration.md)** - Upgrading between versions

## Project Information

- **Repository**: [grafana/docker-otel-lgtm](https://github.com/grafana/docker-otel-lgtm)
- **License**: Apache 2.0
- **Latest Version**: See [releases](https://github.com/grafana/docker-otel-lgtm/releases)
- **Documentation Version**: For LGTM Stack v0.11.16+

## Support

- **Issues**: [GitHub Issues](https://github.com/grafana/docker-otel-lgtm/issues)
- **Discussions**: [GitHub Discussions](https://github.com/grafana/docker-otel-lgtm/discussions)
- **Grafana Community**: [Grafana Community Forums](https://community.grafana.com/)

## Quick Links

- [🚀 Quick Start](quick-start.md)
- [📊 Grafana UI](http://localhost:3000) (when running)
- [📈 Prometheus](http://localhost:9090) (when running)
- [🔍 Tempo](http://localhost:3200) (when running)
- [📝 Loki](http://localhost:3100) (when running)
- [🔥 Pyroscope](http://localhost:4040) (when running)

---

*This documentation is maintained by the Grafana LGTM Stack community. Contributions welcome!* 📚✨