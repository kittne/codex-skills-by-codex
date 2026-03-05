---
name: redis
description: Design, implement, and operate Redis (6+) safely and efficiently. Use for choosing Redis's role (cache/session/rate-limit/queue), key and data-structure modeling, TTL strategy, memory sizing and eviction policies, persistence (RDB/AOF) tradeoffs, HA (replication+Sentinel vs Cluster), performance and latency debugging (slowlog/latency, blocking commands), security (ACLs/TLS/network), container/Kubernetes deployment, monitoring, backups/DR, and anti-pattern review.
---

# Redis

## Workflow
1. Confirm Redis role and acceptable data-loss window.
2. Baseline memory, latency, and command patterns.
3. Design keys/data structures with explicit TTL and retention policies.
4. Select persistence and HA topology from RPO/RTO needs.
5. Implement Streams consumers with recovery and replay strategy when required.
6. Define backup/restore and disaster recovery procedures.
7. Validate security and operational readiness before rollout.

## Preflight (Ask / Check First)
- Redis version and topology: standalone, Sentinel, or Cluster.
- Workload roles: cache, queue, streams, session, leaderboard.
- Memory size, growth forecast, and peak ops/sec.
- Latency SLOs and tolerated loss window.
- Network exposure, TLS, ACLs, and secrets handling.
- Existing incident history: evictions, failover issues, big-key stalls.
- Upgrade posture: confirm `redis_version` and review breaking changes before major upgrades.

## Data Modeling and TTL
- Use consistent key namespaces: `{app}:{env}:{entity}:{id}`.
- Keep values and collections bounded; shard by time/entity when needed.
- Use hashes for object-like records and sorted sets for ranking/time windows.
- Apply TTL to cache-like keys and add expiration jitter.
- Avoid unbounded list/set/stream growth without retention policy.

## Memory and Eviction
- Always set `maxmemory` with headroom for replication and persistence overhead.
- Use `allkeys-lfu`/`allkeys-lru` for cache-heavy workloads.
- Use `volatile-*` only when critical keys must not be evicted.
- Track fragmentation, evictions, and hot keys continuously.

## Performance and Latency
- Avoid blocking commands in hot paths (`KEYS`, huge `HGETALL`, large `LRANGE`).
- Use `SCAN` family for keyspace traversal.
- Prefer `UNLINK` for large key deletion when available.
- Use pipelining and Lua to reduce chatty round-trips.
- Correlate latency spikes with AOF rewrite/fork events.

### Baseline Diagnostics
```bash
redis-cli INFO memory
redis-cli INFO replication
redis-cli INFO persistence
redis-cli SLOWLOG GET 20
redis-cli LATENCY LATEST
```

## Streams Architecture
- Use Streams only when you need durability/replay semantics beyond pub/sub.
- Model stream names per bounded domain and retention policy.
- Use consumer groups for horizontal consumers with explicit ownership.
- Define ack policy and retry/dead-letter flow for poisoned messages.
- Control pending-entry growth with consumer lag monitoring.

## Streams Consumer Recovery
- Inspect pending backlog with `XPENDING` and consumer details.
- Reclaim abandoned messages with `XAUTOCLAIM` after idle threshold.
- Keep replay limits and idempotency keys in downstream processors.
- For Redis 8.6+, use stream idempotency (`IDMP`/`IDMPAUTO`) when duplicate appends are high risk.
- Trim streams intentionally (`MAXLEN`) to control storage growth.

### Streams Ops Commands
```bash
redis-cli XINFO GROUPS mystream
redis-cli XPENDING mystream mygroup
redis-cli XAUTOCLAIM mystream mygroup worker-2 60000 0-0 COUNT 50
redis-cli XTRIM mystream MAXLEN ~ 100000
```

## Persistence, Backup, and Recovery
- Choose persistence mode explicitly: none, RDB, AOF, or hybrid.
- For durability-sensitive workloads, prefer AOF with measured fsync policy.
- Schedule and verify backups of RDB/AOF artifacts.
- Keep restore runbooks for full-node loss and corruption scenarios.
- Rehearse restore and failover regularly; record measured RTO.

## HA and Disaster Recovery
- Use Sentinel for single-primary HA without sharding.
- Use Cluster when sharding is required; design multi-key operations with hash tags.
- Validate client failover behavior under planned cutover drills.
- Keep cross-zone/region replication and restore strategy explicit.

## Security Defaults
- Never expose Redis directly to public internet.
- Enforce ACL least privilege (service-specific users and key patterns).
- On Redis 8+, re-audit ACL category grants because module commands are included in core categories.
- Use TLS on untrusted networks and validate certificates.
- Keep secrets in secret stores, not plaintext config files.
- Audit use of dangerous/admin commands.

## Validation Commands
```bash
redis-cli PING
redis-cli INFO
redis-cli --rdb /tmp/redis-backup-test.rdb
```

## Common Failure Modes
- Using Redis as system-of-record without durability design.
- Missing TTL/retention causing memory saturation.
- Consumer groups accumulating unbounded pending entries.
- Streams without idempotent consumers or retry policies.
- Backups existing but restore never tested.

## Definition of Done
- Data model and TTL policy are bounded and documented.
- Streams usage includes recovery, replay, and trimming policy.
- Persistence/backup strategy matches RPO/RTO requirements.
- Failover and restore drills are tested and measured.
- Security controls and observability are production-ready.

## References
- `references/redis-best-practices.md`
- `references/redis-streams-backup-2026-02-18.md`

## Reference Index
- `rg -n "Streams|consumer group|XPENDING|XAUTOCLAIM" references/redis-streams-backup-2026-02-18.md`
- `rg -n "backup|restore|RDB|AOF|RPO|RTO" references/redis-streams-backup-2026-02-18.md`
- `rg -n "Memory|eviction|latency" references/redis-best-practices.md`
- `rg -n "ACL|TLS|security" references/redis-best-practices.md`
