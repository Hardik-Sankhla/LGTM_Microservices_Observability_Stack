# LGTM Stack - Observability Platform

A comprehensive observability stack built with Docker, featuring Grafana, OpenTelemetry, Loki, Tempo, Prometheus, and Pyroscope for complete application monitoring and observability.

## 🏗️ What's Included

This project provides a complete observability platform with:

- **Grafana** - Visualization and dashboards
- **OpenTelemetry Collector** - Telemetry data collection and processing
- **Prometheus** - Metrics collection and storage
- **Loki** - Log aggregation
- **Tempo** - Distributed tracing
- **Pyroscope** - Continuous profiling

## 🚀 Quick Start

### Prerequisites
- Docker and Docker Compose
- Git

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/Hardik-Sankhla/Hardik-Sankhla-LGTM_Microservices_Observability_Stack.git
   cd Hardik-Sankhla-LGTM_Microservices_Observability_Stack
   ```

2. **Start the stack:**
   ```bash
   docker-compose up -d
   ```

3. **Access the services:**
   - **Grafana**: http://localhost:3000 (admin/admin)
   - **Prometheus**: http://localhost:9090
   - **Loki**: http://localhost:3100
   - **Tempo**: http://localhost:3200
   - **Pyroscope**: http://localhost:4040

## 📚 Documentation

Complete documentation is available at: [https://hardik-sankhla.github.io/Hardik-Sankhla-LGTM_Microservices_Observability_Stack/](https://hardik-sankhla.github.io/Hardik-Sankhla-LGTM_Microservices_Observability_Stack/)

The documentation includes:
- Detailed setup instructions
- Configuration guides
- Architecture overview
- Examples and tutorials
- Production deployment guides

## 🏛️ Architecture

```
┌─────────────────┐    ┌─────────────────┐
│   Application   │───▶│ OpenTelemetry   │
│                 │    │   Collector     │
└─────────────────┘    └─────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────┐
│                 LGTM Stack                      │
│                                                 │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ │
│  │ Prometheus  │ │    Loki     │ │   Tempo     │ │
│  │  Metrics    │ │    Logs     │ │  Traces     │ │
│  └─────────────┘ └─────────────┘ └─────────────┘ │
│                                                 │
│             ┌─────────────────┐                 │
│             │    Grafana      │                 │
│             │ Visualization   │                 │
│             └─────────────────┘                 │
│                                                 │
│             ┌─────────────────┐                 │
│             │   Pyroscope     │                 │
│             │   Profiling     │                 │
│             └─────────────────┘                 │
└─────────────────────────────────────────────────┘
```

## 🛠️ Development

### Local Development Setup

1. **Install dependencies:**
   ```bash
   pip install -r docs-mkdocs/requirements.txt
   ```

2. **Serve documentation locally:**
   ```bash
   cd docs-mkdocs
   mkdocs serve
   ```

3. **Build for production:**
   ```bash
   cd docs-mkdocs
   mkdocs build
   ```

## 📦 Project Structure

```
Hardik-Sankhla-LGTM_Microservices_Observability_Stack/
├── docker/                 # Docker configurations
├── docs-mkdocs/           # Professional documentation (MkDocs)
├── examples/              # Usage examples
├── k8s/                   # Kubernetes manifests
├── docker-compose.yml     # Main stack configuration
├── build-lgtm.*          # Build scripts
├── run-lgtm.*            # Run scripts
└── README.md             # This file
```

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

## 📄 License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

## 👤 Author

**Hardik Sankhla**
- GitHub: [@Hardik-Sankhla](https://github.com/Hardik-Sankhla)

---

**Built with ❤️ by Hardik Sankhla** | © 2026 All Rights Reserved

- Blog posts:
  - [_An OpenTelemetry backend in a Docker image: Introducing grafana/otel-lgtm_][otel-lgtm]
  - [_Observability in under 5 seconds: Reflecting on a year of grafana/otel-lgtm_][otel-lgtm-one-year]

## Get the Docker image

The Docker image is available on [Docker Hub][docker-hub].

```sh
docker pull grafana/otel-lgtm:latest
```

## Run the Docker image

### Linux/Unix

```sh
./run-lgtm.sh
```

### Windows (PowerShell)

```powershell
./run-lgtm
```

### Linux/Unix Using mise

You can also use [mise][mise] to run the Docker image:

```sh
mise run lgtm
```

## Configuration

### Enable logging

You can enable logging in the .env file for troubleshooting:

| Environment Variable     | Enables Logging in:     |
|--------------------------|-------------------------|
| `ENABLE_LOGS_GRAFANA`    | Grafana                 |
| `ENABLE_LOGS_LOKI`       | Loki                    |
| `ENABLE_LOGS_PROMETHEUS` | Prometheus              |
| `ENABLE_LOGS_TEMPO`      | Tempo                   |
| `ENABLE_LOGS_PYROSCOPE`  | Pyroscope               |
| `ENABLE_LOGS_OTELCOL`    | OpenTelemetry Collector |
| `ENABLE_LOGS_ALL`        | All of the above        |

This has nothing to do with any application logs, which are collected by OpenTelemetry.

### Send data to vendors

In addition to the built-in observability tools, you can also send data to vendors.
That way, you can easily try and switch between different backends.

If the [`OTEL_EXPORTER_OTLP_ENDPOINT`][otlp-endpoint]
variable is set, the OpenTelemetry Collector will send data (logs, metrics, and traces)
to the specified endpoint using "OTLP/HTTP".

In addition, you can provide [`OTEL_EXPORTER_OTLP_HEADERS`][otlp-headers],
for example, to authenticate with the backend.

#### Send data to Grafana Cloud

You can find the values for the environment variables in your [Grafana Cloud account][otel-setup].

### Persist data across container instantiation

The various components in the repository are configured to write their data to the `/data`
directory. If you need to persist data across containers being created and destroyed,
you can mount a volume to the `/data` directory. Note that this image is intended for
development, demo, and testing environments and persisting data to an external volume
doesn't change that. However, this feature could be useful in certain cases for
some users even in testing situations.

### Pre-install Grafana plugins

You can pre-install Grafana plugins by adding them to the `GF_PLUGINS_PREINSTALL` environment variable.
See the [Grafana documentation][grafana-preinstall-plugins] for more information.

## Run lgtm in Kubernetes

```sh
# Create k8s resources
kubectl apply -f k8s/lgtm.yaml

