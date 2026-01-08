# Installation Guide

This guide provides comprehensive installation instructions for the LGTM Stack across different platforms and deployment scenarios.

## 📋 Prerequisites

### System Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **CPU** | 2 cores | 4+ cores |
| **Memory** | 4GB RAM | 8GB+ RAM |
| **Disk** | 2GB free | 10GB+ free |
| **Docker** | 20.10+ | Latest stable |
| **OS** | Linux/macOS/Windows | Linux/macOS |

### Required Ports

The stack requires these ports to be available:

| Port | Service | Protocol | Purpose |
|------|---------|----------|---------|
| `3000` | Grafana | HTTP | Web UI and API |
| `4040` | Pyroscope | HTTP | Profiling UI |
| `4317` | OTLP | gRPC | Telemetry ingestion |
| `4318` | OTLP | HTTP | Telemetry ingestion |
| `9090` | Prometheus | HTTP | Metrics API |
| `3100` | Loki | HTTP | Log API |
| `3200` | Tempo | HTTP | Trace API |

### Port Availability Check

#### Linux/macOS
```bash
# Check if ports are in use
netstat -tulpn | grep -E ':(3000|4040|4317|4318|9090|3100|3200)'

# Alternative using lsof
lsof -i :3000,4040,4317,4318,9090,3100,3200
```

#### Windows (PowerShell)
```powershell
# Check port usage
Get-NetTCPConnection | Where-Object { $_.LocalPort -in 3000,4040,4317,4318,9090,3100,3200 } | Select-Object LocalPort, State, OwningProcess
```

## 🚀 Installation Methods

### Method 1: Repository Clone (Recommended)

#### Step 1: Clone Repository
```bash
# Clone the repository
git clone https://github.com/grafana/docker-otel-lgtm.git
cd docker-otel-lgtm

# Verify contents
ls -la
```

#### Step 2: Make Scripts Executable (Linux/macOS)
```bash
# Make all scripts executable
chmod +x run-lgtm.sh run-example.sh generate-traffic.sh build-lgtm.sh

# Verify permissions
ls -la *.sh
```

#### Step 3: Verify Docker Installation
```bash
# Check Docker version
docker --version

# Check Docker daemon
docker info

# Test Docker functionality
docker run hello-world
```

### Method 2: Direct Docker Usage

#### Using Docker Hub
```bash
# Pull the latest image
docker pull grafana/otel-lgtm:latest

# Verify image
docker images grafana/otel-lgtm
```

#### Using GitHub Container Registry
```bash
# Pull from GHCR
docker pull ghcr.io/grafana/docker-otel-lgtm:latest

# Tag for consistency
docker tag ghcr.io/grafana/docker-otel-lgtm:latest grafana/otel-lgtm:latest
```

### Method 3: Docker Compose

#### Create docker-compose.yml
```yaml
version: '3.8'
services:
  lgtm:
    image: grafana/otel-lgtm:latest
    ports:
      - "3000:3000"    # Grafana
      - "4040:4040"    # Pyroscope
      - "4317:4317"    # OTLP gRPC
      - "4318:4318"    # OTLP HTTP
      - "9090:9090"    # Prometheus
      - "3100:3100"    # Loki
      - "3200:3200"    # Tempo
    volumes:
      - ./data/grafana:/data/grafana
      - ./data/prometheus:/data/prometheus
      - ./data/loki:/data/loki
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=secure_password
```

#### Run with Docker Compose
```bash
# Start services
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f
```

## 🖥️ Platform-Specific Installation

### Linux Installation

#### Ubuntu/Debian
```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER

# Restart session or run:
newgrp docker

# Clone and run
git clone https://github.com/grafana/docker-otel-lgtm.git
cd docker-otel-lgtm
./run-lgtm.sh
```

#### CentOS/RHEL/Fedora
```bash
# Install Docker
sudo dnf install -y docker

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group
sudo usermod -aG docker $USER

# Clone and run
git clone https://github.com/grafana/docker-otel-lgtm.git
cd docker-otel-lgtm
./run-lgtm.sh
```

### macOS Installation

#### Using Homebrew
```bash
# Install Docker Desktop
brew install --cask docker

# Start Docker Desktop
open /Applications/Docker.app

# Wait for Docker to start, then:
git clone https://github.com/grafana/docker-otel-lgtm.git
cd docker-otel-lgtm
./run-lgtm.sh
```

