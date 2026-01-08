# 🏗️ LGTM Stack Documentation

<div style="text-align: center; margin: 2rem 0;">
  <img src="assets/images/logo.svg" alt="LGTM Stack Logo" style="height: 120px; margin-bottom: 1rem;">
  <p style="font-size: 1.2em; color: #666; margin-bottom: 2rem;">
    Complete observability stack with Grafana, OpenTelemetry, Loki, Tempo, Prometheus & Pyroscope
  </p>
</div>

## 🚀 What is LGTM Stack?

The **LGTM Stack** is a comprehensive, production-ready observability platform that combines the best open-source monitoring tools into a unified, easy-to-deploy solution. Built with Docker and designed for modern microservices architectures.

!!! success "Key Features"
    - **📊 Complete Observability**: Metrics, logs, traces, and profiling in one stack
    - **🐳 Docker-Native**: Easy deployment with Docker Compose
    - **⚡ Production-Ready**: Configured for high availability and performance
    - **🔧 Extensible**: Customizable dashboards, alerts, and integrations
    - **📚 Well-Documented**: Comprehensive guides and examples

## 🏁 Quick Start

Get your observability stack running in minutes:

1. **Clone the repository**
    ```bash
    git clone https://github.com/Hardik-Sankhla/LGTM_Microservices_Observability_Stack.git
    cd LGTM_Microservices_Observability_Stack
    ```

2. **Start the stack**
    ```bash
    docker-compose up -d
    ```

3. **Access your dashboards**
    - **Grafana**: http://localhost:3000 (admin/admin)
    - **Prometheus**: http://localhost:9090
    - **Loki**: http://localhost:3100
    - **Tempo**: http://localhost:3200

!!! tip "Need Help?"
    Check out our [Quick Start Guide](quick-start.md) for detailed instructions.

## 📋 What's Included

| Component | Purpose | Port | Access |
|-----------|---------|------|--------|
| **Grafana** | Visualization & Dashboards | 3000 | [localhost:3000](http://localhost:3000) |
| **Prometheus** | Metrics Collection | 9090 | [localhost:9090](http://localhost:9090) |
| **Loki** | Log Aggregation | 3100 | [localhost:3100](http://localhost:3100) |
| **Tempo** | Distributed Tracing | 3200 | [localhost:3200](http://localhost:3200) |
| **Pyroscope** | Continuous Profiling | 4040 | [localhost:4040](http://localhost:4040) |
| **OpenTelemetry Collector** | Telemetry Pipeline | 4317/4318 | gRPC/HTTP |

## 📖 Documentation Sections

<div class="grid cards" markdown>

-   :material-rocket-launch:{ .lg .middle } __Getting Started__

    ---

    New to LGTM Stack? Start here with installation and basic concepts.

    [:octicons-arrow-right-24: Get Started](getting-started/overview.md)

-   :material-cog:{ .lg .middle } __Configuration__

    ---

    Learn how to configure and customize your observability stack.

    [:octicons-arrow-right-24: Configure](configuration/index.md)

-   :material-flask:{ .lg .middle } __Examples__

    ---

    See LGTM Stack in action with real-world examples and tutorials.

    [:octicons-arrow-right-24: View Examples](examples/index.md)

-   :material-server:{ .lg .middle } __Production__

    ---

    Deploy LGTM Stack in production with high availability and security.

    [:octicons-arrow-right-24: Production Guide](production/deployment-guide.md)

</div>

## 🤝 Contributing

We welcome contributions! Whether you're fixing bugs, adding features, or improving documentation:

- 📖 [Contributing Guide](contributing/index.md)
- 🐛 [Report Issues](https://github.com/Hardik-Sankhla/LGTM_Microservices_Observability_Stack/issues)
- 💬 [Discussions](https://github.com/Hardik-Sankhla/LGTM_Microservices_Observability_Stack/discussions)

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](https://github.com/Hardik-Sankhla/LGTM_Microservices_Observability_Stack/blob/main/LICENSE) file for details.

---

<div style="text-align: center; margin-top: 3rem; padding-top: 2rem; border-top: 1px solid #e0e0e0;">
  <p style="color: #666;">
    Built with ❤️ by <a href="https://github.com/Hardik-Sankhla" style="color: #666;">Hardik Sankhla</a> |
    <a href="https://github.com/Hardik-Sankhla/LGTM_Microservices_Observability_Stack" style="color: #666;">View on GitHub</a>
  </p>
</div>

## Project layout

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.
