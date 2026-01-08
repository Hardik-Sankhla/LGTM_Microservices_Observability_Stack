# Quick Start Guide

Get the LGTM Stack up and running in under 5 minutes! This guide covers the fastest way to launch the observability stack and start collecting telemetry data.

## 🚀 Prerequisites

### System Requirements
- **Docker**: Version 20.10 or later
- **Memory**: At least 4GB RAM available
- **Disk Space**: 2GB free space for container images and data
- **Ports**: Ensure ports 3000, 4040, 4317, 4318, 9090 are available

### Platform Support
- ✅ **Linux** (Ubuntu, CentOS, Fedora, etc.)
- ✅ **macOS** (Intel and Apple Silicon)
- ✅ **Windows** (PowerShell support)
- ✅ **WSL2** on Windows

## 📦 Installation

### Option 1: Clone Repository (Recommended)

```bash
# Clone the repository
git clone https://github.com/grafana/docker-otel-lgtm.git
cd docker-otel-lgtm

# Make scripts executable (Linux/macOS)
chmod +x run-lgtm.sh run-example.sh generate-traffic.sh
```

### Option 2: Direct Docker Run

```bash
# Pull and run directly (no repository clone needed)
docker run -d --name lgtm \
  -p 3000:3000 \
  -p 4040:4040 \
  -p 4317:4317 \
  -p 4318:4318 \
  -p 9090:9090 \
  grafana/otel-lgtm:latest
```

## 🎯 Launch the Stack

### Linux/macOS
```bash
# Start the observability stack
./run-lgtm.sh
```

### Windows (PowerShell)
```powershell
# Start the observability stack
.\run-lgtm.ps1
```

### Using Docker Compose (Alternative)
```bash
# If you prefer docker-compose
docker-compose up -d
```

## ⏳ Startup Process

The startup process takes **30-60 seconds**. You'll see output like:

```
Waiting for the OpenTelemetry collector and the Grafana LGTM stack to start up...
Grafana is up and running. Startup time: 12 seconds
Loki is up and running. Startup time: 15 seconds
Prometheus is up and running. Startup time: 18 seconds
Tempo is up and running. Startup time: 22 seconds
Pyroscope is up and running. Startup time: 25 seconds
OpenTelemetry collector is up and running. Startup time: 28 seconds
Total startup time: 35 seconds
```

## 🌐 Access the Interfaces

Once started, access these URLs:

| Service | URL | Purpose |
|---------|-----|---------|
| **Grafana** | http://localhost:3000 | Main dashboard (admin/admin) |
| **Prometheus** | http://localhost:9090 | Metrics query interface |
| **Tempo** | http://localhost:3200 | Trace search interface |
| **Loki** | http://localhost:3100 | Log query interface |
| **Pyroscope** | http://localhost:4040 | Profiling interface |

## 📊 Run Example Application

### Start Sample Application

#### Linux/macOS
```bash
# Run the Python example
./run-example.sh
```

#### Windows
```cmd
# Run the Python example
.\run-example.cmd
```

#### Manual Start (Any Platform)
```bash
# Navigate to Python example
cd examples/python

# Install dependencies
pip install -r requirements.txt

# Run the application
python app.py
```

### Verify Application is Running

```bash
# Test the rolldice endpoint
curl http://localhost:8082/rolldice
# Expected output: A random number between 1-6

# Test with player parameter
curl "http://localhost:8082/rolldice?player=Alice"
# Expected output: A random number with player context
```

## 🔄 Generate Sample Data

### Start Traffic Generator

#### Linux/macOS
```bash
# Generate sample telemetry data
./generate-traffic.sh
```

#### Windows
```cmd
# Generate sample telemetry data
.\generate-traffic.cmd
```

### What Happens
- **Metrics**: CPU, memory, and custom business metrics
- **Traces**: Request spans with service dependencies
- **Logs**: Structured logs with trace correlation
- **Profiles**: Performance profiling data

## 📈 Explore the Data

### 1. Grafana Dashboards
1. Open http://localhost:3000
2. Login with `admin` / `admin`
3. Navigate to **Dashboards** → **Browse**
4. Explore pre-built dashboards:
   - **RED Metrics**: Request, Error, Duration metrics
   - **JVM Metrics**: Java application metrics
   - **Service Graph**: Service dependencies

### 2. Query Individual Services

#### Prometheus Metrics
```promql
# Request rate
rate(http_requests_total[5m])

# Error rate
rate(http_requests_total{status_code=~"5.."}[5m])

# 95th percentile latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

#### Loki Logs
```logql
# All logs from the application
{job="rolldice"}

# Logs with errors
{job="rolldice"} |= "ERROR"

# Logs for specific player
{job="rolldice"} |= "Alice"
```

#### Tempo Traces
- Use the **Search** tab to find traces
- Filter by service name, operation, or tags
- Click on traces to see detailed spans

## 🧪 Test Different Languages

The repository includes examples in multiple languages. Try them all:

```bash
# Go example (port 8081)
cd examples/go && ./run.sh

# Java example (port 8080)
cd examples/java && ./run.sh

# Node.js example (port 8084)
cd examples/nodejs && ./run.sh

# .NET example (port 8083)
cd examples/dotnet && ./run.sh
```

Test each service:
```bash
curl http://localhost:8080/rolldice  # Java
curl http://localhost:8081/rolldice  # Go
curl http://localhost:8084/rolldice  # Node.js
curl http://localhost:8083/rolldice  # .NET
```

## 🔧 Troubleshooting

### Stack Won't Start
```bash
# Check if ports are available
netstat -tulpn | grep -E ':(3000|4040|4317|4318|9090)'

# Check Docker status
docker ps -a | grep lgtm

# View container logs
docker logs lgtm
```

### Application Won't Connect
```bash
# Verify OTLP endpoint is accessible
curl -v http://localhost:4318/v1/traces

# Check application logs for connection errors
# Look for OTLP exporter errors
```

### No Data in Dashboards
```bash
# Check if traffic generator is running
ps aux | grep generate-traffic

# Verify metrics are being collected
curl http://localhost:9090/api/v1/query?query=up
```

### Port Conflicts
```bash
# Use different ports with environment variables
export GRAFANA_PORT=3001
export PROMETHEUS_PORT=9091
# ... then run the stack
```

## 🧹 Cleanup

### Stop Everything
```bash
# Stop the stack
docker stop lgtm

# Remove container and volumes
docker rm -v lgtm

# Clean up images (optional)
docker rmi grafana/otel-lgtm:latest
```

### Reset Data
```bash
# Remove data volumes
rm -rf container/grafana/
rm -rf container/prometheus/
rm -rf container/loki/
```

## 🎯 Next Steps

### Learn More
- **[Architecture Guide](architecture.md)**: Understand how components work together
- **[Configuration Guide](configuration/index.md)**: Customize the stack
- **[Examples](examples/index.md)**: Instrument your own applications

### Advanced Usage
- **[Production Deployment](production/readiness.md)**: Moving to production
- **[Custom Dashboards](configuration/dashboards.md)**: Build your own visualizations
- **[Integration Guide](api/integration.md)**: Connect external systems

### Development
- **[Contributing Guide](contributing/index.md)**: Contribute to the project
- **[Testing](contributing/testing.md)**: Run the test suite
- **[Development Setup](contributing/development.md)**: Set up development environment

---

**🎉 Congratulations!** You now have a fully functional observability stack running locally. Explore the dashboards, try different examples, and start instrumenting your own applications!

Need help? Check the [Troubleshooting Guide](operations/troubleshooting.md) or open an issue on [GitHub](https://github.com/grafana/docker-otel-lgtm/issues). 🚀