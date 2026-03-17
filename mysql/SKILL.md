---
name: mysql
description: Design, query, tune, secure, and operate MySQL (8.0+). Use for schema design (types, normalization, keys/FKs), index strategy, query optimization (EXPLAIN/slow log), transactions/locking, safe migrations and online DDL, connection pooling/timeouts, configuration tuning, replication/HA, backups and point-in-time recovery, observability, and production security hardening.
---

# MySQL

## Working Style

- Optimize for correctness and maintainability first, then tune with evidence (EXPLAIN, slow log, metrics).
- Assume InnoDB unless there is a concrete reason to use another engine.
- Design schemas and indexes from real query patterns, not hypothetical models.
- Treat migrations, rollback, and restore tests as part of the system.

## Quick Intake (Ask Before Recommending Changes)

- MySQL version (assume 8.0+ unless specified) and deployment (single node, replicas, managed service).
- Workload: OLTP vs analytics, read/write ratio, peak QPS, concurrency, latency SLOs.
- Dataset: biggest tables, growth rate, "hot" rows, cardinalities, typical filters/sorts.
- Current pain: slow endpoint/query, lock contention, replication lag, outages, disk/memory pressure.
- Safety constraints: downtime allowed, migration tooling, RPO/RTO, backup/PITR requirements.
- Security constraints: network exposure, TLS, least-privilege accounts, PII, audit requirements.

## Schema Design (Fast Defaults)

- Primary keys:
  - Prefer surrogate integer/bigint PKs for OLTP tables unless there is a strong natural key.
  - Keep PKs stable and immutable (PK changes are expensive).
- Foreign keys:
  - Use FKs when they help enforce integrity and you can tolerate the operational constraints.
  - Be explicit about `ON DELETE` / `ON UPDATE` behavior; avoid cascading surprises.
- Data types:
  - Choose the smallest type that fits (storage and cache efficiency matter).
  - Be deliberate with strings/collations; ensure comparisons/sorts match expectations.
  - Avoid "NULL everywhere": use NULL only when it has real semantic meaning.
- Soft deletes:
  - If used, standardize the pattern (`deleted_at` timestamp) and ensure indexes account for it.
- Partitioning:
  - Use only when it solves a proven operational/query problem; it adds complexity.

## Indexing Rules (What Usually Works)

- Start from real query shapes (filters, joins, order-by).
- Composite indexes:
  - Put the most selective equality predicates first, then range predicates, then ordering columns.
  - Remember the leftmost-prefix rule (index usefulness drops when leading columns are skipped).
- Covering indexes can be high-impact for hot reads, but increase write cost.
- Avoid over-indexing:
  - Every index has write amplification and memory/cache cost.
  - Remove unused indexes when confirmed by production usage metrics.

## Query And Index Tuning Loop

1. Reproduce with representative parameters.
2. Capture schema and indexes for involved tables:
   - `SHOW CREATE TABLE <table>\G`
   - `SHOW INDEX FROM <table>;`
3. Inspect the plan:
   - `EXPLAIN <query>;`
   - Use `EXPLAIN ANALYZE` when available/appropriate for measured execution details.
4. Fix the biggest bottleneck with minimal blast radius:
   - Rewrite query shape (avoid `SELECT *`, avoid N+1, avoid `WHERE function(col)=...`).
   - Add/adjust indexes (composite order aligned with filters/joins/sorts).
   - Avoid over-indexing; measure write overhead and storage/caching impact.
5. Validate:
   - Compare before/after timings under realistic load.
   - Confirm correctness (edge cases, pagination semantics, collation behavior).
   - Roll out safely (staging first; monitor slow log, error rates, replication lag).

## Common Query Anti-Patterns (Call Out Early)

