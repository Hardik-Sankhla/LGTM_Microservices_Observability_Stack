# Examples Documentation

This section provides comprehensive examples of instrumenting applications with OpenTelemetry across multiple programming languages and frameworks.

## 📋 Examples Overview

The LGTM Stack includes fully instrumented example applications in 5 languages:

| Language | Framework | Port | Features |
|----------|-----------|------|----------|
| **[Go](go.md)** | Standard Library | 8081 | Manual instrumentation, HTTP server |
| **[Java](java.md)** | Spring Boot | 8080 | Auto-instrumentation, REST API |
| **[Node.js](nodejs.md)** | Express | 8084 | SDK instrumentation, middleware |
| **[Python](python.md)** | Flask | 8082 | Manual instrumentation, WSGI |
| **[.NET](dotnet.md)** | ASP.NET Core | 8083 | Auto-instrumentation, MVC |

## 🎯 Common Features

All examples implement the same **rolldice** service with consistent features:

### Core Functionality
- **HTTP Endpoint**: `GET /rolldice` returns random number (1-6)
- **Optional Player Parameter**: `GET /rolldice?player=Alice`
- **JSON Response**: `{"result": 4}`

### Telemetry Instrumentation
- **Traces**: Request/response spans with context
- **Metrics**: HTTP request counts, durations, error rates
- **Logs**: Structured logging with correlation IDs
- **Profiles**: Performance profiling (where supported)

### Service Metadata
- **Service Name**: `rolldice-{language}`
- **Service Version**: `1.0.0`
- **Resource Attributes**: Language, framework, environment

## 🚀 Running Examples

### Prerequisites
```bash
# Start the LGTM Stack first
./run-lgtm.sh

# Verify stack is ready
curl http://localhost:3000/api/health
```

### Individual Examples

#### Go Example
```bash
cd examples/go
go mod tidy
go run main.go otel.go rolldice.go
# Access: http://localhost:8081/rolldice
```

#### Java Example
```bash
cd examples/java
./mvnw spring-boot:run
# Access: http://localhost:8080/rolldice
```

#### Node.js Example
```bash
cd examples/nodejs
npm install
npm start
# Access: http://localhost:8084/rolldice
```

#### Python Example
```bash
cd examples/python
pip install -r requirements.txt
python app.py
# Access: http://localhost:8082/rolldice
```

#### .NET Example
```bash
cd examples/dotnet
dotnet run
# Access: http://localhost:8083/rolldice
```

### Using Convenience Scripts
```bash
# Run all examples (requires multiple terminals)
./run-example.sh  # Python example
# Then manually run others in separate terminals
```

## 📊 Generated Telemetry

### Metrics (Prometheus)
```promql
# Request rate by service
rate(http_requests_total[5m])

# Error rate
rate(http_requests_total{status_code=~"5.."}[5m])

# 95th percentile latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
```

### Traces (Tempo)
- **Service Graph**: Shows service dependencies
- **Trace Search**: Find traces by service, operation, or tags
- **Span Details**: View individual span timing and attributes

### Logs (Loki)
```logql
# All application logs
{job="rolldice"}

# Logs by language
{job="rolldice-go"}
{job="rolldice-java"}

# Error logs
{job="rolldice"} |= "ERROR"
```

### Profiles (Pyroscope)
- **Flame Graphs**: Performance visualization
- **CPU Profiling**: Time spent in functions
- **Memory Profiling**: Memory allocation patterns

## 🔍 Grafana Dashboards

### Pre-built Dashboards
1. **RED Metrics**: Request, Error, Duration metrics
2. **JVM Metrics**: Java-specific performance metrics
3. **Service Graph**: Service dependency visualization

### Custom Queries

#### Request Rate Dashboard
```promql
sum(rate(http_requests_total[5m])) by (service_name)
```

#### Error Rate Dashboard
```promql
sum(rate(http_requests_total{status_code=~"5.."}[5m])) by (service_name)
```

#### Latency Dashboard
```promql
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service_name))
```

## 🏗️ Instrumentation Patterns

### Manual Instrumentation (Go, Python)
```go
// Go manual instrumentation
tracer := otel.Tracer("rolldice")
_, span := tracer.Start(ctx, "roll_dice")
defer span.End()

span.SetAttributes(
    attribute.Int("result", result),
    attribute.String("player", player),
)
```

```python
# Python manual instrumentation
with tracer.start_as_span("roll_dice") as span:
    span.set_attribute("result", result)
    span.set_attribute("player", player)
    logger.info("Dice rolled", extra={"result": result})
```

### Auto-Instrumentation (Java, .NET, Node.js)
```java
// Java auto-instrumentation (no code changes needed)
// Instrumentation happens automatically via agent
```

```javascript
// Node.js SDK instrumentation
const sdk = new NodeSDK({
  traceExporter: new OTLPTraceExporter(),
  instrumentations: [getNodeAutoInstrumentations()]
})
```