# Configure port forwarding
kubectl port-forward service/lgtm 3000:3000 4040:4040 4317:4317 4318:4318 9090:9090

# Using mise
mise k8s-apply
mise k8s-port-forward
```

## Send OpenTelemetry Data

There's no need to configure anything: the Docker image works with OpenTelemetry's defaults.

```sh
# Not needed, but these are the defaults in OpenTelemetry
export OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
export OTEL_EXPORTER_OTLP_ENDPOINT=http://127.0.0.1:4318
```

## View Grafana

Navigate to <http://127.0.0.1:3000> and log in with the default built-in user `admin` and password `admin`.

## Build the Docker image from scratch

```sh
cd docker/
docker build . -t grafana/otel-lgtm

# Using mise
mise build-lgtm
```

> [!TIP]
> If you built your image locally, you can use the `run-lgtm` scripts with
> the parameters `latest true` to run your local image (or `mise run local-lgtm`).

## Build and run the example app

> [!TIP]
> You can run everything together using [mise][mise] with `mise run all`.

### Run

Run the example REST service:

#### Unix/Linux

```sh
./run-example.sh
```

#### Windows (PowerShell)

```powershell
./run-example
```

#### Unix/Linux Using mise

```sh
mise run example
```

### Generate traffic

#### Unix/Linux

```sh
./generate-traffic.sh
```

#### Windows (PowerShell)

```powershell
./generate-traffic
```

#### Unix/Linux Using mise

```sh
mise run generate-traffic
```

> [!TIP]
> You can use [OTel Checker][otel-checker] to check if the instrumentation is correct.

## Run example apps in different languages

The example apps are in the [`examples/`][examples] directory.
Each example has a `run.sh` or `run.cmd` script to start the app.

Every example implements a rolldice service, which returns a random number between 1 and 6.

Each example uses a different application port
(to be able to run all applications at the same time).

| Example | Service URL                           |
|---------|---------------------------------------|
| Java    | `curl http://127.0.0.1:8080/rolldice` |
| Go      | `curl http://127.0.0.1:8081/rolldice` |
| Python  | `curl http://127.0.0.1:8082/rolldice` |
| .NET    | `curl http://127.0.0.1:8083/rolldice` |
| Node.js | `curl http://127.0.0.1:8084/rolldice` |

