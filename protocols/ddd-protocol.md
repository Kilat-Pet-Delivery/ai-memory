# DDD Protocol - Domain-Driven Design for Microservices
*Bounded contexts, aggregates, domain events, and strategic/tactical patterns*

## Why DDD in Microservices

**Domain-Driven Design is the foundation for correctly splitting a system into microservices.** Each microservice should align with a Bounded Context — a clear boundary where a domain model applies. Without DDD, you end up with distributed monolith instead of true microservices.

## Strategic Design (The Big Picture)

### Bounded Contexts

*Each microservice = one bounded context. Define yours here:*

| # | Bounded Context | Service | Core Domain? | Description |
|---|----------------|---------|-------------|-------------|
| 1 | [CONTEXT_1] | [SERVICE_1] | [Core/Supporting/Generic] | [What domain this covers] |
| 2 | [CONTEXT_2] | [SERVICE_2] | [Core/Supporting/Generic] | [What domain this covers] |
| 3 | [CONTEXT_3] | [SERVICE_3] | [Core/Supporting/Generic] | [What domain this covers] |
| 4 | [CONTEXT_4] | [SERVICE_4] | [Core/Supporting/Generic] | [What domain this covers] |
| 5 | [CONTEXT_5] | [SERVICE_5] | [Core/Supporting/Generic] | [What domain this covers] |

### Domain Classification

| Type | Definition | Investment Level | Examples |
|------|-----------|-----------------|----------|
| **Core Domain** | What makes your business unique. Competitive advantage. | High — best developers, custom code | [YOUR_CORE_DOMAINS] |
| **Supporting Domain** | Supports the core but not unique to you. | Medium — custom but simpler | [YOUR_SUPPORTING_DOMAINS] |
| **Generic Domain** | Common to any business. Use off-the-shelf. | Low — buy/use existing solutions | [YOUR_GENERIC_DOMAINS] |

### Context Map

*How bounded contexts communicate with each other:*

```
[CONTEXT_1] <--Conformist-- [CONTEXT_2]
     |
     +--Partnership-- [CONTEXT_3]
     |
     +--Anti-Corruption Layer-- [CONTEXT_4]
     |
[CONTEXT_5] --Published Language--> [CONTEXT_1]
```

### Context Relationships

| Upstream Context | Downstream Context | Relationship | Description |
|-----------------|-------------------|-------------|-------------|
| [CONTEXT_A] | [CONTEXT_B] | [Type] | [How they interact] |
| [CONTEXT_C] | [CONTEXT_D] | [Type] | [How they interact] |

**Relationship Types**:
| Type | Description | When to Use |
|------|-----------|-------------|
| **Partnership** | Both teams cooperate equally | Two core domains that evolve together |
| **Customer-Supplier** | Upstream serves downstream | Downstream depends on upstream's API |
| **Conformist** | Downstream adapts to upstream | No power to change upstream (e.g., external API) |
| **Anti-Corruption Layer (ACL)** | Translation layer between contexts | Protect your model from external models |
| **Shared Kernel** | Small shared model between contexts | Use sparingly — creates coupling |
| **Published Language** | Standardized protocol/schema | Event schemas, public API contracts |
| **Open Host Service** | Well-defined public API | Service exposes stable API for many consumers |
| **Separate Ways** | No integration needed | Contexts are truly independent |

## Tactical Design (Inside Each Service)

### Domain Layer Structure

```
internal/domain/
├── model/              # Entities + Value Objects + Aggregates
│   ├── [entity].go
│   ├── [value_object].go
│   └── [aggregate_root].go
├── repository/         # Repository interfaces (NOT implementations)
│   └── [entity]_repository.go
├── service/            # Domain services (business logic that doesn't fit in entities)
│   └── [domain]_service.go
└── event/              # Domain events
    └── [domain]_events.go
```

### Building Blocks

#### Entities
- Have a **unique identity** (ID) that persists over time
- Contain **behavior** (methods), not just data
- Are **mutable** — their state changes

```
Example:
  Order entity has: ID, status, items, total
  Methods: AddItem(), RemoveItem(), CalculateTotal(), Submit()
  Identity: OrderID persists across status changes
```

**Project Entities**:
| Bounded Context | Entity | Identity | Key Behavior |
|----------------|--------|----------|-------------|
| [CONTEXT_1] | [Entity_1] | [ID type] | [Key methods] |
| [CONTEXT_1] | [Entity_2] | [ID type] | [Key methods] |
| [CONTEXT_2] | [Entity_3] | [ID type] | [Key methods] |

#### Value Objects
- **No identity** — defined by their attributes
- **Immutable** — create new instead of modifying
- **Self-validating** — always in valid state

```
Example:
  Money value object: {amount: 100, currency: "USD"}
  Address value object: {street, city, state, zip}
  Email value object: validates format on creation
```

**Project Value Objects**:
| Bounded Context | Value Object | Attributes | Validation |
|----------------|-------------|-----------|-----------|
| [CONTEXT_1] | [VO_1] | [Attributes] | [Rules] |
| [CONTEXT_2] | [VO_2] | [Attributes] | [Rules] |

#### Aggregates
- **Cluster of entities and value objects** treated as a single unit
- Has an **Aggregate Root** — the only entry point for modifications
- **Transactional boundary** — one aggregate = one transaction
- Other aggregates are referenced by **ID only**, not by object reference

