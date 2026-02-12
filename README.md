# Microservices MemoryCore - AI Memory Architecture for Microservices

*A markdown-based persistent memory system that helps AI assistants understand, remember, and work with microservices projects across sessions.*

## What This Does

**Microservices MemoryCore** gives your AI assistant deep, persistent knowledge of your microservices architecture. Using simple `.md files` as a database, your AI remembers every service, endpoint, event, database schema, and debugging pattern — across all conversations.

Think of it as a **living architecture document** that your AI reads, writes, and evolves as your system grows.

## Key Features

- **Service Registry**: AI knows every service, its port, database, and responsibility
- **API Catalog**: Complete endpoint reference across all services
- **Event Catalog**: All async events with producers, consumers, and schemas
- **Database Catalog**: Every database, table, and relationship
- **Architecture Patterns**: DDD, CQRS, Saga, Event Sourcing — whatever you use
- **Infrastructure Memory**: Docker, Kubernetes, message brokers, CI/CD
- **Debug Protocols**: Distributed tracing, log correlation, failure patterns
- **Session RAM**: Working memory for current development context
- **Self-Updating**: AI updates memory as your architecture evolves

## Architecture Overview

- **Storage**: Markdown files (.md) as database
- **Memory Types**: Core files (always loaded) + Catalogs (on-demand) + Session RAM
- **Setup**: 2-5 minutes with interactive wizard
- **Updates**: Through natural conversation with AI
- **Compatibility**: Claude, ChatGPT, and other AI systems with file access

## File Structure

```
microservices-memorycore/
├── master-memory.md                   # Entry point & loading system
├── setup-wizard.md                    # Interactive project setup
├── save-protocol.md                   # How AI saves/updates memory
│
├── main/                              # Core components (always loaded)
│   ├── architecture-core.md           # Architecture patterns & conventions
│   ├── service-registry.md            # All services: ports, DBs, tech stack
│   ├── infrastructure.md              # Docker, K8s, brokers, CI/CD
│   └── current-session.md             # RAM-like working memory
│
├── catalog/                           # Detailed catalogs (on-demand)
│   ├── api-catalog.md                 # All REST/gRPC endpoints
│   ├── event-catalog.md               # All async events (Kafka/RabbitMQ/etc.)
│   └── database-catalog.md            # All databases & schemas
│
├── protocols/                         # Action protocols
│   ├── debug-protocol.md              # Distributed debugging guide
│   ├── new-service-protocol.md        # New service scaffolding checklist
│   ├── incident-protocol.md           # Incident response procedures
│   ├── ddd-protocol.md                # Domain-Driven Design patterns
│   └── security-protocol.md           # Auth, OWASP, encryption, secrets
│
└── feature/                           # Optional extensions
    ├── dependency-tracker/             # Service dependency graph
    └── health-monitor/                # Health check patterns
```

## Quick Start

### 1. Copy to Your Project
```bash
cp -r microservices-memorycore/ /path/to/your/project/.ai-memory/
```

### 2. Run Setup Wizard
Tell your AI:
```
Load setup-wizard.md and configure it for my project
```

### 3. Start Using
```
load services          # View all services
load api catalog       # View all endpoints
debug [service-name]   # Debug context for a service
add service            # Register a new service
save                   # Persist all changes
```

## Commands Reference

### Core Commands
| Command | Action |
|---------|--------|
| `load microservices` | Full system restoration from memory |
| `load services` | Load service registry |
| `load architecture` | Load architecture patterns |
| `load infrastructure` | Load infra configuration |

### Catalog Commands
| Command | Action |
|---------|--------|
| `load api catalog` | Load all API endpoints |
| `load events` | Load event catalog |
| `load databases` | Load database schemas |

### Action Commands
| Command | Action |
|---------|--------|
| `add service [name]` | Register a new microservice |
| `add endpoint [service]` | Add API endpoint to catalog |
| `add event [topic]` | Add event to catalog |
| `debug [service]` | Load debug context |
| `save` | Persist all changes to files |

### Protocol Commands
| Command | Action |
|---------|--------|
| `load ddd` | DDD patterns, bounded contexts, aggregates |
| `load security` | Auth, OWASP, encryption, secrets management |
| `new service` | Step-by-step service creation guide |
| `incident [service]` | Incident response protocol |
| `health check` | System health overview |

## How It Works

### Memory Hierarchy

```
ALWAYS LOADED (Core)
├── master-memory.md ......... System entry point
├── architecture-core.md ..... Patterns & conventions
├── service-registry.md ...... Service overview table
└── infrastructure.md ........ Infra configuration

LOADED ON-DEMAND (Catalogs)
├── api-catalog.md ........... When working on APIs
├── event-catalog.md ......... When working on events
└── database-catalog.md ...... When working on data

LOADED ON-DEMAND (Protocols)
├── debug-protocol.md ........ When debugging
├── new-service-protocol.md .. When creating services
├── incident-protocol.md ..... During incidents
├── ddd-protocol.md .......... When designing domains
└── security-protocol.md ..... When reviewing security

SESSION-ONLY (RAM)
└── current-session.md ....... Resets each session
```

### Self-Updating Cycle

```
ARCHITECTURE CHANGE
        |
   AI DETECTS CHANGE
        |
   UPDATES .MD FILES
        |
   MEMORY PERSISTED
        |
   NEXT SESSION RESTORED
```

## Use Cases

- **New team member onboarding**: AI explains your entire architecture instantly
- **Cross-service debugging**: AI correlates logs and traces across services
- **API development**: AI knows all existing endpoints to avoid conflicts
- **Event-driven design**: AI tracks all event flows and suggests patterns
- **Infrastructure changes**: AI remembers all config and deployment details
- **Incident response**: AI guides you through debugging with full context

## Customization

### Adding Custom Protocols
Create a new `.md` file in `protocols/`:
```markdown
# My Custom Protocol
## When to Use: [trigger conditions]
## Steps: [step-by-step process]
## Verification: [how to confirm success]
```

Then add it to `master-memory.md` under Optional Components.

### Adding Feature Extensions
Create a new folder in `feature/`:
```
feature/my-feature/
├── README.md          # Feature documentation
└── my-feature-core.md # Implementation
```

---

**Version**: 1.0
**Inspired by**: [AI MemoryCore](https://github.com/Kiyoraka/Project-AI-MemoryCore) by Kiyoraka Ken
**Purpose**: Persistent AI memory for microservices architecture
**License**: Open Source

*Transform your AI assistant from a stateless tool into a microservices architecture expert that remembers everything about your system.*
