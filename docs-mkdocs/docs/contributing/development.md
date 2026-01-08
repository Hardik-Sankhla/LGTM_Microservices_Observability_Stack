# Development Guide

This guide provides information for developers who want to contribute to the LGTM Stack project, including development setup, coding standards, testing, and release processes.

## Table of Contents

- [Development Environment Setup](#development-environment-setup)
- [Project Structure](#project-structure)
- [Coding Standards](#coding-standards)
- [Testing Strategy](#testing-strategy)
- [Contributing Guidelines](#contributing-guidelines)
- [Release Process](#release-process)
- [CI/CD Pipeline](#cicd-pipeline)

## Development Environment Setup

### Prerequisites

- **Docker**: Version 20.10 or later
- **Docker Compose**: Version 2.0 or later
- **Git**: Version 2.30 or later
- **Python**: Version 3.9 or later (for development scripts)
- **Node.js**: Version 16 or later (for frontend development)
- **Go**: Version 1.19 or later (for Go examples)

### Clone the Repository

```bash
git clone https://github.com/grafana/docker-otel-lgtm.git
cd docker-otel-lgtm
```

### Development Environment

#### Using Docker Compose (Recommended)

```bash
# Start the development environment
docker-compose -f docker-compose.dev.yml up -d

# View logs
docker-compose -f docker-compose.dev.yml logs -f

# Stop the environment
docker-compose -f docker-compose.dev.yml down
```

#### Local Development Setup

```bash
# Install Python dependencies
pip install -r requirements-dev.txt

# Install Node.js dependencies
npm install

# Install Go dependencies
go mod download

# Start services individually for debugging
docker run -d --name prometheus -p 9090:9090 prom/prometheus
docker run -d --name grafana -p 3000:3000 grafana/grafana
# ... start other services
```

### IDE Setup

#### VS Code Configuration

```json
// .vscode/settings.json
{
  "docker.languageserver.diagnostics.deprecatedMaintainer": "ignore",
  "docker.languageserver.diagnostics.emptyContinuationLine": "ignore",
  "yaml.schemas": {
    "https://raw.githubusercontent.com/grafana/docker-otel-lgtm/main/otelcol-config.schema.json": [
      "otelcol-config.yaml"
    ]
  },
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
}
```

#### Recommended Extensions

- Docker
- YAML
- Go
- Python
- ESLint
- Prettier

## Project Structure

```
docker-otel-lgtm/
├── docker/                 # Docker build files
│   ├── Dockerfile          # Main application Dockerfile
│   └── docker-compose.yml  # Production compose file
├── examples/               # Language-specific examples
│   ├── go/
│   ├── java/
│   ├── nodejs/
│   ├── python/
│   └── dotnet/
├── k8s/                    # Kubernetes manifests
│   ├── base/              # Base manifests
│   ├── overlays/          # Environment-specific overlays
│   └── helm/              # Helm charts
├── scripts/                # Utility scripts
│   ├── build.sh           # Build script
│   ├── test.sh            # Test script
│   └── deploy.sh          # Deployment script
├── docs/                   # Documentation
├── tests/                  # Test files
│   ├── integration/       # Integration tests
│   └── unit/              # Unit tests
├── .github/                # GitHub configuration
│   ├── workflows/         # CI/CD workflows
│   └── ISSUE_TEMPLATE/    # Issue templates
├── docker-compose.yml      # Development compose file
├── otelcol-config.yaml     # OpenTelemetry Collector config
├── prometheus.yaml         # Prometheus configuration
├── tempo-config.yaml       # Tempo configuration
├── loki-config.yaml        # Loki configuration
├── pyroscope-config.yaml   # Pyroscope configuration
├── requirements.txt        # Python dependencies
├── package.json            # Node.js dependencies
├── go.mod                  # Go dependencies
└── README.md               # Project README
```

## Coding Standards

### Docker

#### Dockerfile Standards

```dockerfile
# Use specific base images with SHA256 hashes
FROM grafana/grafana:10.1.0@sha256:...

# Use multi-stage builds for smaller images
FROM golang:1.19-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN go build -o main .

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
CMD ["./main"]

# Use .dockerignore to exclude unnecessary files
# .dockerignore
node_modules
.git
*.md
```

#### Docker Compose Standards

```yaml
version: '3.8'

services:
  service-name:
    build:
      context: .
      dockerfile: docker/Dockerfile
    image: lgtm/service-name:${TAG:-latest}
    container_name: lgtm-service-name
    restart: unless-stopped
    environment:
      - SERVICE_NAME=service-name
    ports:
      - "${SERVICE_PORT:-8080}:8080"
    volumes:
      - ./config:/etc/service
    depends_on:
      - dependency-service
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    networks:
      - lgtm-network

networks:
  lgtm-network:
    driver: bridge
```

### Configuration Files

#### YAML Configuration

```yaml
# Use consistent indentation (2 spaces)
# Add comments for complex configurations
# Use anchors and aliases for reusable sections

global:
  scrape_interval: 15s  # Scrape targets every 15 seconds
  evaluation_interval: 15s  # Evaluate rules every 15 seconds

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'otel-collector'
    static_configs:
      - targets: ['otel-collector:8888']
```

### Go Code Standards

```go
package main

import (
    "context"
    "log"
    "time"

    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/trace"
)

// Use meaningful package names
// Follow standard Go naming conventions
// Add package comments

// main is the entry point for the application.
func main() {
    // Initialize tracer
    tracer := otel.Tracer("example-tracer")

    // Create context with timeout
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    // Start span
    ctx, span := tracer.Start(ctx, "main")
    defer span.End()

    // Handle errors properly
    if err := run(ctx); err != nil {
        log.Printf("Error running application: %v", err)
        span.RecordError(err)
        span.SetStatus(codes.Error, err.Error())
    }
}

func run(ctx context.Context) error {
    // Implementation here
    return nil
}
```

### Python Code Standards

```python
"""
Example Python application with OpenTelemetry instrumentation.
"""

import logging
from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def setup_tracing():
    """Set up OpenTelemetry tracing."""
    trace.set_tracer_provider(TracerProvider())
    tracer = trace.get_tracer(__name__)

    otlp_exporter = OTLPSpanExporter(
        endpoint="otel-collector:4317",
        insecure=True,
    )

    trace.get_tracer_provider().add_span_processor(
        BatchSpanProcessor(otlp_exporter)
    )

    return tracer

def main():
    """Main application entry point."""
    tracer = setup_tracing()

    with tracer.start_as_current_span("main") as span:
        span.set_attribute("service.name", "example-python-app")
        logger.info("Application started")

        try:
            # Application logic here
            process_data()
        except Exception as e:
            logger.error("Error processing data: %s", e)
            span.record_exception(e)
            raise

def process_data():
    """Process application data."""
    # Implementation here
    pass

if __name__ == "__main__":
    main()
```

## Testing Strategy

### Test Types

#### Unit Tests

```python
# tests/test_collector.py
import pytest
from unittest.mock import Mock, patch
from collector import TelemetryCollector

class TestTelemetryCollector:
    def setup_method(self):
        self.collector = TelemetryCollector()

    def test_process_metrics(self):
        """Test metrics processing."""
        metrics_data = {
            "name": "test_metric",
            "value": 42.0,
            "timestamp": 1640995200
        }

        result = self.collector.process_metrics(metrics_data)

        assert result["processed"] is True
        assert result["metric_name"] == "test_metric"

    @patch('collector.requests.post')
    def test_send_to_backend(self, mock_post):
        """Test sending data to backend."""
        mock_post.return_value.status_code = 200

        data = {"test": "data"}
        result = self.collector.send_to_backend(data)

        assert result is True
        mock_post.assert_called_once()
```

#### Integration Tests

```python
# tests/integration/test_end_to_end.py
import pytest
import requests
import time
from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter

class TestEndToEnd:
    def test_telemetry_flow(self):
        """Test complete telemetry flow from application to storage."""
        # Setup tracing
        tracer = self._setup_tracing()

        # Generate test telemetry
        with tracer.start_as_current_span("test-span") as span:
            span.set_attribute("test.attribute", "test.value")

        # Wait for processing
        time.sleep(5)

        # Verify data in Prometheus
        response = requests.get("http://prometheus:9090/api/v1/query",
                               params={"query": "up"})
        assert response.status_code == 200
        data = response.json()
        assert data["status"] == "success"

        # Verify data in Tempo
        # Add Tempo verification here

    def _setup_tracing(self):
        """Set up OpenTelemetry tracing for testing."""
        trace.set_tracer_provider(TracerProvider())
        tracer = trace.get_tracer("test-tracer")

        otlp_exporter = OTLPSpanExporter(
            endpoint="otel-collector:4317",
            insecure=True,
        )

        trace.get_tracer_provider().add_span_processor(
            BatchSpanProcessor(otlp_exporter)
        )

        return tracer
```

#### Performance Tests

```python
# tests/performance/test_load.py
import pytest
import threading
import time
import requests
from concurrent.futures import ThreadPoolExecutor

class TestLoadPerformance:
    def test_high_load_metrics_ingestion(self):
        """Test metrics ingestion under high load."""
        num_requests = 1000
        num_threads = 10

        start_time = time.time()

        def send_metrics_batch(batch_id):
            for i in range(100):
                metric_data = {
                    "name": f"test_metric_{batch_id}_{i}",
                    "value": i * 1.0,
                    "timestamp": int(time.time() * 1000)
                }
                response = requests.post(
                    "http://otel-collector:4318/v1/metrics",
                    json=metric_data
                )
                assert response.status_code == 200

        with ThreadPoolExecutor(max_workers=num_threads) as executor:
            futures = [
                executor.submit(send_metrics_batch, i)
                for i in range(num_threads)
            ]

            for future in futures:
                future.result()

        end_time = time.time()
        total_time = end_time - start_time

        # Assert performance requirements
        assert total_time < 30  # Should complete within 30 seconds
        requests_per_second = num_requests / total_time
        assert requests_per_second > 50  # At least 50 RPS
```

### Test Execution

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_collector.py

# Run with coverage
pytest --cov=collector --cov-report=html

# Run integration tests
pytest tests/integration/

# Run performance tests
pytest tests/performance/

# Run tests in parallel
pytest -n auto
```

### Test Configuration

```ini
# pytest.ini
[tool:pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = -v --strict-markers --tb=short
markers =
    unit: Unit tests
    integration: Integration tests
    performance: Performance tests
    slow: Slow running tests
```

## Contributing Guidelines

### Pull Request Process

1. **Fork the repository**
2. **Create a feature branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. **Make your changes**
4. **Add tests for your changes**
5. **Update documentation if needed**
6. **Run the test suite**
   ```bash
   pytest
   ```
7. **Commit your changes**
   ```bash
   git commit -m "Add feature: your feature description"
   ```
8. **Push to your fork**
   ```bash
   git push origin feature/your-feature-name
   ```
9. **Create a Pull Request**

### Commit Message Format

```
type(scope): description

[optional body]

[optional footer]
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes
- `refactor`: Code refactoring
- `test`: Test additions/changes
- `chore`: Maintenance tasks

Examples:
```
feat(collector): add OTLP HTTP receiver support

fix(prometheus): resolve memory leak in query engine

docs(api): update API reference for v2.0
```

### Code Review Process

1. **Automated Checks**
   - CI/CD pipeline runs automatically
   - Code quality checks (linting, formatting)
   - Test execution
   - Security scanning

2. **Peer Review**
   - At least one maintainer review required
   - Review focuses on:
     - Code correctness
     - Test coverage
     - Documentation updates
     - Performance implications
     - Security considerations

3. **Approval and Merge**
   - All checks must pass
   - At least one approval from maintainer
   - Squash merge preferred for clean history

## Release Process

### Version Numbering

Follows [Semantic Versioning](https://semver.org/):

- **MAJOR**: Breaking changes
- **MINOR**: New features, backward compatible
- **PATCH**: Bug fixes, backward compatible

### Release Steps

1. **Prepare Release Branch**
   ```bash
   git checkout -b release/v1.2.3
   ```

2. **Update Version Numbers**
   ```bash
   # Update version in relevant files
   sed -i 's/version: .*/version: 1.2.3/' docker-compose.yml
   sed -i 's/VERSION=.*/VERSION=1.2.3/' scripts/build.sh
   ```

3. **Update Changelog**
   ```markdown
   # Changelog

   ## [1.2.3] - 2023-12-01

   ### Added
   - New feature description

   ### Fixed
   - Bug fix description

   ### Changed
   - Breaking change description
   ```

4. **Create Release Tag**
   ```bash
   git tag -a v1.2.3 -m "Release version 1.2.3"
   git push origin v1.2.3
   ```

5. **Build and Publish**
   ```bash
   # Build Docker images
   docker build -t grafana/docker-otel-lgtm:v1.2.3 .

   # Push to registry
   docker push grafana/docker-otel-lgtm:v1.2.3
   ```

6. **Create GitHub Release**
   - Go to GitHub Releases
   - Create new release from tag
   - Copy changelog content
   - Attach any additional assets

### Pre-release Process

For beta/RC releases:

```bash
# Create pre-release tag
git tag -a v1.2.3-beta.1 -m "Pre-release version 1.2.3-beta.1"
git push origin v1.2.3-beta.1

# Build with pre-release tag
docker build -t grafana/docker-otel-lgtm:v1.2.3-beta.1 .
```

## CI/CD Pipeline

### GitHub Actions Workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Build Docker images
      run: |
        docker-compose build

    - name: Start services
      run: |
        docker-compose up -d
        docker-compose ps

    - name: Wait for services
      run: |
        timeout 300 bash -c 'until docker-compose exec -T otel-collector curl -f http://localhost:13133; do sleep 5; done'

    - name: Run tests
      run: |
        docker-compose exec -T test-runner pytest

    - name: Generate test report
      uses: actions/upload-artifact@v3
      with:
        name: test-results
        path: test-results/

  security:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: 'fs'
        scan-ref: '.'
        format: 'sarif'
        output: 'trivy-results.sarif'

    - name: Upload Trivy scan results
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: 'trivy-results.sarif'

  lint:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Run Docker lint
      run: |
        docker run --rm -i hadolint/hadolint < Dockerfile

    - name: Run YAML lint
      run: |
        yamllint .

    - name: Run shellcheck
      run: |
        find scripts -name "*.sh" -exec shellcheck {} \;
```

### Release Workflow

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Extract version
      run: |
        echo "VERSION=${GITHUB_REF#refs/tags/v}" >> $GITHUB_ENV

    - name: Build and push Docker image
      run: |
        echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
        docker build -t grafana/docker-otel-lgtm:${{ env.VERSION }} .
        docker push grafana/docker-otel-lgtm:${{ env.VERSION }}

    - name: Create GitHub release
      uses: actions/create-release@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        tag_name: ${{ github.ref }}
        release_name: Release ${{ github.ref }}
        body: |
          See [CHANGELOG.md](CHANGELOG.md) for details.
        draft: false
        prerelease: false
```

### Quality Gates

- **Test Coverage**: Minimum 80% code coverage
- **Security Scan**: No critical vulnerabilities
- **Lint Check**: All linting checks pass
- **Build Success**: All Docker images build successfully
- **Integration Tests**: All integration tests pass

### Deployment Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy to Staging

on:
  push:
    branches: [ main ]

jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment: staging
    steps:
    - uses: actions/checkout@v3

    - name: Deploy to staging
      run: |
        kubectl config set-context staging
        kubectl apply -f k8s/overlays/staging/
        kubectl rollout status deployment/lgtm-stack

    - name: Run smoke tests
      run: |
        # Run basic health checks
        curl -f http://staging.example.com/health
```

This development guide provides the foundation for contributing to the LGTM Stack project. Remember to always test your changes thoroughly and follow the established patterns and conventions.