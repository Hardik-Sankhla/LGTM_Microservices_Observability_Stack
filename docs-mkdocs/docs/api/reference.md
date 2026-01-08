# API Reference

This document provides comprehensive API reference for the LGTM Stack components.

## Table of Contents

- [OpenTelemetry Protocol (OTLP)](#opentelemetry-protocol-otlp)
- [Prometheus HTTP API](#prometheus-http-api)
- [Grafana HTTP API](#grafana-http-api)
- [Loki HTTP API](#loki-http-api)
- [Tempo HTTP API](#tempo-http-api)
- [Pyroscope HTTP API](#pyroscope-http-api)

## OpenTelemetry Protocol (OTLP)

### Overview

The OpenTelemetry Protocol (OTLP) is used for ingesting telemetry data (metrics, traces, and logs) into the LGTM Stack.

### Endpoints

#### gRPC Endpoints

| Endpoint | Protocol | Purpose |
|----------|----------|---------|
| `otel-collector:4317` | gRPC | Telemetry data ingestion |
| `otel-collector:4318` | HTTP | Telemetry data ingestion |

#### HTTP Endpoints

```
POST /v1/metrics
POST /v1/traces
POST /v1/logs
```

### Metrics Ingestion

#### Request Format
```protobuf
message ExportMetricsServiceRequest {
  repeated ResourceMetrics resource_metrics = 1;
}

message ResourceMetrics {
  Resource resource = 1;
  repeated ScopeMetrics scope_metrics = 2;
  string schema_url = 3;
}

message ScopeMetrics {
  InstrumentationScope scope = 1;
  repeated Metric metrics = 2;
  string schema_url = 3;
}
```

#### Example Request
```bash
curl -X POST http://otel-collector:4318/v1/metrics \
  -H "Content-Type: application/json" \
  -d '{
    "resourceMetrics": [
      {
        "resource": {
          "attributes": [
            {
              "key": "service.name",
              "value": {
                "stringValue": "example-service"
              }
            }
          ]
        },
        "scopeMetrics": [
          {
            "scope": {
              "name": "example-meter",
              "version": "1.0.0"
            },
            "metrics": [
              {
                "name": "request_count",
                "unit": "1",
                "sum": {
                  "aggregationTemporality": 2,
                  "isMonotonic": true,
                  "dataPoints": [
                    {
                      "attributes": [
                        {
                          "key": "method",
                          "value": {
                            "stringValue": "GET"
                          }
                        }
                      ],
                      "startTimeUnixNano": "1641945600000000000",
                      "timeUnixNano": "1641945660000000000",
                      "value": 100
                    }
                  ]
                }
              }
            ]
          }
        ]
      }
    ]
  }'
```

### Traces Ingestion

#### Request Format
```protobuf
message ExportTraceServiceRequest {
  repeated ResourceSpans resource_spans = 1;
}

message ResourceSpans {
  Resource resource = 1;
  repeated ScopeSpans scope_spans = 2;
  string schema_url = 3;
}
```

#### Example Request
```bash
curl -X POST http://otel-collector:4318/v1/traces \
  -H "Content-Type: application/json" \
  -d '{
    "resourceSpans": [
      {
        "resource": {
          "attributes": [
            {
              "key": "service.name",
              "value": {
                "stringValue": "example-service"
              }
            }
          ]
        },
        "scopeSpans": [
          {
            "scope": {
              "name": "example-tracer",
              "version": "1.0.0"
            },
            "spans": [
              {
                "traceId": "5B8EFFF798038103D269B633813FC60C",
                "spanId": "05EDD006BC5F551A",
                "name": "example-span",
                "kind": 1,
                "startTimeUnixNano": "1641945600000000000",
                "endTimeUnixNano": "1641945660000000000",
                "attributes": [
                  {
                    "key": "http.method",
                    "value": {
                      "stringValue": "GET"
                    }
                  }
                ],
                "status": {
                  "code": 1
                }
              }
            ]
          }
        ]
      }
    ]
  }'
```

### Logs Ingestion

#### Request Format
```protobuf
message ExportLogsServiceRequest {
  repeated ResourceLogs resource_logs = 1;
}

message ResourceLogs {
  Resource resource = 1;
  repeated ScopeLogs scope_logs = 2;
  string schema_url = 3;
}
```

#### Example Request
```bash
curl -X POST http://otel-collector:4318/v1/logs \
  -H "Content-Type: application/json" \
  -d '{
    "resourceLogs": [
      {
        "resource": {
          "attributes": [
            {
              "key": "service.name",
              "value": {
                "stringValue": "example-service"
              }
            }
          ]
        },
        "scopeLogs": [
          {
            "scope": {
              "name": "example-logger",
              "version": "1.0.0"
            },
            "logRecords": [
              {
                "timeUnixNano": "1641945600000000000",
                "severityNumber": 9,
                "severityText": "INFO",
                "body": {
                  "stringValue": "Example log message"
                },
                "attributes": [
                  {
                    "key": "log.level",
                    "value": {
                      "stringValue": "info"
                    }
                  }
                ]
              }
            ]
          }
        ]
      }
    ]
  }'
```

## Prometheus HTTP API

### Overview

The Prometheus HTTP API provides programmatic access to Prometheus's metrics and metadata.

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/query` | Instant queries |
| GET | `/api/v1/query_range` | Range queries |
| GET | `/api/v1/series` | Series metadata |
| GET | `/api/v1/label/<label_name>/values` | Label values |
| GET | `/api/v1/targets` | Target discovery |
| GET | `/api/v1/rules` | Alerting rules |
| GET | `/api/v1/alerts` | Active alerts |

### Query API

#### Instant Query
```bash
curl "http://prometheus:9090/api/v1/query?query=up&time=2023-01-01T00:00:00.000Z"
```

Response:
```json
{
  "status": "success",
  "data": {
    "resultType": "vector",
    "result": [
      {
        "metric": {
          "__name__": "up",
          "instance": "localhost:9090",
          "job": "prometheus"
        },
        "value": [
          1672531200,
          "1"
        ]
      }
    ]
  }
}
```

#### Range Query
```bash
curl "http://prometheus:9090/api/v1/query_range?query=up&start=2023-01-01T00:00:00.000Z&end=2023-01-01T01:00:00.000Z&step=60"
```

#### Series API
```bash
curl "http://prometheus:9090/api/v1/series?match[]=up&start=2023-01-01T00:00:00.000Z&end=2023-01-01T01:00:00.000Z"
```

### Metadata API

#### Label Values
```bash
curl "http://prometheus:9090/api/v1/label/job/values"
```

Response:
```json
{
  "status": "success",
  "data": [
    "prometheus",
    "node",
    "grafana"
  ]
}
```

#### Targets
```bash
curl "http://prometheus:9090/api/v1/targets"
```

Response:
```json
{
  "status": "success",
  "data": {
    "activeTargets": [
      {
        "discoveredLabels": {
          "__address__": "localhost:9090",
          "__metrics_path__": "/metrics",
          "__scheme__": "http",
          "job": "prometheus"
        },
        "labels": {
          "instance": "localhost:9090",
          "job": "prometheus"
        },
        "scrapePool": "prometheus",
        "scrapeUrl": "http://localhost:9090/metrics",
        "lastError": "",
        "lastScrape": "2023-01-01T00:00:00.000Z",
        "lastScrapeDuration": 0.001,
        "health": "up"
      }
    ],
    "droppedTargets": []
  }
}
```

## Grafana HTTP API

### Overview

The Grafana HTTP API allows programmatic access to dashboards, datasources, and administrative functions.

### Authentication

```bash
curl -H "Authorization: Bearer <API_KEY>" http://grafana:3000/api/dashboards
```

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/dashboards` | List dashboards |
| POST | `/api/dashboards/db` | Create dashboard |
| GET | `/api/dashboards/uid/{uid}` | Get dashboard by UID |
| DELETE | `/api/dashboards/uid/{uid}` | Delete dashboard |
| GET | `/api/datasources` | List datasources |
| POST | `/api/datasources` | Create datasource |
| GET | `/api/folders` | List folders |
| POST | `/api/folders` | Create folder |

### Dashboard API

#### List Dashboards
```bash
curl -H "Authorization: Bearer <API_KEY>" \
  "http://grafana:3000/api/search?query=&type=dash-db"
```

#### Create Dashboard
```bash
curl -X POST \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "dashboard": {
      "id": null,
      "title": "Example Dashboard",
      "tags": ["example"],
      "timezone": "browser",
      "panels": [
        {
          "id": 1,
          "title": "Panel Title",
          "type": "graph",
          "gridPos": {
            "h": 8,
            "w": 12,
            "x": 0,
            "y": 0
          },
          "targets": [
            {
              "expr": "up",
              "refId": "A"
            }
          ]
        }
      ],
      "time": {
        "from": "now-1h",
        "to": "now"
      },
      "timepicker": {},
      "templating": {
        "list": []
      },
      "annotations": {
        "list": []
      },
      "refresh": "5s",
      "schemaVersion": 27,
      "version": 0,
      "links": []
    },
    "folderId": 0,
    "overwrite": false
  }' \
  http://grafana:3000/api/dashboards/db
```

#### Get Dashboard
```bash
curl -H "Authorization: Bearer <API_KEY>" \
  http://grafana:3000/api/dashboards/uid/example-dashboard
```

### Datasource API

#### List Datasources
```bash
curl -H "Authorization: Bearer <API_KEY>" \
  http://grafana:3000/api/datasources
```

#### Create Datasource
```bash
curl -X POST \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Prometheus",
    "type": "prometheus",
    "url": "http://prometheus:9090",
    "access": "proxy",
    "isDefault": true
  }' \
  http://grafana:3000/api/datasources
```

## Loki HTTP API

### Overview

The Loki HTTP API provides programmatic access to log data and metadata.

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/loki/api/v1/query` | Instant log queries |
| GET | `/loki/api/v1/query_range` | Range log queries |
| GET | `/loki/api/v1/series` | Series metadata |
| GET | `/loki/api/v1/labels` | Available labels |
| GET | `/loki/api/v1/label/<name>/values` | Label values |

### Query API

#### Instant Query
```bash
curl "http://loki:3100/loki/api/v1/query?query={job=\"example\"}&limit=100"
```

Response:
```json
{
  "status": "success",
  "data": {
    "resultType": "streams",
    "result": [
      {
        "stream": {
          "job": "example",
          "level": "info"
        },
        "values": [
          [
            "1641945600000000000",
            "Example log message 1"
          ],
          [
            "1641945660000000000",
            "Example log message 2"
          ]
        ]
      }
    ]
  }
}
```

#### Range Query
```bash
curl "http://loki:3100/loki/api/v1/query_range?query={job=\"example\"}&start=1641945600&end=1641949200&limit=1000"
```

#### Label Queries
```bash
# Get all labels
curl "http://loki:3100/loki/api/v1/labels"

# Get values for a specific label
curl "http://loki:3100/loki/api/v1/label/job/values"
```

## Tempo HTTP API

### Overview

The Tempo HTTP API provides programmatic access to trace data.

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/traces/{traceID}` | Get trace by ID |
| GET | `/api/search` | Search traces |
| GET | `/api/echo` | Health check |

### Trace API

#### Get Trace
```bash
curl "http://tempo:3200/api/traces/5B8EFFF798038103D269B633813FC60C"
```

Response:
```json
{
  "batches": [
    {
      "resource": {
        "attributes": [
          {
            "key": "service.name",
            "value": {
              "stringValue": "example-service"
            }
          }
        ]
      },
      "scopeSpans": [
        {
          "scope": {
            "name": "example-tracer"
          },
          "spans": [
            {
              "traceId": "5B8EFFF798038103D269B633813FC60C",
              "spanId": "05EDD006BC5F551A",
              "name": "example-span",
              "kind": "SPAN_KIND_SERVER",
              "startTimeUnixNano": "1641945600000000000",
              "endTimeUnixNano": "1641945660000000000",
              "attributes": [
                {
                  "key": "http.method",
                  "value": {
                    "stringValue": "GET"
                  }
                }
              ],
              "status": {
                "code": "STATUS_CODE_OK"
              }
            }
          ]
        }
      ]
    }
  ]
}
```

#### Search Traces
```bash
curl "http://tempo:3200/api/search?tags=service.name%3Dexample-service&start=1641945600&end=1641949200&limit=20"
```

## Pyroscope HTTP API

### Overview

The Pyroscope HTTP API provides programmatic access to profiling data.

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/render` | Render profile data |
| POST | `/ingest` | Ingest profile data |
| GET | `/labels` | Get available labels |
| GET | `/label-values` | Get label values |

### Profiling API

#### Render Profile
```bash
curl "http://pyroscope:4040/render?query=example.app.cpu&from=1641945600&until=1641949200&format=json"
```

#### Ingest Profile
```bash
curl -X POST \
  -H "Content-Type: multipart/form-data" \
  -F "profile=@profile.pb.gz" \
  -F "sampleType=CPU" \
  -F "sampleUnit=nanoseconds" \
  -F "labels={\"service\":\"example\"}" \
  http://pyroscope:4040/ingest
```

#### Labels API
```bash
# Get all labels
curl "http://pyroscope:4040/labels"

# Get values for a specific label
curl "http://pyroscope:4040/label-values?label=service"
```

## Error Handling

### Common HTTP Status Codes

| Status Code | Description |
|-------------|-------------|
| 200 | Success |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 405 | Method Not Allowed |
| 429 | Too Many Requests |
| 500 | Internal Server Error |
| 502 | Bad Gateway |
| 503 | Service Unavailable |

### Error Response Format

```json
{
  "status": "error",
  "error": "error message",
  "errorType": "error_type"
}
```

## Rate Limiting

### Default Limits

- **Prometheus**: 10 queries per second per client
- **Grafana**: 30 requests per minute per API key
- **Loki**: 100 queries per second per tenant
- **Tempo**: 100 traces per second
- **Pyroscope**: 10 profiles per second

### Rate Limit Headers

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1641945660
Retry-After: 60
```

## Authentication

### API Key Authentication

```bash
curl -H "Authorization: Bearer <API_KEY>" \
  http://service:port/api/endpoint
```

### Basic Authentication

```bash
curl -u "username:password" \
  http://service:port/api/endpoint
```

### OAuth2 Authentication

```bash
curl -H "Authorization: Bearer <ACCESS_TOKEN>" \
  http://service:port/api/endpoint
```

## SDK Examples

### Python Client

```python
import requests
from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# Configure OTLP exporter
otlp_exporter = OTLPSpanExporter(
    endpoint="otel-collector:4317",
    insecure=True
)

# Set up tracing
trace.set_tracer_provider(TracerProvider())
tracer = trace.get_tracer(__name__)
trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(otlp_exporter)
)

# Create a span
with tracer.start_as_current_span("example-span") as span:
    span.set_attribute("example.key", "example.value")
    print("Hello, World!")
```

### JavaScript Client

```javascript
const { NodeTracerProvider } = require('@opentelemetry/sdk-trace-node');
const { OTLPTraceExporter } = require('@opentelemetry/exporter-otlp-grpc');
const { BatchSpanProcessor } = require('@opentelemetry/sdk-trace-base');

// Configure OTLP exporter
const exporter = new OTLPTraceExporter({
  url: 'http://otel-collector:4317',
});

// Set up tracing
const provider = new NodeTracerProvider();
provider.addSpanProcessor(new BatchSpanProcessor(exporter));
provider.register();

// Create a span
const tracer = provider.getTracer('example-tracer');
const span = tracer.startSpan('example-span');
span.setAttribute('example.key', 'example.value');
span.end();
```

### Go Client

```go
package main

import (
    "context"
    "log"

    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracegrpc"
    "go.opentelemetry.io/otel/sdk/trace"
)

func main() {
    // Configure OTLP exporter
    exporter, err := otlptracegrpc.New(
        context.Background(),
        otlptracegrpc.WithEndpoint("otel-collector:4317"),
        otlptracegrpc.WithInsecure(),
    )
    if err != nil {
        log.Fatal(err)
    }

    // Set up tracing
    tp := trace.NewTracerProvider(
        trace.WithBatcher(exporter),
    )
    otel.SetTracerProvider(tp)

    // Create a span
    tracer := otel.Tracer("example-tracer")
    _, span := tracer.Start(context.Background(), "example-span")
    span.SetAttributes(attribute.String("example.key", "example.value"))
    span.End()
}
```