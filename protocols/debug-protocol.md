# Debug Protocol - Microservices MemoryCore
*Systematic guide for debugging distributed microservices issues*

## Debug Philosophy

**Microservices debugging is different from monolith debugging.** A single user request may traverse 3-5 services, any of which could fail. This protocol provides a systematic approach to tracing issues across service boundaries.

## Quick Debug Commands

```
"debug [service-name]"    -> Load debug context for specific service
"trace [request-id]"      -> Follow request across services
"check health"            -> Health status of all services
"show logs [service]"     -> Recent logs for a service
"debug event [event-name]" -> Debug async event flow
```

## Step 1: Identify the Symptom

### Classification
| Symptom Type | Example | Likely Cause |
|-------------|---------|-------------|
| HTTP 4xx | 400, 401, 403, 404 | Client error, auth, routing |
| HTTP 5xx | 500, 502, 503 | Server error, downstream failure |
| Timeout | Request hangs | Slow dependency, deadlock |
| Data inconsistency | Stale/wrong data | Event processing lag, race condition |
| Connection refused | Cannot reach service | Service down, wrong port |
| Event not processed | Consumer silent | Broker issue, deserialization error |

### Initial Questions
1. **Which service returns the error?** (Check API gateway logs first)
2. **Is it consistent or intermittent?**
3. **When did it start?** (Recent deployment? Config change?)
4. **Does it affect all users or specific ones?**
5. **Are other services healthy?**

## Step 2: Check Service Health

### Health Check All Services
```bash
# Check each service health endpoint
curl http://localhost:[PORT_1]/health  # [SERVICE_1]
curl http://localhost:[PORT_2]/health  # [SERVICE_2]
curl http://localhost:[PORT_3]/health  # [SERVICE_3]
curl http://localhost:[GATEWAY_PORT]/health  # API Gateway
```

### Check Infrastructure
```bash
# Database connectivity
docker-compose ps [database-container]
psql -h localhost -p [DB_PORT] -U [USER] -c "SELECT 1;"

# Message broker
docker-compose ps [broker-container]
# Kafka: check topic list
kafka-topics --list --bootstrap-server localhost:[BROKER_PORT]

# Container status
docker-compose ps
```

### Common Infrastructure Issues
| Symptom | Check | Fix |
|---------|-------|-----|
| Connection refused | Container running? | `docker-compose up -d [service]` |
| Port already in use | Port conflict? | `netstat -tlnp \| grep [PORT]` |
| Out of memory | Container resources? | Check Docker memory limits |
| Disk full | Docker volumes? | `docker system df` + prune |

## Step 3: Trace the Request

### Request Flow Tracing
```
CLIENT REQUEST
      |
[API GATEWAY] -- Check: Does request reach gateway?
      |            Log: Gateway access log
      |
[TARGET SERVICE] -- Check: Does service receive request?
      |               Log: Service request log
      |
[DATABASE] -- Check: Is query executing?
      |         Log: Slow query log
      |
[DEPENDENT SERVICE] -- Check: Is inter-service call succeeding?
      |                  Log: HTTP client log
      |
[MESSAGE BROKER] -- Check: Is event published?
      |               Log: Producer log
      |
[CONSUMER SERVICE] -- Check: Is event consumed?
                       Log: Consumer log
```

### Log Correlation
```bash
# Search logs by request ID
docker-compose logs [service] | grep "[REQUEST_ID]"

# Search logs by correlation ID
docker-compose logs [service] | grep "[CORRELATION_ID]"

# Tail logs in real-time
docker-compose logs -f [service]

# Search across all services
docker-compose logs | grep "[SEARCH_TERM]"
```

## Step 4: Common Debug Patterns

### Pattern: Service-to-Service Call Fails
```
Symptom: Service A gets 5xx when calling Service B
Debug:
  1. Is Service B healthy? -> curl http://localhost:[PORT_B]/health
  2. Is Service B reachable? -> Can A resolve B's hostname?
  3. Check Service A's HTTP client timeout settings
  4. Check Service B's logs for errors
  5. Check network: are they on the same Docker network?
```

