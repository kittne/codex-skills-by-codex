## MySQL Best Practices

# MySQL Best Practices

This guide is a practical, “use-anytime” reference for working with MySQL in both **development** and **production** environments. It is written to be broadly applicable (general → advanced) and is suitable as source material for a coding/deployment skill.

> Scope notes:
>
> * Recommendations assume **modern MySQL (8.0+)** features and defaults where possible.
> * Where behavior differs by version or environment, the guidance calls it out explicitly.
> * Treat this as a baseline; always validate changes with load tests and backups.

---

## Table of Contents

* [Core principles](#core-principles)
* [Development vs production](#development-vs-production)
* [Schema design](#schema-design)
* [Data types](#data-types)
* [Indexes](#indexes)
* [Query design and optimization](#query-design-and-optimization)
* [Transactions, locking, and concurrency](#transactions-locking-and-concurrency)
* [Stored routines, triggers, views](#stored-routines-triggers-views)
* [Configuration and tuning](#configuration-and-tuning)
* [Connection management](#connection-management)
* [Backups and recovery](#backups-and-recovery)
* [Replication, HA, and scaling](#replication-ha-and-scaling)
* [Migrations and zero/low-downtime changes](#migrations-and-zerolow-downtime-changes)
* [Security](#security)
* [Observability: logging, metrics, profiling](#observability-logging-metrics-profiling)
* [Operational hygiene](#operational-hygiene)
* [Checklists](#checklists)

---

## Core principles

1. **Optimize for correctness and maintainability first**, then tune performance with evidence (profiling, EXPLAIN, slow logs).
2. **Prefer InnoDB** for almost all use cases. Treat other engines as specialized.
3. **Design schemas and indexes from your queries**, not from hypothetical models.
4. **Plan for change**: migrations, versioning, rollback, and test restores are part of the system.
5. **Security is part of schema and deployment** (least privilege, encryption in transit, safe secrets handling).
6. **Every performance fix must be measurable** (before/after, representative workload).

---

## Development vs production

### Development best practices

* Run MySQL close to production **features** (charset, SQL mode, auth plugin, major version), even if you relax durability for speed.
* Use containers for reproducibility, but **persist volumes** if you need stable datasets.
* Use smaller buffer sizes and lower connection limits to surface scaling issues early.
* Enable useful diagnostics:

  * `slow_query_log=ON` with a reasonable threshold
  * `performance_schema=ON`
* Seed dev data with realistic cardinalities (e.g., thousands/millions where it matters), otherwise indexing decisions will mislead you.

### Production best practices

* Treat configuration as code: version-control `my.cnf`, changes via PR, documented rationale.
* Use high-availability patterns (replicas, automated failover) appropriate to your RTO/RPO.
* Enforce strict SQL modes, consistent collation, and schema migration discipline.
* Backups are not optional: **automate them** and **test restores**.
* Separate write and read traffic when needed (primary + replicas), but validate read-after-write requirements.

---

## Schema design

### Normalization vs denormalization

* **Normalize** to reduce anomalies and keep writes consistent.
* **Denormalize intentionally** for performance or simplicity at read time:

  * document why
  * ensure write-path correctness
  * validate with tests and monitoring

Practical rule:

* Normalize first; denormalize only when query patterns and bottlenecks justify it.

### Naming conventions

* Choose consistent, boring names:

  * tables: `snake_case`, plural or singular (pick one)
  * primary keys: `id`
  * foreign keys: `<referenced_table>_id`
  * timestamps: `created_at`, `updated_at`, optionally `deleted_at`
* Avoid reserved keywords and ambiguous abbreviations.

### Primary keys

* Prefer **surrogate integer keys** (`BIGINT` auto-increment) or **UUIDs** (if globally unique IDs are required).
* If using UUIDs, consider:

  * storage as `BINARY(16)` (more compact than text)
  * using “time-ordered” UUID variants where possible to reduce index fragmentation
* Keep primary keys stable. Never update PKs unless absolutely required.

### Foreign keys

* Use FK constraints when they improve correctness, but balance:

  * operational complexity (bulk loads, migrations)
  * write amplification (extra checks)
* If you use FKs:

  * ensure indexed FK columns
  * define `ON DELETE` / `ON UPDATE` actions deliberately (avoid accidental cascades)

### Soft deletes

* Soft delete is useful for audit/restore but increases query complexity.
* If using soft deletes:

  * standardize on `deleted_at` (NULL means active)
  * ensure common queries filter `deleted_at IS NULL`
  * consider partial isolation via separate “archived” tables for very large datasets

### Partitioning

Partitioning can help manage very large tables, but adds complexity.

Use partitioning when:

* you have a natural partition key (e.g., time) and
* you regularly drop/archival partitions, or
* your queries reliably include the partition key

Avoid partitioning when:

* you’re trying to “fix” missing indexes or poorly-written queries.

---

## Data types

### Choose the smallest type that fits

* Use `INT` vs `BIGINT` deliberately.
* Use `TINYINT(1)` for booleans (or `BOOLEAN` alias).
* Use `DECIMAL(p,s)` for money and exact values; avoid floating point for currency.
* Use `DATETIME` vs `TIMESTAMP` intentionally:

  * `TIMESTAMP` is UTC-oriented and has range limits
  * `DATETIME` is often simpler for application-controlled time
* Use `JSON` type when it’s truly semi-structured—but don’t use it as a dumping ground. Add generated columns for commonly queried fields.

### Strings and collations

* Use `utf8mb4` (not legacy 3-byte UTF-8).
* Choose collation intentionally:

  * case-insensitive vs case-sensitive
  * sorting rules consistent with your product
* Keep collation consistent across schema unless you have a clear reason.

### Avoid “NULL everywhere”

* Use `NOT NULL` when possible; it simplifies logic and indexing.
* If `NULL` has semantic meaning (unknown vs empty), use it intentionally.

---

## Indexes

Indexes are the #1 lever for MySQL performance.

### General rules

* Index **columns used in**:

  * `WHERE` filters
  * join predicates
  * `ORDER BY`
  * `GROUP BY`
* Avoid indexing columns that are rarely filtered or extremely low-cardinality (e.g., `is_active`) unless combined with other columns.

### Composite indexes

* Composite index order matters:

  * place the most selective columns first **unless** your query shape demands otherwise
  * align index prefixes with common query patterns
* Remember leftmost prefix behavior: an index on `(a,b,c)` helps queries on `(a)` and `(a,b)` but not on `(b)` alone.

### Covering indexes

* A “covering index” includes all columns needed for the query, allowing MySQL to avoid extra table lookups.
* Use covering indexes for high-QPS endpoints, but keep them lean (don’t over-index).

### Unique indexes

* Use unique constraints to enforce correctness at the DB layer (e.g., unique email per tenant).
* Ensure you understand collation effects (case-insensitive collations may treat values as equal).

### Index maintenance

* Indexes have costs:

  * slower writes
  * more storage
  * more cache pressure
* Periodically review:

  * unused indexes (via performance schema/sys schema)
  * redundant indexes (prefixes duplicated)
* Drop indexes only after measuring (and ideally in a staging environment with realistic load).

---

## Query design and optimization

### Start with query shape

* Fetch only what you need (avoid `SELECT *` for APIs).
* Prefer explicit column lists for stability.
* Avoid N+1 query patterns:

  * batch fetch with `IN (...)`
  * join carefully
  * use caching or prefetch strategies

### Use EXPLAIN and EXPLAIN ANALYZE

**EXPLAIN** shows the plan; **EXPLAIN ANALYZE** (where available) measures execution details.

Workflow:

1. Run the query with representative parameters.
2. `EXPLAIN` to verify index usage and row estimates.
3. If slow, use `EXPLAIN ANALYZE` / profiling tools and compare alternatives.

Watch for:

* full table scans (`type=ALL`)
* “Using temporary; Using filesort” (not always bad, but investigate)
* large “rows” estimates

### Avoid anti-patterns

* `WHERE function(col) = ...` prevents index usage (e.g., `DATE(created_at)=...`).

  * Prefer range predicates: `created_at >= ... AND created_at < ...`
* Leading wildcards: `LIKE '%term'` usually can’t use an index.
* Huge `IN (...)` lists can be costly; consider temp tables or join to a derived dataset.

### Pagination

* Avoid large `OFFSET` pagination for big datasets.
* Prefer **keyset pagination** (“seek method”):

  * `WHERE (created_at, id) < (?, ?) ORDER BY created_at DESC, id DESC LIMIT 50`

### Aggregations

* Use appropriate indexes for `GROUP BY` and `ORDER BY`.
* Consider pre-aggregation tables or materialized summaries for heavy reporting.

### Parameterization and prepared statements

* Use prepared statements to:

  * reduce parse overhead
  * avoid injection risks
* Ensure app-layer uses parameter binding, not string concatenation.

---

## Transactions, locking, and concurrency

### Use transactions deliberately

* Group logically related writes in a single transaction.
* Keep transactions **short** to reduce lock contention.
* Choose the right isolation level (often `REPEATABLE READ` default is OK, but validate).
* Avoid “chatty transactions” that wait on user input or network calls while holding locks.

### Understand locks and deadlocks

* InnoDB uses row-level locks, but patterns can escalate contention:

  * updating rows in different orders across code paths increases deadlock risk
  * large range updates can lock many rows (gap locks depending on isolation)

Best practices:

* Always update rows in a consistent order.
* Use indexes to make updates target fewer rows.
* Handle deadlocks in application code with retries (deadlocks can occur even in well-designed systems).

### Idempotency

* For write APIs, design idempotent operations:

  * unique keys + upserts
  * request IDs stored and checked
* This prevents duplicate writes on retry.

---

## Stored routines, triggers, views

### Stored procedures / functions

Use when:

* you need to co-locate a small amount of logic for performance (reduce round trips)
* you can version and test them like code

Avoid when:

* logic becomes complex and tightly coupled to the DB
* you need portability or easy local testing across DB versions

### Triggers

Use sparingly:

* Triggers can hide behavior and surprise developers.
* If you use them:

  * keep logic small and deterministic
  * document clearly
  * test thoroughly (especially for bulk writes)

### Views

* Views can simplify common joins or filters.
* They do not inherently improve performance; they are a query abstraction.
* Avoid stacking views on views unless you control complexity.

---

## Configuration and tuning

### General approach

1. Start with known-good defaults for your workload class.
2. Change one thing at a time.
3. Validate with load tests and production metrics.

### Baseline config knobs (InnoDB-centric)

> Names and recommended values vary by workload. Use these as concepts to validate, not blind copy-paste.

* `innodb_buffer_pool_size`: typically the biggest lever. In dedicated DB hosts, often 60–80% of RAM.
* `innodb_log_file_size` / redo log sizing: impacts write throughput and recovery time.
* `innodb_flush_log_at_trx_commit`:

  * `1` safest durability (common in prod)
  * `2` trades durability for performance (may be acceptable in some cases; understand risk)
* `sync_binlog`:

  * `1` safest for binlog durability
  * higher values improve throughput at some durability risk
* `max_connections`: keep realistic; too high can cause memory storms.
* `sql_mode`: enable strict modes in prod; avoid permissive defaults that hide bugs.
* `character_set_server` / `collation_server`: set explicitly to avoid surprises.

### OS / host tuning (production)

* Ensure sufficient file descriptors.
* Use fast storage (SSD/NVMe) for redo logs and data.
* Validate filesystem options and I/O scheduler for database workloads.
* Use stable NTP time sync.

---

## Connection management

### Use a connection pool

* Do not create a new DB connection per request.
* Use pool settings appropriate to concurrency:

  * pool size per app instance
  * max lifetime / idle timeout
* Consider using a proxy (e.g., ProxySQL) if you need:

  * connection multiplexing
  * query routing to replicas
  * failover handling

### Avoid overloaded `max_connections`

* Bigger is not always better.
* Often a smaller max connections + queueing at the app layer yields better stability.

### Timeouts

Set and enforce:

* connection timeout
* query timeout
* transaction timeout (application-level)
* server-side idle timeouts (`wait_timeout`) aligned with your pool behavior

---

## Backups and recovery

Backups are a process, not a command.

### Backup types

* **Logical backups** (`mysqldump`, `mysqlpump`):

  * portable
  * slower for large datasets
  * easier to inspect
* **Physical backups** (filesystem-level, XtraBackup, vendor tooling):

  * faster for large datasets
  * better for point-in-time recovery when paired with binlogs
  * require careful procedures

### Best practices

* Always enable and retain **binary logs** to support point-in-time recovery (PITR).
* Automate backups on a schedule appropriate to RPO.
* Store backups off-host and off-region.
* Encrypt backups at rest.
* **Test restores regularly**:

  * restore into a staging environment
  * validate checksums and application-level sanity
* Practice a disaster recovery runbook:

  * restore full backup
  * apply binlogs to target timestamp
  * verify service readiness

### Backup verification quick checklist

* [ ] latest backup exists
* [ ] restore succeeds
* [ ] schemas and row counts match expected
* [ ] application smoke tests pass
* [ ] PITR procedure tested

---

## Replication, HA, and scaling

### Replication basics

* Use replicas for:

  * read scaling
  * HA failover
  * backup offloading
* Make sure your application understands replication lag:

  * reads from replicas might not see recent writes

### GTID and failover readiness

* Prefer GTID-based replication if your operational model supports it.
* Monitor replication health:

  * lag
  * relay log errors
  * IO/SQL thread status

### HA patterns

Common patterns include:

* **Primary + replicas + failover manager**
* **InnoDB Cluster / Group Replication** (more complex, but integrated options)
* **Proxy layer** to route writes/reads and handle failover

Choose based on:

* operational maturity
* failure modes you can tolerate
* consistency requirements

### Scaling approach

* Vertical scaling first (CPU/RAM/IO), then:

  * query/index optimization
  * caching (Redis or app caches)
  * read replicas
  * sharding (only when necessary; complexity is high)

---

## Migrations and zero/low-downtime changes

Schema migrations are where many outages originate.

### Principles

* Make migrations **backwards compatible**:

  * deploy schema changes first
  * deploy app changes second
  * remove old schema elements last
* Prefer additive changes:

  * add new columns/tables
  * backfill in batches
  * switch reads/writes to new columns
  * then drop old columns later

### Online schema change tools

For large tables, avoid blocking `ALTER TABLE` if it will lock writes.

Options:

* Use MySQL online DDL when it truly is non-blocking for your operation.
* Use tools like:

  * `gh-ost`
  * `pt-online-schema-change`
    These perform table-copy or trigger-based changes with controlled cutover.

### Backfills

* Backfill in small batches (`LIMIT`, ordered by PK).
* Sleep between batches to reduce write pressure.
* Monitor replication lag during backfill.

---

## Security

### Account and privilege management

* Use separate users for:

  * applications
  * migrations
  * read-only reporting
  * admin operations
* Apply least privilege:

  * avoid `GRANT ALL` for applications
  * avoid SUPER-like privileges for non-admin tasks
* Rotate credentials regularly; use a secret manager.

### Authentication

* Prefer modern auth plugins and strong passwords.
* Avoid remote root logins.
* Disable anonymous users and remove test databases in prod.

### Network security

* Bind MySQL to private interfaces only.
* Restrict inbound access using firewalls/security groups.
* Require TLS for client connections in production.

### Encryption

* Encrypt in transit: TLS between app and DB.
* Encrypt at rest:

  * disk encryption (host-level) and/or
  * MySQL tablespace encryption (if configured)
* Encrypt backups.

### SQL injection prevention

* Always use parameter binding.
* Prohibit string-concatenated queries in code review.

### Audit and compliance

* Enable audit logging if you have compliance requirements (via plugins or external tooling).
* Keep logs secured and centralized.

---

## Observability: logging, metrics, profiling

### Logs

Enable and manage:

* error log
* slow query log
* general log only for short-term debugging (it’s very verbose)

Slow query log best practices:

* set a threshold appropriate to your SLOs
* sample or rate-limit if needed
* analyze with tools (pt-query-digest, etc.)

### Metrics

Monitor:

* QPS, TPS
* latency percentiles (app-side and DB-side)
* buffer pool hit ratio
* row lock waits / deadlocks
* replication lag (if applicable)
* disk I/O, CPU, memory
* connection usage

### Diagnostic commands

Common, safe commands:

* `SHOW PROCESSLIST;`
* `SHOW ENGINE INNODB STATUS\G`
* `EXPLAIN <query>;`
* `SHOW STATUS LIKE 'Threads_connected';`

Use a performance workflow:

1. detect slow endpoint
2. identify query
3. EXPLAIN
4. fix index/query
5. verify improvement

---

## Operational hygiene

* Patch and upgrade intentionally; read release notes.
* Use maintenance windows for major changes; have rollback plans.
* Document runbooks:

  * failover
  * restore
  * schema migration
  * handling deadlocks / spikes
* Use infrastructure automation for provisioning and config drift prevention.

---

## Checklists

### Schema and query checklist

* [ ] tables have primary keys
* [ ] foreign keys indexed (if used)
* [ ] correct data types (no money in floats)
* [ ] utf8mb4 and chosen collation set explicitly
* [ ] indexes match real query patterns
* [ ] queries avoid `SELECT *` in APIs
* [ ] EXPLAIN verified for critical queries
* [ ] pagination uses keyset for large datasets

### Production readiness checklist

* [ ] TLS enabled for connections
* [ ] least-privilege DB users per app/service
* [ ] backups automated + offsite + encrypted
* [ ] restore tested recently
* [ ] slow query log enabled and monitored
* [ ] replication/HA plan documented (if needed)
* [ ] migration approach supports low downtime
* [ ] monitoring/alerting for lag, locks, CPU, disk, memory

---

## Quick reference snippets

### Keyset pagination example

```sql
SELECT id, created_at, title
FROM posts
WHERE (created_at, id) < (?, ?)
ORDER BY created_at DESC, id DESC
LIMIT 50;
```

### “Avoid function on indexed column” rewrite

Bad:

```sql
SELECT * FROM events WHERE DATE(created_at) = '2026-02-07';
```

Better:

```sql
SELECT *
FROM events
WHERE created_at >= '2026-02-07 00:00:00'
  AND created_at <  '2026-02-08 00:00:00';
```

---

---