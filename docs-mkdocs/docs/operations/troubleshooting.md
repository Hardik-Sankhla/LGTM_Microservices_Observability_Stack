# Troubleshooting Guide

This comprehensive troubleshooting guide helps you diagnose and resolve common issues with the LGTM Stack.

## 🚨 Quick Diagnosis

### Stack Health Check
```bash
# Check if container is running
docker ps | grep lgtm

# Check container logs
docker logs lgtm | tail -20

# Check all service endpoints
curl -s http://localhost:3000/api/health && echo " - Grafana OK" || echo " - Grafana FAIL"
curl -s http://localhost:9090/api/v1/status/runtimeinfo && echo " - Prometheus OK" || echo " - Prometheus FAIL"
curl -s http://localhost:3100/ready && echo " - Loki OK" || echo " - Loki FAIL"
curl -s http://localhost:3200/ready && echo " - Tempo OK" || echo " - Tempo FAIL"
curl -s http://localhost:4040/ready && echo " - Pyroscope OK" || echo " - Pyroscope FAIL"
curl -s http://localhost:13133/ready && echo " - OTLP Collector OK" || echo " - OTLP Collector FAIL"
```

### Port Availability
```bash
# Check port conflicts
netstat -tulpn | grep -E ':(3000|4040|4317|4318|9090|3100|3200)' || echo "All ports available"

# Find what's using a port
lsof -i :3000  # Replace with problematic port
```

## 🔧 Startup Issues

### Container Won't Start
```bash
# Check Docker status
docker info

# Check available resources
docker system df

# Try starting with minimal resources
docker run --rm grafana/otel-lgtm:latest echo "Container works"

# Check for port conflicts
docker run --rm -p 3000:3000 --name test-lgtm grafana/otel-lgtm:latest &
sleep 5
docker logs test-lgtm
docker stop test-lgtm
```

### Services Won't Start
```bash
# Check startup logs
docker logs lgtm | grep -A 5 -B 5 "failed\|error\|Error"

# Check data directory permissions
ls -la /data/
docker exec lgtm ls -la /data/

# Check available memory
free -h
docker stats lgtm --no-stream
```

### Port Already in Use
```bash
# Find conflicting process
sudo lsof -i :3000
sudo netstat -tulpn | grep :3000

# Kill conflicting process (be careful!)
sudo kill -9 <PID>

# Or use different ports
export GRAFANA_PORT=3001
export PROMETHEUS_PORT=9091
./run-lgtm.sh
```

## 🌐 Connectivity Issues

### Cannot Access Grafana
```bash
# Check if Grafana is responding
curl -v http://localhost:3000

# Check firewall
sudo ufw status
sudo iptables -L

# Check Docker networking
docker network ls
docker inspect lgtm | jq .[0].NetworkSettings.Ports
```

### OTLP Data Not Received
```bash
# Test OTLP endpoint
curl -X POST http://localhost:4318/v1/traces \
  -H "Content-Type: application/json" \
  -d '{"resourceSpans":[]}'

# Check OTLP Collector logs
docker logs lgtm | grep -i otelcol

# Verify collector health
curl http://localhost:13133/ready
```

### Applications Cannot Connect
```bash
# Test from application container
docker run --rm --network container:lgtm \
  curlimages/curl http://127.0.0.1:4318/v1/traces

# Check application configuration
# Verify OTEL_EXPORTER_OTLP_ENDPOINT
# Verify OTEL_EXPORTER_OTLP_HEADERS
```

## 📊 Data Issues

### No Metrics in Prometheus
```bash
# Check Prometheus health
curl http://localhost:9090/api/v1/status/runtimeinfo

# Query basic metrics
curl "http://localhost:9090/api/v1/query?query=up"

# Check OTLP ingestion
curl "http://localhost:9090/api/v1/query?query=otelcol_exporter_send_failed_spans_total"

# Verify data directory
docker exec lgtm ls -la /data/prometheus/
```

### No Traces in Tempo
```bash
# Check Tempo health
curl http://localhost:3200/ready

# Search for traces
curl "http://localhost:3200/api/search?tags=service.name%3Drolldice"

# Check Tempo logs
docker logs lgtm | grep -i tempo

# Verify data directory
docker exec lgtm ls -la /data/tempo/
```

