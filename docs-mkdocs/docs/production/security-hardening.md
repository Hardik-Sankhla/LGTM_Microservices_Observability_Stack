# Security Hardening Guide

This guide provides comprehensive security hardening recommendations for the LGTM Stack in production environments.

## Table of Contents

- [Security Assessment](#security-assessment)
- [Authentication and Authorization](#authentication-and-authorization)
- [Network Security](#network-security)
- [Data Protection](#data-protection)
- [Infrastructure Security](#infrastructure-security)
- [Monitoring and Compliance](#monitoring-and-compliance)
- [Incident Response](#incident-response)

## Security Assessment

### Current Security Posture

**Strengths:**
- Container-based deployment provides isolation
- OpenTelemetry provides secure data transmission protocols
- Version-pinned dependencies reduce supply chain risks

**Weaknesses:**
- No authentication mechanisms
- Plain text configuration files
- No network segmentation
- Missing encryption at rest
- No audit logging

### Threat Model

**Attack Vectors:**
1. **Unauthorized Access**
   - Exposed dashboards without authentication
   - Weak or default credentials
   - Misconfigured access controls

2. **Data Exfiltration**
   - Unencrypted data in transit
   - Insecure API endpoints
   - Vulnerable third-party dependencies

3. **Denial of Service**
   - Resource exhaustion attacks
   - Malformed telemetry data
   - Network flooding

4. **Supply Chain Attacks**
   - Compromised container images
   - Malicious dependencies
   - Insecure CI/CD pipelines

## Authentication and Authorization

### Implementing Authentication

#### 1. Grafana Authentication
```yaml
# grafana.ini
[auth]
disable_login_form = true

[auth.generic_oauth]
enabled = true
name = Keycloak
client_id = grafana
client_secret = ${GRAFANA_CLIENT_SECRET}
scopes = openid profile email
auth_url = https://keycloak.example.com/auth/realms/master/protocol/openid-connect/auth
token_url = https://keycloak.example.com/auth/realms/master/protocol/openid-connect/token
api_url = https://keycloak.example.com/auth/realms/master/protocol/openid-connect/userinfo
```

#### 2. Prometheus Basic Auth
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'secure-service'
    basic_auth:
      username: ${PROMETHEUS_USERNAME}
      password: ${PROMETHEUS_PASSWORD}
    static_configs:
      - targets: ['service:9090']
```

#### 3. API Gateway Authentication
```yaml
# istio-gateway.yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: lgtm-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: lgtm-tls
    hosts:
    - lgtm.example.com
```

### Role-Based Access Control (RBAC)

#### Kubernetes RBAC
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: observability-reader
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: observability-readers
subjects:
- kind: Group
  name: observability-users
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: observability-reader
  apiGroup: rbac.authorization.k8s.io
```

#### Service-Level RBAC
```yaml
# Loki RBAC configuration
auth_enabled: true
multitenancy_enabled: true

# Tempo RBAC
auth:
  enabled: true
  oidc:
    issuer_url: https://keycloak.example.com/auth/realms/master
    client_id: tempo
    client_secret: ${TEMPO_CLIENT_SECRET}
```

## Network Security

### Service Mesh Security

#### Istio Configuration
```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: observability
spec:
  mtls:
    mode: STRICT

---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: grafana-policy
  namespace: observability
spec:
  selector:
    matchLabels:
      app: grafana
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/observability/sa/grafana"]
    to:
    - operation:
        methods: ["GET", "POST"]
```

### Network Policies

#### Kubernetes Network Policies
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: observability-network-policy
  namespace: observability
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    - podSelector:
        matchLabels:
          app: api-gateway
    ports:
    - protocol: TCP
      port: 3000  # Grafana
    - protocol: TCP
      port: 9090  # Prometheus
  egress:
  - to:
    - podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53
  - to: []
    ports:
    - protocol: TCP
      port: 443  # HTTPS outbound
```

### TLS Configuration

#### Certificate Management
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx

---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: lgtm-tls
  namespace: observability
spec:
  secretName: lgtm-tls-secret
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - lgtm.example.com
  - grafana.example.com
  - prometheus.example.com
```

## Data Protection

### Encryption at Rest

#### Database Encryption
```yaml
# Prometheus with encryption
global:
  scrape_interval: 15s

storage:
  tsdb:
    path: /prometheus
    wal_compression: true
  encryption:
    enabled: true
    key_file: /etc/prometheus/encryption.key
```

#### Loki Encryption
```yaml
# Loki configuration with encryption
storage_config:
  filesystem:
    directory: /loki/chunks
  boltdb_shipper:
    active_index_directory: /loki/index
    cache_location: /loki/cache
    shared_store: filesystem
    encryption:
      enabled: true
      key_file: /etc/loki/encryption.key
```

### Data in Transit

#### OTLP TLS Configuration
```yaml
# OpenTelemetry Collector TLS
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
        tls:
          cert_file: /etc/ssl/certs/otel-collector.crt
          key_file: /etc/ssl/private/otel-collector.key
      http:
        endpoint: 0.0.0.0:4318
        tls:
          cert_file: /etc/ssl/certs/otel-collector.crt
          key_file: /etc/ssl/private/otel-collector.key
```

### Data Sanitization

#### Input Validation
```python
# Python input validation for telemetry data
from pydantic import BaseModel, validator
from typing import Optional

class TelemetryData(BaseModel):
    service_name: str
    metric_name: str
    value: float
    timestamp: Optional[int] = None

    @validator('service_name')
    def validate_service_name(cls, v):
        if not v or len(v) > 100:
            raise ValueError('Invalid service name')
        return v

    @validator('metric_name')
    def validate_metric_name(cls, v):
        if not v or len(v) > 200:
            raise ValueError('Invalid metric name')
        return v
```

## Infrastructure Security

### Container Security

#### Security Context
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-grafana
spec:
  template:
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 472
        runAsGroup: 472
        fsGroup: 472
      containers:
      - name: grafana
        image: grafana/grafana:10.1.0
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop:
            - ALL
        volumeMounts:
        - name: grafana-storage
          mountPath: /var/lib/grafana
        - name: tmp-volume
          mountPath: /tmp
      volumes:
      - name: grafana-storage
        persistentVolumeClaim:
          claimName: grafana-pvc
      - name: tmp-volume
        emptyDir: {}
```

#### Image Security
```dockerfile
# Secure Dockerfile
FROM grafana/grafana:10.1.0

# Remove unnecessary packages
RUN apt-get update && apt-get remove -y \
    curl \
    wget \
    && rm -rf /var/lib/apt/lists/*

# Add non-root user
RUN useradd -r -u 472 -g root grafana

# Set proper permissions
RUN chown -R grafana:root /var/lib/grafana && \
    chmod -R 755 /var/lib/grafana

USER grafana
```

### Secret Management

#### HashiCorp Vault Integration
```yaml
# Vault configuration
apiVersion: v1
kind: Secret
metadata:
  name: vault-config
type: Opaque
data:
  VAULT_ADDR: aHR0cHM6Ly92YXVsdC5leGFtcGxlLmNvbTo4MjAw
  VAULT_TOKEN: czFzMnQzdDR0NW42N3M4OWUwMTIzNDU2Nzg5MGFiY2RlZg==

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana-with-vault
spec:
  template:
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:10.1.0
        env:
        - name: GF_DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: grafana-secrets
              key: db-password
        volumeMounts:
        - name: vault-secrets
          mountPath: /vault/secrets
      initContainers:
      - name: vault-init
        image: vault:1.13.0
        command: ['vault', 'agent', '-config=/etc/vault/config.hcl']
        volumeMounts:
        - name: vault-config
          mountPath: /etc/vault
        - name: vault-secrets
          mountPath: /vault/secrets
      volumes:
      - name: vault-config
        secret:
          secretName: vault-config
      - name: vault-secrets
        emptyDir: {}
```

## Monitoring and Compliance

### Security Monitoring

#### Audit Logging
```yaml
# Kubernetes audit policy
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
  verbs: ["create", "update", "patch", "delete"]
  resources:
  - group: ""
    resources: ["secrets"]
- level: RequestResponse
  verbs: ["create", "update"]
  resources:
  - group: "rbac.authorization.k8s.io"
    resources: ["clusterroles", "clusterrolebindings"]
```

#### Security Dashboards
```json
// Grafana security dashboard panels
{
  "panels": [
    {
      "title": "Failed Authentication Attempts",
      "type": "graph",
      "targets": [
        {
          "expr": "rate(grafana_login_failed_total[5m])",
          "legendFormat": "Failed Logins"
        }
      ]
    },
    {
      "title": "Suspicious Network Traffic",
      "type": "table",
      "targets": [
        {
          "expr": "rate(istio_requests_total{response_code=~\"4..|5..\"}[5m])",
          "legendFormat": "Error Rate"
        }
      ]
    }
  ]
}
```

### Compliance Requirements

#### GDPR Compliance
- Data minimization principles
- Right to erasure implementation
- Data processing records
- Privacy by design

#### SOC 2 Compliance
- Security controls documentation
- Access control procedures
- Change management processes
- Incident response procedures

#### HIPAA Compliance (if applicable)
- PHI data handling procedures
- Audit trail requirements
- Data encryption standards
- Access control matrices

## Incident Response

### Security Incident Response Plan

#### 1. Preparation Phase
- Establish incident response team
- Define roles and responsibilities
- Prepare communication channels
- Create incident response runbook

#### 2. Detection Phase
```yaml
# Security alerting rules
groups:
- name: security.alerts
  rules:
  - alert: HighFailedLoginRate
    expr: rate(grafana_login_failed_total[5m]) > 10
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "High rate of failed login attempts"
      description: "More than 10 failed login attempts per minute detected"

  - alert: UnauthorizedAccess
    expr: istio_requests_total{response_code="403"} > 5
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "Multiple unauthorized access attempts"
      description: "Multiple 403 responses detected"
```

#### 3. Response Phase
```bash
# Incident response script
#!/bin/bash

# Isolate affected components
kubectl scale deployment compromised-service --replicas=0

# Enable emergency access controls
kubectl apply -f emergency-security-policy.yaml

# Collect forensic data
kubectl logs -l app=compromised-service --since=1h > incident_logs.txt

# Notify security team
curl -X POST -H 'Content-type: application/json' \
  --data '{"text":"Security incident detected"}' \
  $SLACK_WEBHOOK_URL
```

#### 4. Recovery Phase
- Restore from clean backups
- Apply security patches
- Update security controls
- Conduct post-mortem analysis

#### 5. Lessons Learned
- Document incident details
- Update security procedures
- Improve detection capabilities
- Conduct security training

### Communication Plan

#### Internal Communication
- Incident response team notifications
- Status updates to management
- Technical team coordination

#### External Communication
- Customer impact assessment
- Regulatory reporting requirements
- Public communication strategy

### Forensic Procedures

#### Log Collection
```bash
# Collect system logs
journalctl --since "1 hour ago" > system_logs.txt

# Collect Kubernetes events
kubectl get events --sort-by=.metadata.creationTimestamp > k8s_events.txt

# Collect network logs
tcpdump -i any -w incident_traffic.pcap
```

#### Evidence Preservation
- Create forensic images of affected systems
- Preserve volatile memory
- Document chain of custody
- Maintain evidence integrity

## Security Testing

### Automated Security Testing

#### Container Image Scanning
```yaml
# Trivy configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: trivy-config
data:
  trivy.yaml: |
    scan:
      scanners:
        - vuln
        - secret
        - config
      skip-dirs:
        - node_modules
      skip-files:
        - "*.test.go"
```

#### Dependency Scanning
```yaml
# OWASP Dependency Check
stages:
  - security
security_scan:
  stage: security
  script:
    - /usr/local/bin/dependency-check.sh
      --project "LGTM Stack"
      --scan .
      --format "ALL"
      --out dependency-check-report
  artifacts:
    reports:
      dependency_scanning: dependency-check-report/dependency-check-report.json
```

### Penetration Testing

#### Testing Scope
- External network perimeter
- Web application interfaces
- API endpoints
- Authentication mechanisms

#### Testing Methodology
1. Reconnaissance
2. Scanning
3. Gaining Access
4. Maintaining Access
5. Covering Tracks

### Compliance Audits

#### Regular Security Assessments
- Quarterly vulnerability scans
- Annual penetration testing
- Continuous compliance monitoring
- Security control validation

#### Audit Preparation
- Maintain security documentation
- Conduct internal audits
- Prepare evidence collection
- Train staff on audit procedures

## Security Maintenance

### Patch Management

#### Automated Updates
```yaml
# Kyverno policy for image updates
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: update-container-images
spec:
  rules:
  - name: update-images
    match:
      resources:
        kinds:
        - Pod
    mutate:
      patchStrategicMerge:
        spec:
          containers:
          - (name): "*"
            image: "{{ mutate.containers[].image | update_registry('registry.example.com') }}"
```

#### Update Strategy
- Test updates in staging environment
- Schedule maintenance windows
- Implement rollback procedures
- Monitor for regressions

### Security Training

#### Team Training Requirements
- Security awareness training
- Role-specific security training
- Incident response drills
- Compliance training

#### Training Schedule
- Annual security training
- Quarterly incident response drills
- Monthly security updates
- Ad-hoc training for new threats

## Security Metrics

### Key Security Metrics

1. **Prevention Metrics**
   - Vulnerability scan results
   - Patch compliance percentage
   - Security training completion rate

2. **Detection Metrics**
   - Mean time to detect (MTTD)
   - False positive rate
   - Security alert volume

3. **Response Metrics**
   - Mean time to respond (MTTR)
   - Incident resolution time
   - Recovery time objective achievement

4. **Compliance Metrics**
   - Audit finding remediation rate
   - Policy compliance percentage
   - Security control effectiveness

### Security Dashboards

Create comprehensive security dashboards to monitor:
- Authentication failures
- Unauthorized access attempts
- Security control status
- Compliance metrics
- Incident trends