```
Example:
  Order Aggregate:
    Root: Order (entity)
    Contains: OrderItem (entity), ShippingAddress (value object)
    Referenced by ID: customerId, productId (not full objects)

  Rule: To add an item, you go through Order.AddItem(),
        never modify OrderItem directly
```

**Project Aggregates**:
| Bounded Context | Aggregate Root | Contains | References (by ID) |
|----------------|---------------|----------|-------------------|
| [CONTEXT_1] | [AggRoot_1] | [Entities, VOs] | [External IDs] |
| [CONTEXT_2] | [AggRoot_2] | [Entities, VOs] | [External IDs] |

#### Domain Events
- **Something that happened** in the domain that other parts care about
- Named in **past tense**: OrderPlaced, PaymentCompleted, UserRegistered
- **Immutable** — once happened, cannot be changed
- Trigger **side effects** in other bounded contexts

```
Example:
  OrderPlaced event:
    - Published by: Order Service (when order is submitted)
    - Consumed by: Payment Service (create payment)
    - Consumed by: Notification Service (send confirmation)
    - Payload: {orderId, customerId, total, items, timestamp}
```

**Project Domain Events**:
| Event | Source Context | Target Context(s) | Trigger |
|-------|--------------|-------------------|---------|
| [Event_1] | [CONTEXT_A] | [CONTEXT_B, C] | [When this happens] |
| [Event_2] | [CONTEXT_B] | [CONTEXT_A] | [When this happens] |

#### Domain Services
- Business logic that **doesn't naturally belong to any entity**
- Often involves **multiple aggregates** or **external concerns**
- **Stateless** — no own state, operates on entities

```
Example:
  PricingService: calculates price using Product + Discount + Tax rules
  TransferService: moves money between two Account aggregates
```

**Project Domain Services**:
| Bounded Context | Domain Service | Purpose | Involved Aggregates |
|----------------|---------------|---------|-------------------|
| [CONTEXT_1] | [Service_1] | [What it does] | [Aggregates used] |

#### Repositories
- **Interface** defined in domain layer (not implementation)
- **Implementation** in infrastructure layer
- One repository per **Aggregate Root**
- Abstracts data persistence

```
Example:
  // Domain layer — interface
  type OrderRepository interface {
      FindByID(id string) (*Order, error)
      Save(order *Order) error
      FindByCustomer(customerID string) ([]*Order, error)
  }

  // Infrastructure layer — implementation
  type PostgresOrderRepository struct { db *gorm.DB }
  func (r *PostgresOrderRepository) FindByID(id string) (*Order, error) { ... }
```

### Aggregate Design Rules

| Rule | Description | Why |
|------|-----------|-----|
| **One aggregate per transaction** | Never modify two aggregates in one DB transaction | Scalability and consistency |
| **Reference by ID** | Aggregates reference others by ID, not object | Loose coupling between contexts |
| **Small aggregates** | Keep aggregates small and focused | Performance and concurrency |
| **Eventual consistency** | Use domain events between aggregates | Avoids distributed transactions |
| **Invariant protection** | Aggregate root enforces all business rules | Consistency within boundary |

## Ubiquitous Language

*The shared vocabulary between developers and domain experts. Each bounded context has its own language.*

### [CONTEXT_1] Language
| Term | Definition | NOT the same as |
|------|-----------|----------------|
| [Term_1] | [What it means in this context] | [Different meaning in other context] |
| [Term_2] | [What it means in this context] | [Different meaning in other context] |

### [CONTEXT_2] Language
| Term | Definition | NOT the same as |
|------|-----------|----------------|
| [Term_1] | [What it means in this context] | [Different meaning in other context] |

**Example**: "Order" might mean:
- In **Sales Context**: A customer's purchase request
- In **Fulfillment Context**: A shipping instruction
- In **Billing Context**: An invoice trigger

## Common DDD Mistakes in Microservices

| Mistake | Problem | Fix |
|---------|---------|-----|
| Sharing database between services | Breaks bounded context | Database per service |
| Anemic domain model | Entities are just data bags, all logic in services | Put behavior in entities |
| Too many sync calls | Distributed monolith | Use domain events for cross-context |
| CRUD-only services | No domain logic, just database wrappers | Model real business processes |
| Shared entity models | Tight coupling between contexts | Each context has own models |
| Too large aggregates | Performance and locking issues | Split into smaller aggregates |
| Ignoring ubiquitous language | Miscommunication, wrong abstractions | Define terms per context |

## DDD Checklist for New Service

```
[ ] Identified bounded context and its boundaries
[ ] Classified as Core / Supporting / Generic domain
[ ] Defined ubiquitous language for this context
[ ] Mapped context relationships (upstream/downstream)
[ ] Identified aggregate roots
[ ] Defined entities vs value objects
[ ] Designed domain events (published and consumed)
[ ] Created repository interfaces in domain layer
[ ] Put business logic in entities/aggregates, not in handlers
[ ] References to other contexts use IDs, not objects
[ ] Anti-corruption layer for external context translations
```

---

**Version**: DDD Protocol v1.0
**Last Updated**: [DATE]
**Status**: Template — requires project-specific domain modeling

*This file ensures your microservices are designed around real business domains, not just technical splits!*
