# Incident Protocol - Microservices MemoryCore
*Incident response and troubleshooting procedures for microservices systems*

## Protocol Activation

**Trigger**: User says "incident", system is down, or multiple errors detected

**Priority Levels**:
| Level | Description | Response Time |
|-------|-------------|---------------|
| P1 - Critical | System-wide outage, data loss risk | Immediate |
| P2 - High | Major feature broken, one service down | Within minutes |
| P3 - Medium | Degraded performance, intermittent errors | Within hours |
| P4 - Low | Minor issue, cosmetic, non-blocking | Next session |

## Immediate Response (First 5 Minutes)

### Step 1: Assess Impact
```
[ ] What is broken? (Which endpoint / feature / flow)
[ ] Who is affected? (All users / specific users / internal only)
[ ] When did it start? (Timestamp of first report)
[ ] Is it getting worse? (Spreading / stable / recovering)
```

### Step 2: Quick Health Check
```bash
# Check all services at once
echo "=== Service Health ==="
for port in [PORT_1] [PORT_2] [PORT_3] [PORT_4] [PORT_5]; do
  echo "Port $port: $(curl -s -o /dev/null -w '%{http_code}' http://localhost:$port/health)"
done

echo "=== Infrastructure ==="
docker-compose ps

echo "=== Recent Errors ==="
docker-compose logs --since=5m 2>&1 | grep -i "error\|fatal\|panic"
```

### Step 3: Identify Blast Radius
```
Affected Service(s): [List]
Affected Downstream: [Services that depend on affected service]
Affected Upstream:   [Services that call affected service]
Data at Risk:        [Any data that could be lost/corrupted]
```

## Diagnosis Phase

### Common Incident Patterns

#### Pattern 1: Single Service Down
```
Symptom: One service not responding
Check:
  1. docker-compose ps [service] — Is container running?
  2. docker-compose logs [service] — Any panic/fatal errors?
  3. Container restart count — Is it crash-looping?
  4. Memory/CPU usage — Resource exhaustion?
Fix:
  - docker-compose restart [service]
  - If crash-looping: check logs, fix code, rebuild
  - If resource issue: increase limits
```

#### Pattern 2: Cascade Failure
```
Symptom: Multiple services failing in sequence
Check:
  1. Which service failed FIRST? (Check timestamps)
  2. Is it a shared dependency? (Database, broker, cache)
  3. Are circuit breakers tripping?
  4. Is there a retry storm amplifying the failure?
Fix:
  - Fix the root cause service first
  - Other services should recover automatically
  - If not: restart dependent services in dependency order
```

#### Pattern 3: Database Unreachable
```
Symptom: Services get connection errors
Check:
  1. docker-compose ps [database] — Container running?
  2. Can connect manually? — psql connection test
  3. Connection pool exhausted? — Check pool config
  4. Disk space? — docker system df
  5. Too many connections? — Check max_connections
Fix:
  - Restart database container
  - If data volume corrupted: restore from backup
  - If pool exhausted: restart services to reset pools
```

#### Pattern 4: Message Broker Failure
```
Symptom: Events not being processed, consumers lagging
Check:
  1. docker-compose ps [broker] — Container running?
  2. Broker management UI — Topic health?
  3. Consumer group status — Lag increasing?
  4. ZooKeeper (if Kafka) — Running?
  5. Disk space for broker logs
Fix:
  - Restart broker (consumers will resume from last offset)
  - If ZooKeeper issue: restart ZooKeeper first, then broker
  - Check for poison messages in DLQ
```

#### Pattern 5: API Gateway Not Routing
```
Symptom: All requests return 502/503
Check:
  1. Gateway container running?
  2. Upstream services reachable from gateway network?
  3. Gateway config correct? (routes, ports)
  4. DNS resolution working within Docker network?
Fix:
  - Restart gateway
  - Verify Docker network connectivity
  - Check gateway route configuration
```

#### Pattern 6: Out of Memory / Disk
```
Symptom: Containers being killed, OOM errors
Check:
  1. docker stats — Memory usage per container
  2. docker system df — Disk usage
  3. Database WAL/log files — Growing unbounded?
  4. Broker log segments — Retention policy correct?
Fix:
  - docker system prune — Remove unused images/volumes
  - Increase container memory limits
  - Set proper log rotation and retention
```

## Recovery Procedures

### Service Recovery Order
*When restarting the system, follow this order:*

```
1. Infrastructure (databases, brokers, cache)
   Wait: Until healthy and accepting connections

2. Core Services (auth, user management)
   Wait: Until health checks pass

3. Business Services (orders, payments, etc.)
   Wait: Until health checks pass

4. API Gateway
   Wait: Until all upstream services routable

5. Background Workers / Consumers
   Wait: Until processing resumes
```

### Full System Restart
```bash
# 1. Stop everything
docker-compose down

# 2. Start infrastructure
docker-compose up -d [database] [broker] [cache]
sleep 10  # Wait for infrastructure

# 3. Start core services
docker-compose up -d [auth-service]
sleep 5

# 4. Start business services
docker-compose up -d [service-1] [service-2] [service-3]
sleep 5

# 5. Start gateway
docker-compose up -d [gateway]

# 6. Verify
docker-compose ps
curl http://localhost:[GATEWAY_PORT]/health
```

### Data Recovery
```
IF data inconsistency detected:
  1. Identify which service has correct data (source of truth)
  2. Check event logs for missed events
  3. Replay events if needed (re-publish to topic)
  4. Manual data fix as last resort
  5. Document what happened and why
```

## Post-Incident

### Incident Report Template
```markdown
## Incident Report

**Date**: [DATE]
**Duration**: [START] to [END] ([DURATION])
**Severity**: [P1/P2/P3/P4]
**Affected Services**: [List]
**Impact**: [What users experienced]

### Timeline
- [TIME] — First symptom detected
- [TIME] — Root cause identified
- [TIME] — Fix applied
- [TIME] — Service recovered
- [TIME] — Incident closed

### Root Cause
[Description of what caused the incident]

### Resolution
[What was done to fix it]

### Action Items
- [ ] [Preventive measure 1]
- [ ] [Preventive measure 2]
- [ ] [Monitoring improvement]
```

### Memory Updates After Incident
```
- [ ] Update debug-protocol.md if new pattern discovered
- [ ] Update infrastructure.md if config changed
- [ ] Update service-registry.md if service config changed
- [ ] Add to architecture-core.md Technical Debt if applicable
- [ ] Update current-session.md with incident context
```

## Incident Quick Reference Card

```
ASSESS -> DIAGNOSE -> FIX -> VERIFY -> DOCUMENT

1. ASSESS:  What's broken? Who's affected? Since when?
2. DIAGNOSE: Check health, check logs, trace the failure
3. FIX:     Apply fix (restart, config change, code fix)
4. VERIFY:  Health checks pass, users can use the system
5. DOCUMENT: Incident report, update memory files
```

---

**Protocol Status**: Ready
**Activation**: "incident" or during system failures
**Philosophy**: Assess first, fix root cause, document everything

*Follow this protocol to systematically respond to and recover from microservices incidents!*