### Pattern: Event Not Being Processed
```
Symptom: Publisher sends event, consumer never processes it
Debug:
  1. Is event actually published? -> Check producer logs
  2. Is broker healthy? -> Check broker container
  3. Is consumer subscribed to correct topic? -> Check consumer config
  4. Is consumer group registered? -> Check broker management UI
  5. Is deserialization failing? -> Check consumer error logs
  6. Is consumer lagging? -> Check consumer group offset
  7. Is event in DLQ? -> Check dead letter queue
```

### Pattern: Data Inconsistency Across Services
```
Symptom: Service A has data that Service B doesn't
Debug:
  1. Was the event published? -> Check event producer logs
  2. Was the event consumed? -> Check consumer logs
  3. Is there processing delay? -> Check consumer lag
  4. Did consumer processing fail? -> Check for errors
  5. Is the event idempotent? -> Check for duplicate processing
  6. Check event ordering -> Partition key correct?
```

### Pattern: Database Connection Issues
```
Symptom: Service fails with "connection refused" or pool exhausted
Debug:
  1. Is database running? -> docker-compose ps [db]
  2. Can you connect manually? -> psql connection test
  3. Is connection pool exhausted? -> Check pool size config
  4. Is database accepting connections? -> Check max_connections
  5. Are credentials correct? -> Check environment variables
  6. Is database initialized? -> Check if tables exist
```

### Pattern: Request Timeout
```
Symptom: Request takes too long and times out
Debug:
  1. Which step is slow? -> Add timing logs or trace
  2. Is database query slow? -> Check EXPLAIN ANALYZE
  3. Is downstream service slow? -> Check dependent service latency
  4. Is message broker slow? -> Check broker performance
  5. Is there a deadlock? -> Check database locks
  6. Is there a retry storm? -> Check retry configuration
```

### Pattern: Authentication/Authorization Failure
```
Symptom: 401 Unauthorized or 403 Forbidden
Debug:
  1. Is token present? -> Check Authorization header
  2. Is token valid? -> Decode and check expiry
  3. Is auth service healthy? -> Health check
  4. Is token signature correct? -> Check JWT secret matches
  5. Does user have required role? -> Check token claims
  6. Is CORS configured correctly? -> Check gateway CORS settings
```

## Step 5: Distributed Tracing

### Manual Tracing
```bash
# 1. Get the trace/request ID from the initial response headers
# 2. Search across all service logs
for service in [SERVICE_1] [SERVICE_2] [SERVICE_3]; do
  echo "=== $service ==="
  docker-compose logs $service | grep "[TRACE_ID]"
done
```

### Tracing Tools (If Configured)
- **Jaeger**: `http://localhost:16686`
- **Zipkin**: `http://localhost:9411`
- **OpenTelemetry**: Check collector endpoint

## Step 6: Resolution Patterns

### Quick Fixes
| Issue | Quick Fix |
|-------|-----------|
| Service down | `docker-compose restart [service]` |
| Stale container | `docker-compose up -d --build [service]` |
| DB migration missing | Run migration script |
| Port conflict | Check and stop conflicting process |
| Event backlog | Check consumer, restart if stuck |
| Memory issue | Increase container memory limit |

### When to Escalate
- Multiple services failing simultaneously
- Data corruption detected
- Message broker unresponsive
- Database cluster issue
- Network-level failures

## Debug Checklist

```
[ ] Identify which service returns the error
[ ] Check health of that service
[ ] Check health of its dependencies
[ ] Check infrastructure (DB, broker, cache)
[ ] Check logs with request/trace ID
[ ] Trace the request path across services
[ ] Check recent deployments/changes
[ ] Check event flow if async involved
[ ] Verify configuration/environment variables
[ ] Test fix in isolation before deploying
```

## Debug Context Template (For Session Memory)

When debugging, update `current-session.md` with:
```markdown
## Active Debug Session
- **Issue**: [Description of the problem]
- **Affected Service(s)**: [Service names]
- **Symptom**: [What user/system sees]
- **Hypothesis**: [Current theory]
- **Evidence**: [Log entries, errors, etc.]
- **Steps Taken**: [What was tried]
- **Resolution**: [Fix applied or pending]
```

---

**Protocol Status**: Ready
**Activation**: "debug [service]" or when troubleshooting
**Philosophy**: Systematic, trace-based, infrastructure-aware

*Follow this protocol to efficiently debug distributed microservices issues!*