## Verifying Container Image Signatures

The container images that are published are signed using [cosign][cosign]. You
can verify the signatures using a command similar to the following example:

```sh
VERSION="0.11.16"
IMAGE="docker.io/grafana/otel-lgtm:${VERSION}"
IDENTITY="https://github.com/grafana/docker-otel-lgtm/.github/workflows/release.yml@refs/tags/v${VERSION}"
OIDC_ISSUER="https://token.actions.githubusercontent.com"

cosign verify ${IMAGE} --certificate-identity ${IDENTITY} --certificate-oidc-issuer ${OIDC_ISSUER}
```

It is also possible to verify the signatures of images from our continuous integration
that are published to the [GitHub Container Registry][ghcr]. For example for the `main` branch:

```sh
VERSION="main"
IMAGE="ghcr.io/grafana/docker-otel-lgtm:${VERSION}"
WORKFLOW="ghcr-image-build-and-publish.yml"
IDENTITY="https://github.com/grafana/docker-otel-lgtm/.github/workflows/${WORKFLOW}@refs/heads/${VERSION}"
OIDC_ISSUER="https://token.actions.githubusercontent.com"

cosign verify ${IMAGE} --certificate-identity ${IDENTITY} --certificate-oidc-issuer ${OIDC_ISSUER}
```

## Related Work

- [Metrics, Logs, Traces and Profiles in Grafana][mltp]
- [OpenTelemetry Acceptance Tests (OATs)][oats]

<!-- editorconfig-checker-disable -->
<!-- markdownlint-disable MD013 -->

[app-o11y]: https://grafana.com/products/cloud/application-observability/
[cosign]: https://github.com/sigstore/cosign "Cosign on GitHub"
[docker-hub]: https://hub.docker.com/r/grafana/otel-lgtm
[docker-latest]: https://img.shields.io/docker/v/grafana/otel-lgtm?logo=docker&label=latest&color=blue
[docker-pulls]: https://img.shields.io/docker/pulls/grafana/otel-lgtm?logo=docker&label=pulls
[examples]: https://github.com/grafana/docker-otel-lgtm/tree/main/examples
[ghcr]: https://github.com/grafana/docker-otel-lgtm/pkgs/container/docker-otel-lgtm
[grafana-preinstall-plugins]: https://grafana.com/docs/grafana/latest/setup-grafana/configure-docker/#install-plugins-in-the-docker-container
[mise]: https://github.com/jdx/mise
[mltp]: https://github.com/grafana/intro-to-mltp
[otel-checker]: https://github.com/grafana/otel-checker/
[otel-lgtm]: https://grafana.com/blog/2024/03/13/an-opentelemetry-backend-in-a-docker-image-introducing-grafana-otel-lgtm
[otel-lgtm-one-year]: https://grafana.com/blog/2025/07/08/observability-in-under-5-seconds-reflecting-on-a-year-of-grafana-otel-lgtm/
[otel-setup]: https://grafana.com/docs/grafana-cloud/send-data/otlp/send-data-otlp/#manual-opentelemetry-setup-for-advanced-users
[otlp-endpoint]: https://opentelemetry.io/docs/languages/sdk-configuration/otlp-exporter/#otel_exporter_otlp_endpoint
[otlp-headers]: https://opentelemetry.io/docs/languages/sdk-configuration/otlp-exporter/#otel_exporter_otlp_headers
[oats]: https://github.com/grafana/oats