#### Manual Installation
1. Download Docker Desktop from [docker.com](https://www.docker.com/products/docker-desktop)
2. Install and start Docker Desktop
3. Open Terminal and run:
```bash
git clone https://github.com/grafana/docker-otel-lgtm.git
cd docker-otel-lgtm
./run-lgtm.sh
```

### Windows Installation

#### Windows 10/11 Pro/Enterprise
```powershell
# Install Docker Desktop
# Download from: https://www.docker.com/products/docker-desktop
# Run installer as administrator

# Open PowerShell as administrator
git clone https://github.com/grafana/docker-otel-lgtm.git
cd docker-otel-lgtm
.\run-lgtm.ps1
```

#### Windows Home (Using WSL2)
```powershell
# Enable WSL2
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# Install Ubuntu from Microsoft Store
# Then in Ubuntu terminal:
sudo apt update
sudo apt install docker.io
sudo systemctl start docker

git clone https://github.com/grafana/docker-otel-lgtm.git
cd docker-otel-lgtm
./run-lgtm.sh
```

## ⚙️ Configuration Options

### Environment Variables

#### Basic Configuration
```bash
# Custom ports
export GRAFANA_PORT=3001
export PROMETHEUS_PORT=9091

# Admin credentials
export GF_SECURITY_ADMIN_USER=myadmin
export GF_SECURITY_ADMIN_PASSWORD=mypassword

# Run with custom config
./run-lgtm.sh
```

#### Advanced Configuration
```bash
# Enable debug logging
export ENABLE_LOGS_ALL=true

# External OTLP export
export OTEL_EXPORTER_OTLP_ENDPOINT=https://my-otel-collector.com:4317
export OTEL_EXPORTER_OTLP_HEADERS="authorization=Bearer token123"

# Grafana plugins
export GF_PLUGINS_PREINSTALL=grafana-piechart-panel,grafana-worldmap-panel

# Run with configuration
./run-lgtm.sh
```

### Data Persistence

#### Volume Mounting
```bash
# Create data directories
mkdir -p data/{grafana,prometheus,loki,tempo,pyroscope}

# Run with persistent volumes
docker run -d --name lgtm \
  -v $(pwd)/data/grafana:/data/grafana \
  -v $(pwd)/data/prometheus:/data/prometheus \
  -v $(pwd)/data/loki:/data/loki \
  -v $(pwd)/data/tempo:/data/tempo \
  -v $(pwd)/data/pyroscope:/data/pyroscope \
  -p 3000:3000 -p 4040:4040 -p 4317:4317 -p 4318:4318 -p 9090:9090 \
  grafana/otel-lgtm:latest
```

#### Docker Compose with Volumes
```yaml
version: '3.8'
services:
  lgtm:
    image: grafana/otel-lgtm:latest
    volumes:
      - grafana_data:/data/grafana
      - prometheus_data:/data/prometheus
      - loki_data:/data/loki
      - tempo_data:/data/tempo
      - pyroscope_data:/data/pyroscope

volumes:
  grafana_data:
  prometheus_data:
  loki_data:
  tempo_data:
  pyroscope_data:
```

## 🧪 Verification

### Health Checks

#### Automatic Health Check
The container includes built-in health checks. Monitor startup:

```bash
# Watch startup logs
docker logs -f lgtm

# Check when ready
docker exec lgtm cat /tmp/ready
```

#### Manual Health Checks
```bash
# Grafana
curl -s http://localhost:3000/api/health | jq .database

# Prometheus
curl -s http://localhost:9090/api/v1/status/runtimeinfo | jq .status

# Loki
curl -s http://localhost:3100/ready

# Tempo
curl -s http://localhost:3200/ready

# Pyroscope
curl -s http://localhost:4040/ready

# OTLP Collector
curl -s http://localhost:13133/ready
```

### Service Verification

#### Check Running Services
```bash
# List container processes
docker exec lgtm ps aux

# Check network connections
docker exec lgtm netstat -tulpn
```

#### Test Telemetry Ingestion
```bash
# Send test trace
curl -X POST http://localhost:4318/v1/traces \
  -H "Content-Type: application/json" \
  -d '{"resourceSpans":[]}'

# Send test metrics
curl -X POST http://localhost:4318/v1/metrics \
  -H "Content-Type: application/json" \
  -d '{"resourceMetrics":[]}'

# Send test logs
curl -X POST http://localhost:4318/v1/logs \
  -H "Content-Type: application/json" \
  -d '{"resourceLogs":[]}'
```

## 🔧 Troubleshooting Installation

### Common Issues

#### Docker Not Running
```bash
# Start Docker service
sudo systemctl start docker  # Linux
# Or open Docker Desktop     # macOS/Windows

# Check Docker status
docker info
```

#### Port Conflicts
```bash
# Find what's using the ports
sudo lsof -i :3000  # Linux/macOS
netstat -ano | findstr :3000  # Windows

# Use different ports
docker run -p 3001:3000 -p 9091:9090 ... grafana/otel-lgtm:latest
```

#### Permission Issues
```bash
# Fix Docker socket permissions (Linux)
sudo chmod 666 /var/run/docker.sock

# Or add user to docker group
sudo usermod -aG docker $USER
newgrp docker
```

#### Memory Issues
```bash
# Check available memory
free -h  # Linux
vm_stat  # macOS

# Increase Docker memory limit in Docker Desktop settings
# Or run with memory limits
docker run --memory=4g --memory-swap=8g ... grafana/otel-lgtm:latest
```

#### Image Pull Issues
```bash
# Clear Docker cache
docker system prune -a

# Pull manually
docker pull grafana/otel-lgtm:latest

# Check network connectivity
curl -I https://registry-1.docker.io/
```

### Advanced Troubleshooting

#### Debug Mode
```bash
# Run with debug logging
docker run -e ENABLE_LOGS_ALL=true grafana/otel-lgtm:latest

# Check container logs
docker logs lgtm

# Enter container for debugging
docker exec -it lgtm /bin/bash
```

#### Network Issues
```bash
# Check container networking
docker network ls
docker inspect lgtm | jq .[0].NetworkSettings

# Test internal connectivity
docker exec lgtm curl -s http://127.0.0.1:3000/api/health
```

## 📦 Alternative Installation Methods

### Using Podman (Linux)
```bash
# Install Podman
sudo dnf install podman  # Fedora/RHEL
sudo apt install podman  # Ubuntu

# Run with Podman
podman run -d --name lgtm \
  -p 3000:3000 -p 4040:4040 -p 4317:4317 -p 4318:4318 -p 9090:9090 \
  docker.io/grafana/otel-lgtm:latest
```

### Using Kubernetes
```bash
# Apply Kubernetes manifests
kubectl apply -f k8s/lgtm.yaml

# Port forward services
kubectl port-forward service/lgtm 3000:3000 4040:4040 4317:4317 4318:4318 9090:9090

# Check pod status
kubectl get pods -l app=lgtm
```

### Using Helm
```bash
# Add Helm repository
helm repo add grafana https://grafana.github.io/helm-charts

# Install LGTM stack
helm install lgtm grafana/lgtm-stack
```

## 🔄 Upgrades and Updates

### Updating the Stack
```bash
# Pull latest image
docker pull grafana/otel-lgtm:latest

# Stop current container
docker stop lgtm

# Remove old container
docker rm lgtm

# Start with new image
docker run -d --name lgtm [same options as before] grafana/otel-lgtm:latest
```

### Version Pinning
```bash
# Use specific version
docker run grafana/otel-lgtm:v0.11.16

# Check available versions
curl -s https://registry.hub.docker.com/v2/repositories/grafana/otel-lgtm/tags | jq .results[].name
```

## 🆘 Getting Help

### Support Resources
- **Installation Issues**: [GitHub Issues](https://github.com/grafana/docker-otel-lgtm/issues)
- **Documentation**: [Full Documentation](README.md)
- **Community**: [GitHub Discussions](https://github.com/grafana/docker-otel-lgtm/discussions)

### Diagnostic Information
```bash
# System information
uname -a
docker --version
docker info

# Container information
docker ps -a
docker logs lgtm
docker inspect lgtm

# Network information
docker network ls
docker exec lgtm netstat -tulpn
```

---

**🎉 Installation Complete!** Your LGTM Stack is now ready. Proceed to the [Quick Start Guide](quick-start.md) to begin exploring your observability data! 🚀