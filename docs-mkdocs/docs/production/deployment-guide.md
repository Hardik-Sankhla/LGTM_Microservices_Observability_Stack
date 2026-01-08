# Production Deployment Guide

This guide covers production deployment considerations, security hardening, scalability, and operational best practices for the LGTM Stack.

## Table of Contents

- [Deployment Strategies](#deployment-strategies)
- [Security Considerations](#security-considerations)
- [Scalability and Performance](#scalability-and-performance)
- [Monitoring and Alerting](#monitoring-and-alerting)
- [Backup and Recovery](#backup-and-recovery)
- [Maintenance Procedures](#maintenance-procedures)

## Deployment Strategies

### Kubernetes Deployment

For production environments, deploy the LGTM Stack using Kubernetes manifests:

```yaml
# Example: prometheus-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: observability
spec:
  replicas: 2
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:v2.45.0
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: config
          mountPath: /etc/prometheus
        - name: data
          mountPath: /prometheus
      volumes:
      - name: config
        configMap:
          name: prometheus-config
      - name: data
        persistentVolumeClaim:
          claimName: prometheus-data
```

### Docker Compose (Staging/Development)

Use the provided `docker-compose.yml` for staging environments with production-like settings:

```yaml
version: '3.8'
services:
  otel-collector:
    image: otel/opentelemetry-collector-contrib:0.88.0
    environment:
      - OTEL_TRACES_EXPORTER=otlp
      - OTEL_METRICS_EXPORTER=prometheus
    volumes:
      - ./otelcol-config.yaml:/etc/otelcol/config.yaml
    ports:
      - "4317:4317"  # OTLP gRPC
      - "4318:4318"  # OTLP HTTP
```

## Security Considerations

### Network Security

1. **Service Mesh Integration**
   - Deploy with Istio or Linkerd for service-to-service authentication
   - Enable mTLS for encrypted communication between services

2. **Access Control**
   - Configure authentication for Grafana dashboards
   - Implement RBAC for Prometheus and Loki access
   - Use OAuth2/OIDC for single sign-on

3. **Network Policies**
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: observability-policy
   spec:
     podSelector:
       matchLabels:
         app: grafana
     policyTypes:
     - Ingress
     ingress:
     - from:
       - namespaceSelector:
           matchLabels:
             name: observability
   ```

### Data Security

1. **Encryption at Rest**
   - Enable encryption for persistent volumes
   - Configure TLS for data in transit

2. **Secret Management**
   - Use Kubernetes secrets or external secret managers
   - Rotate credentials regularly

## Scalability and Performance

### Horizontal Scaling

1. **Prometheus Federation**
   ```yaml
   global:
     scrape_interval: 15s
   rule_files:
     - "rules.yml"
   scrape_configs:
     - job_name: 'federate'
       scrape_interval: 15s
       honor_labels: true
       metrics_path: '/federate'
       params:
         'match[]':
           - '{job=~".+"}'
       static_configs:
         - targets:
           - 'prometheus-1:9090'
           - 'prometheus-2:9090'
   ```

2. **Tempo Scalability**
   - Use distributed tracing with multiple Tempo instances
   - Configure object storage (S3, GCS) for trace storage

3. **Loki Scaling**
   - Implement chunk and index storage separation
   - Use boltdb-shipper for index storage

### Performance Optimization

1. **Resource Limits**
   ```yaml
   resources:
     requests:
       memory: "512Mi"
       cpu: "250m"
     limits:
       memory: "1Gi"
       cpu: "500m"
   ```

2. **Query Optimization**
   - Use appropriate retention policies
   - Implement query result caching
   - Optimize PromQL queries

## Monitoring and Alerting

### Alerting Rules

```yaml
groups:
- name: observability.alerts
  rules:
  - alert: PrometheusDown
    expr: up{job="prometheus"} == 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Prometheus is down"
      description: "Prometheus has been down for more than 5 minutes."

  - alert: HighMemoryUsage
    expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 90
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High memory usage detected"
      description: "Memory usage is above 90% for more than 5 minutes."
```

### Health Checks

1. **Readiness Probes**
   ```yaml
   readinessProbe:
     httpGet:
       path: /ready
       port: 9090
     initialDelaySeconds: 30
     periodSeconds: 10
   ```

2. **Liveness Probes**
   ```yaml
   livenessProbe:
     httpGet:
       path: /-/healthy
       port: 9090
     initialDelaySeconds: 30
     periodSeconds: 30
   ```

## Backup and Recovery

### Data Backup

1. **Prometheus Data**
   ```bash
   # Backup Prometheus data
   kubectl exec -it prometheus-0 -- tar czf /tmp/prometheus-backup.tar.gz -C /prometheus .
   kubectl cp prometheus-0:/tmp/prometheus-backup.tar.gz ./prometheus-backup.tar.gz
   ```

2. **Loki Data**
   ```bash
   # Backup Loki chunks and indexes
   loki-cli backup --backend=s3 --bucket=loki-backups
   ```

### Disaster Recovery

1. **RTO/RPO Objectives**
   - Recovery Time Objective (RTO): 4 hours
   - Recovery Point Objective (RPO): 1 hour

2. **Multi-region Deployment**
   - Deploy across multiple availability zones
   - Use geo-redundant storage

## Maintenance Procedures

### Updates and Upgrades

1. **Rolling Updates**
   ```bash
   kubectl rollout restart deployment/prometheus
   kubectl rollout status deployment/prometheus
   ```

2. **Version Pinning**
   - Pin all component versions in production
   - Test upgrades in staging environment first

### Log Rotation

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: loki-config
data:
  loki.yaml: |
    limits_config:
      retention_period: 30d
    chunk_store_config:
      max_look_back_period: 30d
```

### Capacity Planning

1. **Resource Monitoring**
   - Monitor CPU, memory, and disk usage trends
   - Plan for 30% headroom for unexpected growth

2. **Scaling Triggers**
   - Auto-scale based on CPU utilization > 70%
   - Manual scaling for planned load increases

## Troubleshooting Production Issues

### Common Issues

1. **OOM Kills**
   - Increase memory limits
   - Optimize queries and retention policies

2. **Slow Queries**
   - Add query result caching
   - Use query optimization techniques

3. **Data Loss**
   - Implement proper backup procedures
   - Use persistent volumes with replication

### Debug Commands

```bash
# Check pod status
kubectl get pods -n observability

# View logs
kubectl logs -f deployment/prometheus -n observability

# Debug network issues
kubectl exec -it prometheus-0 -n observability -- curl http://loki:3100/ready
```