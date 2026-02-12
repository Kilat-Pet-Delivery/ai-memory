# Save Protocol - Microservices MemoryCore
*Triggered when user types "save" — persists all architecture changes to .md files*

## Core Philosophy

**When user types "save", the AI immediately saves all architecture changes, new discoveries, and session context to the .md files.** This ensures that service additions, endpoint changes, event flow updates, and debugging insights are permanently preserved.

## "Save" Command Trigger

When user types **"save"**, AI immediately performs:

### What Gets Saved
1. **New Services**: Any services discussed or created during session
2. **API Changes**: New endpoints, modified routes, deprecated APIs
3. **Event Changes**: New events, updated schemas, changed consumers
4. **Database Changes**: New tables, schema migrations, index changes
5. **Infrastructure Changes**: New containers, config changes, scaling decisions
6. **Architecture Decisions**: Pattern changes, convention updates
7. **Debug Insights**: Discovered failure patterns, workarounds, fixes
8. **Session Context**: Current work state for next session continuity

### Save Process
```
1. DETECT: User types "save" command
2. ANALYZE: Review current session for all architecture changes
3. CATEGORIZE: Sort changes by file destination
4. UPDATE: Modify relevant .md files with new information
5. VERIFY: Confirm all updates applied correctly
6. CONFIRM: Tell user what was saved and where
```

## File-Specific Save Rules

### service-registry.md Updates
**Triggers**:
- New microservice created or discussed
- Service port changed
- Service database changed
- Service status changed (active/deprecated/planned)
- New service dependency discovered

**Save Process**:
```
1. DETECT: New service or service change
2. VALIDATE: Confirm service details (name, port, DB, responsibility)
3. UPDATE: Add/modify entry in service registry table
4. CROSS-REF: Update master-memory.md quick reference table
5. VERIFY: No port conflicts, no duplicate names
```

### api-catalog.md Updates
**Triggers**:
- New API endpoint created
- Endpoint path or method changed
- Request/response schema changed
- Authentication requirement changed
- Endpoint deprecated or removed

**Save Process**:
```
1. DETECT: API endpoint change
2. DOCUMENT: Method, path, auth, request/response schema
3. CATEGORIZE: File under correct service section
4. CROSS-REF: Check for path conflicts across services
5. UPDATE: Modify api-catalog.md
```

### event-catalog.md Updates
**Triggers**:
- New async event created
- Event schema changed
- New producer or consumer added
- Event topic renamed
- Event flow modified

**Save Process**:
```
1. DETECT: Event system change
2. DOCUMENT: Topic, event name, producer, consumer(s), payload
3. MAP: Update event flow diagram
4. CROSS-REF: Verify producer/consumer services exist in registry
5. UPDATE: Modify event-catalog.md
```

### database-catalog.md Updates
**Triggers**:
- New database created
- New table added
- Schema migration applied
- Index added or removed
- Database extension enabled

**Save Process**:
```
1. DETECT: Database schema change
2. DOCUMENT: Database, table, columns, relationships
3. CROSS-REF: Link to owning service in registry
4. MIGRATE: Note migration version if applicable
5. UPDATE: Modify database-catalog.md
```

### architecture-core.md Updates
**Triggers**:
- New architecture pattern adopted
- Coding convention changed
- Shared library added or modified
- Communication pattern changed
- New architectural decision made

**Save Process**:
```
1. DETECT: Architecture decision or pattern change
2. DOCUMENT: Pattern name, rationale, implementation details
3. IMPACT: Note which services are affected
4. UPDATE: Modify architecture-core.md
```

### infrastructure.md Updates
**Triggers**:
- New container/service added to orchestration
- Database configuration changed
- Message broker topic added
- CI/CD pipeline modified
- Monitoring/alerting changed

**Save Process**:
```
1. DETECT: Infrastructure change
2. DOCUMENT: Component, configuration, purpose
3. CROSS-REF: Link to affected services
4. UPDATE: Modify infrastructure.md
```

### current-session.md Updates
**Triggers**:
- Every significant development action
- Service being actively worked on changes
- Debugging context discovered
- Important decision made

**Save Process**:
```
1. CONTINUOUS: Update throughout conversation
2. CONTEXT: Track active service and current task
3. PROGRESS: Note completed and pending work
4. PREPARE: Set up continuity for next session
```

## Continuous Learning Loop

### Real-Time Detection Cycle
```
CONVERSATION EXCHANGE
        |
ARCHITECTURE CHANGE DETECTED?
        |
    YES -> QUEUE FOR SAVE
        |
    NO  -> CONTINUE
        |
USER TYPES "SAVE"
        |
PROCESS ALL QUEUED CHANGES
        |
UPDATE .MD FILES
        |
CONFIRM TO USER
```

### Background Monitoring
While conversing, AI continuously monitors for:
- **Service mentions**: New services or changes to existing ones
- **Endpoint discussions**: API creation, modification, or deprecation
- **Event flow changes**: New events, updated schemas
- **Database work**: Migrations, new tables, schema changes
- **Infrastructure updates**: Docker, K8s, broker config changes
- **Architecture decisions**: Pattern adoptions, convention changes

## Save Confirmation Format

After saving, AI reports:
```
Saved! Here's what was updated:

  service-registry.md:
    + Added payment-service (port 8003, payments_db)

  api-catalog.md:
    + POST /api/payments/charge (payment-service)
    + GET /api/payments/:id (payment-service)

  event-catalog.md:
    + payment.completed event (payment -> notification)

  current-session.md:
    ~ Updated active context and next steps

4 files updated, 5 changes saved.
```

## Quality Standards

### Every Save Must Be
- **Accurate**: Based on confirmed information from conversation
- **Complete**: All related files updated (not just one)
- **Consistent**: No conflicts between files (port clashes, missing refs)
- **Cross-Referenced**: Services in events match registry, etc.
- **Verified**: AI confirms save was successful

## Error Prevention

### Save Safeguards
- **Port Conflict Check**: No two services share a port
- **Name Uniqueness**: No duplicate service names
- **Reference Integrity**: Events reference real services
- **Schema Consistency**: Database entries match service registry
- **Backup Awareness**: Warn if overwriting significant data

---

**Protocol Status**: Ready
**Activation**: User types "save"
**Integration**: Synchronizes all memory files

*Type "save" anytime to permanently preserve your microservices architecture knowledge!*