### No Logs in Loki
```bash
# Check Loki health
curl http://localhost:3100/ready

# Query logs
curl "http://localhost:3100/loki/api/v1/query?query={job=\"rolldice\"}"

# Check Loki logs
docker logs lgtm | grep -i loki

# Verify data directory
docker exec lgtm ls -la /data/loki/
```

### No Profiles in Pyroscope
```bash
# Check Pyroscope health
curl http://localhost:4040/ready

# List applications
curl http://localhost:4040/api/apps

# Check Pyroscope logs
docker logs lgtm | grep -i pyroscope

# Verify data directory
docker exec lgtm ls -la /data/pyroscope/
```

## 🔍 Grafana Issues

### Cannot Login to Grafana
```bash
# Check default credentials
# Username: admin
# Password: admin

# Reset admin password
docker exec -it lgtm grafana-cli admin reset-admin-password newpassword

# Check Grafana logs
docker logs lgtm | grep -i grafana
```

### Data Sources Not Working
```bash
# Test data source connections
curl http://localhost:3000/api/datasources

# Check individual data sources
curl http://localhost:3000/api/datasources/1/health  # Prometheus
curl http://localhost:3000/api/datasources/2/health  # Tempo
curl http://localhost:3000/api/datasources/3/health  # Loki
curl http://localhost:3000/api/datasources/4/health  # Pyroscope
```

### Dashboards Show No Data
```bash
# Check query in panel
# Open browser dev tools → Network tab
# Look for failed API calls

# Test queries directly
curl "http://localhost:9090/api/v1/query?query=up"
curl "http://localhost:3100/loki/api/v1/query?query={job=\"rolldice\"}"
```

## 📈 Performance Issues

### High Memory Usage
```bash
# Check container memory
docker stats lgtm

# Check component memory usage
docker exec lgtm ps aux --sort=-%mem | head -10

# Limit container memory
docker run --memory=4g --memory-swap=8g grafana/otel-lgtm:latest
```

### High CPU Usage
```bash
# Check container CPU
docker stats lgtm

# Check component CPU usage
docker exec lgtm ps aux --sort=-%cpu | head -10

# Limit container CPU
docker run --cpus=2 grafana/otel-lgtm:latest
```

### Slow Queries
```bash
# Check query performance
curl "http://localhost:9090/api/v1/query_range?query=up&start=$(date +%s - 3600)&end=$(date +%s)&step=60"

# Check Tempo query performance
curl "http://localhost:3200/api/metrics/query_range?query=rate(tempo_request_duration_seconds_count[5m])&start=$(date +%s - 3600)&end=$(date +%s)&step=60"

# Optimize queries
# Use appropriate time ranges
# Add query filters
```

### Storage Issues
```bash
# Check disk usage
df -h
docker system df

# Check data directory sizes
docker exec lgtm du -sh /data/*

# Clean up old data
docker exec lgtm find /data -name "*.tmp" -delete
```

## 🔄 Data Flow Issues

### Telemetry Pipeline Broken
```bash
# Test complete pipeline
# 1. Send test data
curl -X POST http://localhost:4318/v1/traces \
  -H "Content-Type: application/json" \
  -d '{"resourceSpans":[{"resource":{"attributes":[{"key":"service.name","value":{"stringValue":"test"}}]},"scopeSpans":[{"spans":[{"name":"test","kind":1,"spanId":"123","traceId":"456"}]}]}]}'

# 2. Check if data reached backends
curl "http://localhost:9090/api/v1/query?query=otelcol_exporter_sent_spans_total"
curl "http://localhost:3200/api/search?tags=service.name%3Dtest"
```

### Batch Processing Issues
```bash
# Check batch processor metrics
curl "http://localhost:8888/metrics" | grep batch

# Adjust batch settings (requires config change)
# Edit docker/otelcol-config.yaml
processors:
  batch:
    timeout: 5s
    send_batch_size: 1024
```

## 🔐 Security Issues

