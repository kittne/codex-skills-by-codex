---
name: postgresql
description: >
  Design, query, tune, secure, and operate PostgreSQL clusters for production workloads. Use when
  working on schema design, indexing/query plans, transaction behavior, autovacuum/bloat,
  backup/restore, replication and HA/DR, observability, security hardening, or upgrades.
---

# PostgreSQL

## Workflow
1. Confirm PostgreSQL version, hosting model (self-managed vs managed), and business RPO/RTO.
2. Establish workload shape: OLTP vs analytics, write/read ratio, peak concurrency, data growth.
3. Baseline effective runtime config and current health before changing anything.
4. Review schema constraints, index strategy, and query plan quality for top latency queries.
5. Validate transaction behavior, lock patterns, vacuum throughput, and bloat pressure.
6. Verify backup integrity with restore drills and replication lag guardrails.
7. Apply least-privilege security, network controls, and extension policy.
8. Execute changes incrementally with measurable before/after evidence.

## Preflight (Ask / Check First)
- Current PostgreSQL major/minor version and planned upgrade path.
- Hardware or service tier constraints (CPU, RAM, disk IOPS, throughput).
- Critical SLOs (latency, availability) and compliance requirements.
- Known incidents: lock storms, replica lag, bloat, failover instability.
- Tooling available: `pg_stat_statements`, log pipeline, metrics/alerts, backup tooling.
- Change window and rollback expectations.

## Operating Principles
- Keep settings-as-code and verify effective values with `pg_settings`.
- Treat connection count and memory as coupled capacity variables.
- Prefer conservative global defaults; raise memory per role/session when needed.
- Keep durability posture explicit (`fsync` on, scoped `synchronous_commit` decisions).
- Tune autovacuum per table for high-churn relations.
- Prove recovery with routine restore tests, not only backup success messages.
- Prefer guardrails and feedback loops over static folklore values.

## Configuration and Capacity Planning
- Tune `max_connections` with pooling strategy; avoid unbounded direct connections.
- Balance `shared_buffers`, `work_mem`, and `maintenance_work_mem` against concurrency.
- Tune checkpoint behavior (`checkpoint_timeout`, `max_wal_size`) to reduce I/O spikes.
- Keep `checkpoint_completion_target` high to smooth checkpoint write pressure.
- Enable diagnostic logging for checkpoints and slow query analysis.
- Use role/database scoped settings for special workloads instead of global inflation.

### Baseline SQL Checks
```sql
SELECT name, setting, unit, source
FROM pg_settings
WHERE name IN (
  'max_connections','shared_buffers','work_mem','maintenance_work_mem',
  'max_wal_size','checkpoint_timeout','checkpoint_completion_target',
  'autovacuum','log_checkpoints'
)
ORDER BY name;
```

```sql
SELECT datname, numbackends, xact_commit, xact_rollback, blks_hit, blks_read
FROM pg_stat_database
ORDER BY datname;
```

## Schema and Data Modeling
- Use normalized relational modeling by default; denormalize with measured justification.
- Enforce business rules in constraints (PK/FK/UNIQUE/CHECK/NOT NULL).
- Prefer identity columns for surrogate keys in modern versions.
- Model many-to-many relations explicitly when metadata on relationships is needed.
- Define partitioning only when data lifecycle and access patterns justify complexity.
- Keep migration scripts reversible where feasible and test on production-like data.

## Indexing and Query Tuning
- Index for read patterns, joins, and selective filters; avoid speculative indexes.
- Use multi-column indexes with left-prefix behavior in mind.
- Use partial indexes for skewed predicates and sparse workloads.
- Prefer covering patterns (`INCLUDE`) when heap fetch reduction matters.
- Track index bloat and redundant/unused indexes.
- Use `EXPLAIN (ANALYZE, BUFFERS)` for plan truth, not assumptions.

