# Database Catalog - [PROJECT_NAME]
*Complete reference of all databases, schemas, and data models*

## Database Overview

**Primary Database**: [PostgreSQL / MySQL / MongoDB / etc.]
**Version**: [DB_VERSION]
**Host**: [DB_HOST]
**Port**: [DB_PORT]
**Total Databases**: [COUNT]
**ORM**: [GORM / Hibernate / Prisma / SQLAlchemy / etc.]
**Migration Tool**: [golang-migrate / Flyway / Alembic / Prisma]

## Database Inventory

| # | Database Name | Owner Service | Extensions | Size Est. |
|---|--------------|--------------|------------|-----------|
| 1 | [DB_NAME_1] | [SERVICE_1] | [Extensions] | [Size] |
| 2 | [DB_NAME_2] | [SERVICE_2] | [Extensions] | [Size] |
| 3 | [DB_NAME_3] | [SERVICE_3] | [Extensions] | [Size] |
| 4 | [DB_NAME_4] | [SERVICE_4] | [Extensions] | [Size] |

## Connection Strings

```
# Pattern
[DB_TYPE]://[USER]:[PASSWORD]@[HOST]:[PORT]/[DB_NAME]?sslmode=[MODE]

# Service-specific
[SERVICE_1]: [DB_TYPE]://[USER]:[PASS]@[HOST]:[PORT]/[DB_NAME_1]
[SERVICE_2]: [DB_TYPE]://[USER]:[PASS]@[HOST]:[PORT]/[DB_NAME_2]
[SERVICE_3]: [DB_TYPE]://[USER]:[PASS]@[HOST]:[PORT]/[DB_NAME_3]
```

---

## Database: [DB_NAME_1] (Service: [SERVICE_1])

### Tables

#### Table: [table_name_1]
**Purpose**: [What data this table stores]
**Row Estimate**: [Approximate row count]

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID / BIGINT | NO | gen_random_uuid() | Primary key |
| [column_2] | VARCHAR(255) | NO | — | [Description] |
| [column_3] | INTEGER | YES | 0 | [Description] |
| [column_4] | BOOLEAN | NO | false | [Description] |
| [column_5] | TIMESTAMP | NO | NOW() | [Description] |
| created_at | TIMESTAMP | NO | NOW() | Record creation time |
| updated_at | TIMESTAMP | NO | NOW() | Last update time |

**Indexes**:
| Index Name | Columns | Type | Purpose |
|-----------|---------|------|---------|
| pk_[table] | id | PRIMARY | Primary key |
| idx_[table]_[col] | [column] | BTREE | [Purpose] |
| idx_[table]_[col] | [column] | UNIQUE | [Purpose] |

**Foreign Keys**:
| Column | References | On Delete |
|--------|-----------|-----------|
| [column] | [other_table](id) | CASCADE / SET NULL |

---

#### Table: [table_name_2]
**Purpose**: [What data this table stores]

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | gen_random_uuid() | Primary key |
| [fk_column] | UUID | NO | — | FK to [table_1] |
| [column_2] | TEXT | YES | — | [Description] |
| created_at | TIMESTAMP | NO | NOW() | Record creation |

---

*Copy the table template for each additional table.*

## Database: [DB_NAME_2] (Service: [SERVICE_2])

### Tables

#### Table: [table_name]
**Purpose**: [Description]

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | gen_random_uuid() | Primary key |
| [columns...] | | | | |

---

*Add additional database sections as needed.*

## Entity Relationship Summary

```
[TABLE_A] 1---* [TABLE_B]    (One-to-Many)
[TABLE_B] *---1 [TABLE_C]    (Many-to-One)
[TABLE_D] *---* [TABLE_E]    (Many-to-Many via junction)
```

### Cross-Service Data References
*Note: In microservices, services reference each other by ID, not by foreign key.*

| Source Service | Source Table | Referenced ID | Target Service | Target Table |
|---------------|-------------|---------------|---------------|-------------|
| [SERVICE_A] | [table_a] | user_id | [SERVICE_B] | users |
| [SERVICE_A] | [table_a] | order_id | [SERVICE_C] | orders |

## Database Extensions

| Extension | Database(s) | Purpose |
|-----------|------------|---------|
| uuid-ossp | [All] | UUID generation |
| [postgis] | [Geo DB] | Geospatial queries |
| [pg_trgm] | [Search DB] | Text search |

## Migration History

### [SERVICE_1] Migrations
| Version | Name | Date | Description |
|---------|------|------|-------------|
| V001 | create_[table]_table | [DATE] | Initial table creation |
| V002 | add_[column]_to_[table] | [DATE] | Added [column] |
| V003 | create_[index]_index | [DATE] | Performance index |

### Migration Commands
```bash
# Run migrations for a service
[migration_tool] -path [service]/migrations -database "[connection_string]" up

# Rollback last migration
[migration_tool] -path [service]/migrations -database "[connection_string]" down 1

# Check migration status
[migration_tool] -path [service]/migrations -database "[connection_string]" version
```

## Seed Data

### Required Seed Data
| Database | Table | Purpose | Count |
|----------|-------|---------|-------|
| [DB_1] | [table] | [Default roles, configs, etc.] | [N] |
| [DB_2] | [table] | [Test data] | [N] |

### Seed Script Location
```
[path/to/seed/scripts]
```

## Backup & Recovery

### Backup Strategy
- **Frequency**: [Daily / Hourly / Continuous]
- **Type**: [Full / Incremental / WAL archiving]
- **Retention**: [Duration]
- **Storage**: [S3 / GCS / Local]

### Recovery Commands
```bash
# Backup specific database
pg_dump -h [HOST] -p [PORT] -U [USER] -d [DB_NAME] > backup.sql

# Restore from backup
psql -h [HOST] -p [PORT] -U [USER] -d [DB_NAME] < backup.sql
```

## Adding a New Database

When a new database is needed:
1. Add entry to **Database Inventory** table
2. Create a **Database section** with table definitions
3. Add connection string to **Connection Strings**
4. Update `main/service-registry.md` with database info
5. Document any required extensions
6. Create initial migration files
7. Add seed data if needed

---

**Version**: Database Catalog v1.0
**Last Updated**: [DATE]
**Status**: Template — requires project-specific schemas

*This file documents all data storage. Keep it updated when schemas change!*