### Authentication Problems
```bash
# Check Grafana auth settings
docker exec lgtm env | grep GF_AUTH

# Reset to defaults
unset GF_SECURITY_ADMIN_PASSWORD
docker restart lgtm
```

### Permission Issues
```bash
# Check data directory permissions
docker exec lgtm ls -la /data/

# Fix permissions
docker exec lgtm chown -R 472:472 /data/grafana
docker exec lgtm chown -R 65534:65534 /data/loki
```

## 🌐 Network Issues

### DNS Resolution
```bash
# Check DNS from container
docker exec lgtm nslookup google.com

# Check network connectivity
docker exec lgtm ping -c 3 8.8.8.8
```

### Firewall Issues
```bash
# Check firewall rules
sudo ufw status
sudo iptables -L

# Allow Docker networking
sudo ufw allow 3000
sudo ufw allow 4317:4318/tcp
```

### Proxy Issues
```bash
# Check proxy settings
env | grep -i proxy

# Configure Docker proxy
# Edit ~/.docker/config.json
{
  "proxies": {
    "default": {
      "httpProxy": "http://proxy.example.com:3128",
      "httpsProxy": "http://proxy.example.com:3128"
    }
  }
}
```

## 🐳 Docker Issues

### Container Crashes
```bash
# Check crash logs
docker logs lgtm

# Check exit code
docker inspect lgtm | jq .[0].State.ExitCode

# Check resource limits
docker stats lgtm --no-stream
```

### Image Issues
```bash
# Check image integrity
docker images grafana/otel-lgtm
docker run --rm grafana/otel-lgtm:latest echo "Image OK"

# Re-pull image
docker rmi grafana/otel-lgtm:latest
docker pull grafana/otel-lgtm:latest
```

### Volume Issues
```bash
# Check volume mounts
docker inspect lgtm | jq .[0].Mounts

# Verify host directories exist
ls -la ./data/

# Check volume permissions
ls -la ./data/grafana/
```

## 📝 Log Analysis

### Enable Debug Logging
```bash
# Enable all logs
export ENABLE_LOGS_ALL=true
docker restart lgtm

# Check logs
docker logs -f lgtm
```

### Common Log Patterns
```bash
# Errors
docker logs lgtm | grep -i error

# Warnings
docker logs lgtm | grep -i warn

# Startup sequence
docker logs lgtm | grep -E "(starting|started|ready)"

# OTLP collector issues
docker logs lgtm | grep otelcol
```

## 🔧 Recovery Procedures

### Data Recovery
```bash
# Backup data
docker run --rm -v lgtm_data:/data -v $(pwd):/backup alpine tar czf /backup/backup.tar.gz -C /data .

# Restore data
docker run --rm -v lgtm_data:/data -v $(pwd):/backup alpine tar xzf /backup/backup.tar.gz -C /data
```

### Configuration Reset
```bash
# Reset to defaults
docker rm -f lgtm
docker volume rm lgtm_data
./run-lgtm.sh
```

### Clean Restart
```bash
# Stop everything
docker stop lgtm
docker rm lgtm

# Clean up
docker system prune -f

# Restart fresh
./run-lgtm.sh
```

## 📞 Getting Help

### Diagnostic Information
```bash
# System info
uname -a
docker --version
docker info

# Container info
docker ps -a
docker logs lgtm | tail -100
docker inspect lgtm

# Network info
docker network ls
ip route
```

### Support Resources
- **GitHub Issues**: [Report bugs](https://github.com/grafana/docker-otel-lgtm/issues)
- **Community**: [GitHub Discussions](https://github.com/grafana/docker-otel-lgtm/discussions)
- **Documentation**: [Full docs](README.md)

### Emergency Contacts
- **Critical Issues**: Open GitHub issue with "CRITICAL" label
- **Security Issues**: Report privately to security@grafana.com
- **Commercial Support**: [Grafana Cloud support](https://grafana.com/support/)

---

This troubleshooting guide covers the most common issues. If you encounter an issue not covered here, please check the [GitHub issues](https://github.com/grafana/docker-otel-lgtm/issues) or open a new issue with detailed diagnostic information. 🐛🔧