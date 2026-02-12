# Dependency Tracker - Feature Extension
*Service dependency graph and impact analysis*

## What This Does

Tracks and visualizes how microservices depend on each other, enabling:
- **Impact Analysis**: Know which services are affected before making changes
- **Deployment Order**: Understand correct startup/shutdown sequence
- **Failure Propagation**: Predict cascade failures
- **Architecture Review**: Identify tight coupling and circular dependencies

## Quick Setup

1. Load this feature: `"load dependencies"`
2. AI reads `dependency-map.md` and understands your service graph
3. Ask questions like:
   - "What depends on user-service?"
   - "If payment-service goes down, what breaks?"
   - "Show me the dependency graph"
   - "Are there circular dependencies?"

## Commands

| Command | Action |
|---------|--------|
| `load dependencies` | Load dependency map |
| `impact [service]` | Show all services affected if this service fails |
| `depends on [service]` | Show what this service depends on |
| `depended by [service]` | Show what depends on this service |
| `check circular` | Check for circular dependencies |

## When to Update

Update `dependency-map.md` when:
- A new service is added
- A service starts calling another service
- An event consumer is added
- A service is deprecated or removed
- Inter-service communication pattern changes