- `SELECT *` on hot paths (fetch only needed columns).
- Functions on indexed columns in `WHERE` (precompute or rewrite).
- Leading-wildcard `LIKE` (`%foo`) without a plan (full scans).
- OFFSET pagination at scale (prefer keyset/seek pagination).
- N+1 queries from application loops (batch or join).
- Unbounded `IN (...)` lists (batch or temp table strategy).

## Schema Changes (Low Downtime Defaults)

- Prefer additive, backwards-compatible migrations:
  - Deploy schema additions first (new columns/tables/indexes).
  - Backfill in small batches (ordered by PK; watch replication lag).
  - Deploy app changes second.
  - Remove old schema elements last.
- For large tables, avoid blocking `ALTER TABLE` unless you've verified it is online for the exact operation.
  - Use MySQL online DDL where it is truly non-blocking.
  - Consider `gh-ost` / `pt-online-schema-change` for controlled cutovers.

## Transactions, Locking, And Concurrency

- Keep transactions short; do not hold locks across network calls or user interaction.
- Update rows in a consistent order across code paths to reduce deadlocks.
- Expect deadlocks to happen; implement safe retries where appropriate.
- Use idempotency patterns for write APIs (unique keys, upserts, request IDs).

## Observability And Debugging (Minimum Set)

- Capture plans and schema:
  - `EXPLAIN FORMAT=JSON <query>;`
  - `SHOW CREATE TABLE <table>\\G`
- Look at engine health:
  - `SHOW ENGINE INNODB STATUS\\G` (deadlocks, history length, mutex waits)
- Find hot statements:
  - Use slow query log (or managed service equivalents) and aggregate by total time.
  - Use Performance Schema when available to identify top digests.

## Operational Safety Defaults

- Backups:
  - Automate backups and test restores regularly.
  - Keep binary logs for point-in-time recovery if you need PITR.
- Connection management:
  - Use a connection pool; set realistic pool sizes and timeouts.
  - Avoid masking problems by setting `max_connections` extremely high.
- Replication/HA:
  - Treat replica reads as potentially stale; verify read-after-write requirements.
  - Monitor lag and replication thread health.
  - Migrate from deprecated `binlog_format` tuning to row-based replication defaults.
  - Use `replica_skip_errors` instead of deprecated `slave_skip_errors`.
  - Remove `temptable_use_mmap` from MySQL 9.4+ configs; the variable is removed and can break startup validation.
  - For replicas using `caching_sha2_password` without a secure connection, set `SOURCE_PUBLIC_KEY_PATH` or `GET_SOURCE_PUBLIC_KEY=1` during `CHANGE REPLICATION SOURCE TO`.
- Security:
  - Separate users for app/migrations/admin; apply least privilege.
  - Restrict network exposure; prefer private interfaces and firewall rules.
  - Use TLS in transit where required.
  - Avoid `mysql_native_password`; deprecated in 8.0.34, disabled by default in 8.4, removed in 9.0.
  - Prevent SQL injection with parameter binding (no string concatenation).

## Reference

- Use `references/mysql-best-practices.md` for the full guide, checklists, and snippets.
- Find topics quickly:
  - `rg -n '^## Schema design' references/mysql-best-practices.md`
  - `rg -n '^## Indexes' references/mysql-best-practices.md`
  - `rg -n '^## Query design and optimization' references/mysql-best-practices.md`
  - `rg -n '^## Migrations and zero/low-downtime changes' references/mysql-best-practices.md`
  - `rg -n '^## Security' references/mysql-best-practices.md`
  - `rg -n '^## Observability' references/mysql-best-practices.md`

## Extended Guidance
Use this when query performance or migrations are risky.

## Explain Plan Tips
- Watch for `type: ALL` (full scan) on large tables.
- Compare `rows` estimates before/after index changes.
- Ensure `Using filesort` is acceptable for the workload.

## Quick Debug Commands
```sql
SHOW PROCESSLIST;
SHOW ENGINE INNODB STATUS\\G
```

## Operational Reminder
- Keep slow query logs enabled for production triage.