### Middleware Integration
```javascript
// Express middleware
app.use((req, res, next) => {
  const span = trace.getSpan(context.active())
  span?.setAttribute('http.method', req.method)
  next()
})
```

## 🔄 Cross-Service Communication

### Service Calls
```bash
# Call between services (example)
curl "http://localhost:8080/rolldice?player=ServiceA"
# This creates traces showing service dependencies
```

### Distributed Tracing
- **Trace Context Propagation**: Headers passed between services
- **Service Graph Generation**: Automatic dependency mapping
- **End-to-End Visibility**: Complete request flows

## 📈 Load Testing

### Generate Traffic
```bash
# Use the traffic generator
./generate-traffic.sh

# Or manual testing
for i in {1..100}; do
  curl "http://localhost:8080/rolldice?player=TestUser$i" &
done
```

### Monitoring Load
```bash
# Watch metrics increase
curl http://localhost:9090/api/v1/query?query=http_requests_total

# View live traces
# Open Tempo UI at http://localhost:3200
```

## 🐛 Troubleshooting Examples

### Application Won't Start
```bash
# Check dependencies
cd examples/java && ./mvnw dependency:tree

# Check ports
netstat -tulpn | grep :8080

# Check logs
tail -f examples/java/logs/application.log
```

### No Telemetry Data
```bash
# Check OTLP connectivity
curl -X POST http://localhost:4318/v1/traces \
  -H "Content-Type: application/json" \
  -d '{"resourceSpans":[]}'

# Verify service configuration
curl http://localhost:8888/metrics | grep otelcol
```

### Grafana Shows No Data
```bash
# Check data source health
curl http://localhost:3000/api/datasources

# Verify queries work
curl "http://localhost:9090/api/v1/query?query=up"
```

## 🔧 Customization Examples

### Adding Custom Metrics
```python
# Python custom metrics
from opentelemetry.metrics import get_meter

meter = get_meter("rolldice")
roll_counter = meter.create_counter("dice_rolls_total")
roll_histogram = meter.create_histogram("roll_duration_seconds")

# Use in code
roll_counter.add(1, {"face": result})
```

### Adding Custom Attributes
```go
// Go custom attributes
span.SetAttributes(
    attribute.String("game.type", "dice"),
    attribute.Bool("player.vip", isVIP),
    attribute.Float64("roll.probability", 1.0/6.0),
)
```

### Custom Log Correlation
```javascript
// Node.js log correlation
const span = trace.getSpan(context.active())
const traceId = span?.spanContext().traceId

logger.info('Dice rolled', {
  traceId,
  result,
  player,
  timestamp: new Date().toISOString()
})
```

## 🚀 Advanced Examples

### Multi-Service Architecture
```bash
# Run multiple examples simultaneously
cd examples/go && go run . &  # Port 8081
cd examples/java && ./mvnw spring-boot:run &  # Port 8080
cd examples/nodejs && npm start &  # Port 8084

# Generate cross-service traffic
curl "http://localhost:8080/rolldice?player=MultiService"
```

### External Service Integration
```bash
# Configure external OTLP export
export OTEL_EXPORTER_OTLP_ENDPOINT=https://api.honeycomb.io:443
export OTEL_EXPORTER_OTLP_HEADERS="x-honeycomb-team=your-api-key"

# Restart examples to send data externally
```

## 📚 Learning Path

### Beginner
1. **Start the Stack**: `./run-lgtm.sh`
2. **Run One Example**: Python or Node.js (easiest)
3. **Explore Grafana**: View dashboards and data
4. **Try Queries**: Use Prometheus, Loki, Tempo UIs

### Intermediate
1. **Run Multiple Examples**: All 5 languages
2. **Generate Traffic**: Use the traffic generator
3. **Custom Dashboards**: Create your own visualizations
4. **Cross-Service Tracing**: See service dependencies

### Advanced
1. **Instrumentation Deep Dive**: Study each language's approach
2. **Custom Metrics**: Add business-specific telemetry
3. **External Export**: Send data to cloud providers
4. **Production Patterns**: Apply to real applications

## 🎯 Best Practices Demonstrated

### Instrumentation
- **Consistent Naming**: Use clear span/metric names
- **Proper Context**: Propagate trace context
- **Resource Attributes**: Identify services clearly
- **Error Handling**: Capture exceptions properly

### Observability
- **RED Metrics**: Rate, Error, Duration
- **USE Metrics**: Utilization, Saturation, Errors
- **Structured Logging**: Consistent log formats
- **Trace Correlation**: Link logs to traces

### Code Quality
- **Error Handling**: Graceful failure modes
- **Configuration**: Environment-based config
- **Dependencies**: Proper dependency management
- **Documentation**: Clear code comments

---

The examples provide practical, working code for learning OpenTelemetry. Start with one language, then explore others to understand different instrumentation approaches. For detailed language-specific guides, select from the examples above. For custom instrumentation, see the [Instrumentation Guide](../api/integration.md). 🎯📚