### Plan Analysis Snippet
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT o.id, o.created_at
FROM orders o
WHERE o.customer_id = $1
ORDER BY o.created_at DESC
LIMIT 50;
```

### Query Tuning Loop
1. Capture top queries by total time and mean latency.
2. Reproduce with representative parameters and data volume.
3. Analyze plan row estimates vs actual rows to detect stats issues.
4. Apply smallest change first: index, query rewrite, or stats target.
5. Re-test and record before/after latency and buffer profile.

## Transactions and Concurrency
- Keep transactions short to reduce lock duration and vacuum interference.
- Choose the lowest isolation level that preserves correctness.
- Use retry-safe patterns for serialization/deadlock failures.
- Identify lock hotspots with lock-wait diagnostics.
- Avoid long-running idle-in-transaction sessions.

### Lock Diagnostic Query
```sql
SELECT pid, wait_event_type, wait_event, state, query
FROM pg_stat_activity
WHERE wait_event_type IS NOT NULL
ORDER BY state, pid;
```

## Vacuum, Bloat, and Maintenance
- Monitor dead tuples and vacuum cadence per large/high-churn table.
- Tune autovacuum scale factors and thresholds at table level.
- Monitor transaction ID age and wraparound risk.
- Use `ANALYZE` strategy to keep planner statistics fresh.
- Reserve `VACUUM FULL` for controlled maintenance windows only.

## Backup, Restore, Replication, and HA/DR
- Define backup policy from RPO/RTO first, then tool selection.
- Validate PITR regularly by restoring into isolated environments.
- Monitor replication lag and WAL retention behavior.
- Cap slot-driven WAL growth with explicit guardrails.
- Document failover, promotion, and rejoin runbooks.

### Recovery Readiness Checklist
- Base backup + WAL archival validated.
- Restore command tested and timed.
- Replica lag alerts configured.
- Failover drill performed and rollback plan documented.
- Post-failover application reconnection behavior validated.

## Monitoring and Observability
- Enable `pg_stat_statements` where policy allows.
- Track: query latency, TPS, cache hit ratio, locks, checkpoints, replication lag.
- Correlate database events with app and infrastructure telemetry.
- Keep structured logs and sane retention/rotation.
- Alert on saturation trends, not only hard failures.

## Security Hardening
- Apply role-based least privilege and default-deny grants.
- Restrict network access and enforce TLS where supported.
- Keep secrets out of SQL scripts and logs.
- Control extension installation policy and review risk per extension.
- Audit superuser usage and privileged DDL activity.

## Upgrades, Migrations, and Extensions
- Plan major upgrades with explicit compatibility matrix and dry-runs.
- Test extension compatibility before upgrade cutover.
- Use phased rollout with canary traffic when possible.
- Record rollback criteria and downgrade constraints.
- Re-run performance baselines after major upgrades.

## Managed Service Guidance
- Map provider knobs to PostgreSQL semantics before tuning.
- Validate defaults that differ from upstream PostgreSQL docs.
- Confirm backup retention, replica behavior, and maintenance windows.
- Keep provider failover semantics aligned with application retry logic.

## Validation Commands (Typical)
```bash
psql -X -d "$DATABASE_URL" -c "SELECT version();"
psql -X -d "$DATABASE_URL" -c "SHOW max_connections;"
psql -X -d "$DATABASE_URL" -c "SELECT now();"
```

```bash
psql -X -d "$DATABASE_URL" -f sql/baseline-health.sql
psql -X -d "$DATABASE_URL" -f sql/top-queries.sql
```

## Common Failure Modes
- Oversized `work_mem` combined with high concurrency causing memory exhaustion.
- Under-tuned checkpoints producing periodic latency cliffs.
- Autovacuum lag producing bloat and poor plans.
- "Backup succeeded" without verified restore workflow.
- Replication slots causing unbounded WAL growth.
- Security drift from broad grants and unmanaged extensions.

## Definition of Done
- Baseline, change, and outcome are documented with metrics.
- Query-plan and lock-impact checks passed for critical paths.
- Backup/restore and replication readiness verified.
- Security controls reviewed against least-privilege policy.
- Runbooks updated for on-call operations.

## References
- `references/postgresql`

## Reference Index
- `rg -n "^## Configuration|^### Recommended patterns" references/postgresql`
- `rg -n "^## Schema design|^## Indexing" references/postgresql`
- `rg -n "EXPLAIN|ANALYZE|index|partial" references/postgresql`
- `rg -n "autovacuum|vacuum|bloat|wraparound" references/postgresql`
- `rg -n "backup|restore|replication|high availability|RPO|RTO" references/postgresql`
- `rg -n "security|privilege|TLS|extension" references/postgresql`
- `rg -n "upgrade|migration|wal_keep_size|version" references/postgresql`

## Quick Questions (When Stuck)
- What is the smallest safe change that can be measured quickly?
- Which SLO is currently at risk: latency, availability, or durability?
- Is this a planner/stats problem, an I/O problem, or a lock problem?
- Can this be fixed with scoped settings instead of global changes?
- What evidence proves recovery works today?
