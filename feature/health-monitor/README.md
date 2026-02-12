# Health Monitor - Feature Extension
*Health check patterns and system monitoring memory*

## What This Does

Tracks the health status and monitoring configuration of all microservices, enabling:
- **Quick Health Checks**: Instantly know the status of all services
- **Monitoring Config**: Remember all metrics, alerts, and dashboards
- **SLA Tracking**: Monitor uptime and performance targets
- **Alert Management**: Know what triggers alerts and how to respond

## Quick Setup

1. Load this feature: `"health check"` or `"load health"`
2. AI reads `health-dashboard.md` and knows your monitoring setup
3. Ask questions like:
   - "Are all services healthy?"
   - "What's the SLA for payment-service?"
   - "What alerts are configured?"
   - "Show me the health dashboard"

## Commands

| Command | Action |
|---------|--------|
| `health check` | Check health of all services |
| `load health` | Load full health monitoring config |
| `health [service]` | Health status of specific service |
| `alerts` | Show all configured alerts |
| `sla status` | Show SLA compliance |

## When to Update

Update `health-dashboard.md` when:
- Health check endpoints change
- New monitoring metrics are added
- Alert thresholds are adjusted
- SLA targets change
- New dashboards are created
- Monitoring tools change
