---
name: postgresql
description: >
  Design, query, tune, secure, and operate PostgreSQL clusters for production workloads. Use when
  working on schema design, indexing/query plans, transaction behavior, autovacuum/bloat,
  backup/restore, replication and HA/DR, observability, security hardening, or upgrades.
---

# PostgreSQL

## Workflow
1. Confirm PostgreSQL version, hosting model, and business RPO/RTO.
2. Baseline workload shape, config, and cluster health before changes.
3. Optimize schema/index/query behavior for critical paths.
4. Validate concurrency, locking, vacuum, and bloat controls.
5. Implement backup/restore and PITR procedures with evidence.
6. Harden security and operational governance.
7. Roll out incrementally with measurable before/after metrics.

## Preflight (Ask / Check First)
- Current major/minor version and upgrade target.
- Managed service vs self-managed constraints.
- Latency/availability/durability SLO priorities.
- WAL volume, replication topology, and lag behavior.
- Known incidents: lock contention, bloat, restore gaps.

## Configuration and Capacity
- Align `max_connections` with pooler strategy.
- Balance memory settings against real concurrency.
- Smooth checkpoints to reduce latency cliffs.
- Keep autovacuum globally enabled and table-tuned for churn.
- Verify effective settings from `pg_settings`, not assumptions.

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

## Schema, Indexing, and Query Plans
- Keep constraints explicit (PK/FK/UNIQUE/CHECK/NOT NULL).
- Use multi-column and partial indexes only for observed query patterns.
- Use `EXPLAIN (ANALYZE, BUFFERS)` for truth.
- Track index bloat and unused index overhead.
- Recheck plans after major data distribution shifts.

## Transactions and Concurrency
- Keep transactions short and retry-safe.
- Use lowest viable isolation level for correctness.
- Monitor lock waits and idle-in-transaction sessions.
- Guard against long-running transactions blocking vacuum.

### Lock Insight Query
```sql
SELECT pid, wait_event_type, wait_event, state, query
FROM pg_stat_activity
WHERE wait_event_type IS NOT NULL
ORDER BY state, pid;
```

## Vacuum and Bloat Control
- Monitor dead tuples and autovacuum recency per table.
- Tune table-level autovacuum thresholds for hot tables.
- Watch transaction ID age and wraparound risk.
- Reserve `VACUUM FULL` for planned maintenance windows.

## Backup and Restore Strategy
- Derive strategy from RPO/RTO, then pick tooling.
- Use base backup + WAL archiving for PITR-capable systems.
- Use logical dumps for selective migrations, not PITR.
- Keep backup storage encrypted and access-controlled.
- Validate backup chain continuity and restore prerequisites.

## PITR and Recovery Operations
- Configure `archive_command` and verify WAL archiving health.
- Test `restore_command` paths in isolated recovery environments.
- Run point-in-time restore drills to explicit timestamps.
- Measure recovery duration against RTO and adjust playbooks.
- Validate application behavior after promotion/failover.

### Operational Backup Commands
```bash
pg_basebackup -D /var/backups/pg/base -X stream -c fast -P
pg_dump -Fc -f /var/backups/pg/app.dump appdb
pg_restore -d appdb_restore --clean --if-exists /var/backups/pg/app.dump
```

## Replication and HA/DR
- Monitor replication lag and slot WAL retention growth.
- Define failover, promotion, and replica rejoin runbooks.
- Test failover with application reconnect behavior.
- Keep region-level DR strategy explicit and validated.

## Security and Governance
- Enforce least privilege roles and default-deny grants.
- Restrict network paths and require TLS where possible.
- Keep secrets out of scripts and logs.
- Review extension allowlist and privileged operations.
- Audit high-impact DDL and superuser activity.

## Validation Commands
```bash
psql -X -d "$DATABASE_URL" -c "SELECT version();"
psql -X -d "$DATABASE_URL" -c "SHOW wal_level;"
psql -X -d "$DATABASE_URL" -c "SELECT now();"
```

## Common Failure Modes
- Treating successful backup jobs as proof of recoverability.
- Missing WAL archives for intended PITR windows.
- Oversized `work_mem` causing memory pressure under concurrency.
- Autovacuum lag creating bloat and planner regressions.
- Replication slots retaining WAL until disk exhaustion.

## Definition of Done
- Capacity, query, and lock baselines are documented.
- Backup and PITR procedures are tested and timed.
- Replication and failover workflows are validated.
- Security controls match least-privilege policy.
- Runbooks are current and on-call ready.

## References
- `references/postgresql`
- `references/postgresql-backup-restore-2026-02-18.md`

## Reference Index
- `rg -n "backup|restore|PITR|archive_command|restore_command" references/postgresql-backup-restore-2026-02-18.md`
- `rg -n "replication|slot|lag|failover" references/postgresql-backup-restore-2026-02-18.md`
- `rg -n "autovacuum|bloat|lock|EXPLAIN" references/postgresql`
- `rg -n "security|TLS|privilege" references/postgresql`
