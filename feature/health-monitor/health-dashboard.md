# Health Dashboard - [PROJECT_NAME]
*Health check configuration, monitoring, and SLA tracking*

## Health Check Endpoints

| # | Service | Health URL | Expected Response | Timeout |
|---|---------|-----------|-------------------|---------|
| 1 | [SERVICE_1] | `http://localhost:[PORT]/health` | 200 OK | 5s |
| 2 | [SERVICE_2] | `http://localhost:[PORT]/health` | 200 OK | 5s |
| 3 | [SERVICE_3] | `http://localhost:[PORT]/health` | 200 OK | 5s |
| 4 | [SERVICE_4] | `http://localhost:[PORT]/health` | 200 OK | 5s |
| 5 | [SERVICE_5] | `http://localhost:[PORT]/health` | 200 OK | 5s |
| 6 | [API_GATEWAY] | `http://localhost:[PORT]/health` | 200 OK | 5s |

### Health Check Response Format
```json
{
  "status": "healthy",
  "service": "[SERVICE_NAME]",
  "version": "[VERSION]",
  "uptime": "[DURATION]",
  "checks": {
    "database": "connected",
    "broker": "connected",
    "cache": "connected"
  }
}
```

### Quick Health Check Script
```bash
#!/bin/bash
echo "=== [PROJECT_NAME] Health Dashboard ==="
echo ""

services=(
  "[SERVICE_1]:[PORT_1]"
  "[SERVICE_2]:[PORT_2]"
  "[SERVICE_3]:[PORT_3]"
  "[SERVICE_4]:[PORT_4]"
  "[SERVICE_5]:[PORT_5]"
  "[GATEWAY]:[GATEWAY_PORT]"
)

for svc in "${services[@]}"; do
  name="${svc%%:*}"
  port="${svc##*:}"
  status=$(curl -s -o /dev/null -w '%{http_code}' "http://localhost:$port/health" 2>/dev/null)
  if [ "$status" = "200" ]; then
    echo "  OK   $name (port $port)"
  else
    echo "  DOWN $name (port $port) - HTTP $status"
  fi
done

echo ""
echo "=== Infrastructure ==="
docker-compose ps --format "table {{.Name}}\t{{.Status}}" 2>/dev/null
```

## Service Level Objectives (SLO)

| Service | Availability | Latency p95 | Latency p99 | Error Rate |
|---------|-------------|-------------|-------------|------------|
| [SERVICE_1] | [99.9%] | [<200ms] | [<500ms] | [<0.1%] |
| [SERVICE_2] | [99.9%] | [<200ms] | [<500ms] | [<0.1%] |
| [SERVICE_3] | [99.5%] | [<500ms] | [<1s] | [<0.5%] |
| [API_GATEWAY] | [99.99%] | [<50ms] | [<100ms] | [<0.01%] |

## Key Metrics

### Per-Service Metrics
| Metric | Description | Alert Threshold |
|--------|-------------|----------------|
| `http_requests_total` | Total HTTP requests | — |
| `http_request_duration_seconds` | Request latency histogram | p99 > [THRESHOLD] |
| `http_requests_errors_total` | Total 5xx errors | rate > [THRESHOLD]/min |
| `db_connections_active` | Active DB connections | > [POOL_SIZE * 0.8] |
| `db_query_duration_seconds` | Database query latency | p99 > [THRESHOLD] |
| `broker_messages_produced` | Events published | — |
| `broker_messages_consumed` | Events consumed | — |
| `broker_consumer_lag` | Consumer group lag | > [THRESHOLD] messages |

### Infrastructure Metrics
| Metric | Component | Alert Threshold |
|--------|-----------|----------------|
| CPU usage | All containers | > 80% sustained |
| Memory usage | All containers | > 85% |
| Disk usage | Database volumes | > 80% |
| Connection count | Database | > [max_connections * 0.8] |
| Broker disk | Message broker | > 80% |
| Container restarts | All services | > 3 in 5 minutes |

## Alert Configuration

### Critical Alerts (P1 - Immediate)
| Alert | Condition | Action |
|-------|-----------|--------|
| Service Down | Health check fails 3x consecutive | Check service, restart if needed |
| Database Down | DB health check fails | Check DB container, connections |
| Broker Down | Broker health check fails | Check broker, restart |
| Error Rate Spike | 5xx rate > [N]/min for 5 min | Check logs, identify cause |
| Data Loss Risk | DB replication lag > [N]s | Check replication, network |

### Warning Alerts (P2 - Soon)
| Alert | Condition | Action |
|-------|-----------|--------|
| High Latency | p99 > [THRESHOLD] for 10 min | Check slow queries, dependencies |
| Consumer Lag | Lag > [N] messages for 10 min | Check consumer, scale if needed |
| Memory High | > 85% for 15 min | Check for leaks, increase limit |
| Disk Space | > 80% usage | Prune old data, increase volume |
| Certificate Expiry | < 30 days to expiry | Renew certificate |

### Informational Alerts (P3 - FYI)
| Alert | Condition | Action |
|-------|-----------|--------|
| Deployment | Service restarted | Verify new version healthy |
| Traffic Spike | Request rate > 2x normal | Monitor, scale if needed |
| New Error Type | Previously unseen error code | Investigate in next session |

## Monitoring Tools

### Dashboards
| Dashboard | URL | Purpose |
|-----------|-----|---------|
| [Main Dashboard] | [URL] | System overview |
| [Service Detail] | [URL] | Per-service deep dive |
| [Infrastructure] | [URL] | DB, broker, cache metrics |
| [Broker Monitor] | [URL] | Kafka/RabbitMQ management |

### Log Access
| Tool | URL/Command | Purpose |
|------|------------|---------|
| [Log Aggregator] | [URL] | Centralized log search |
| Docker logs | `docker-compose logs -f [svc]` | Local log tailing |
| [Tracing UI] | [URL] | Distributed tracing |

## Health Status Template (For Session Memory)

When checking health, update `current-session.md` with:
```markdown
## System Health (as of [TIMESTAMP])
| Service | Status | Notes |
|---------|--------|-------|
| [SVC_1] | OK/DOWN | [Any issues] |
| [SVC_2] | OK/DOWN | [Any issues] |
| Infrastructure: [OK/ISSUES]
| Last Incident: [Date and brief description]
```

---

**Version**: Health Dashboard v1.0
**Last Updated**: [DATE]
**Status**: Template — requires project-specific configuration

*This file tracks the health and monitoring of your entire microservices system!*
