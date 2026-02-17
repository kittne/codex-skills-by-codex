---
name: redis
description: Design, implement, and operate Redis (6+) safely and efficiently. Use for choosing Redis's role (cache/session/rate-limit/queue), key and data-structure modeling, TTL strategy, memory sizing and eviction policies, persistence (RDB/AOF) tradeoffs, HA (replication+Sentinel vs Cluster), performance and latency debugging (slowlog/latency, blocking commands), security (ACLs/TLS/network), container/Kubernetes deployment, monitoring, backups/DR, and anti-pattern review.
---

# Redis

## Working Style

- Start by clarifying Redis's role (cache vs durable-ish store) and the acceptable data-loss window.
- Make memory behavior explicit: `maxmemory` and eviction policy must be intentional in production.
- Design keys and data structures for both latency and memory efficiency.
- Avoid blocking the main thread: big-key operations and full scans are common outage causes.

## Quick Intake (Ask Before Recommending Changes)

- Redis version and deployment: standalone, Sentinel HA, or Cluster (sharded).
- Role: cache, sessions, rate limiting, pub/sub, queueing/streams, leaderboards, etc.
- Durability needs: none vs RDB vs AOF (and acceptable data loss).
- Memory: current dataset size, key counts, largest keys, expected growth, headroom.
- Latency SLOs and peak ops/sec; client count and connection pattern.
- Security: network exposure, ACL usage, TLS requirements, secret management.
- Platform: VMs vs containers vs Kubernetes; resource limits; persistence volumes.

## Default Design Rules

- Key naming:
  - Use a stable namespace pattern like `{app}:{env}:{entity}:{id}:{attr}`.
  - Keep keys short but readable; every byte matters at scale.
- Data modeling:
  - Prefer hashes for object-like data instead of many small keys.
  - Prefer sorted sets for ranking/time windows; streams for consumer-group patterns.
  - Avoid huge single keys; shard by entity/time.
- TTL:
  - Apply TTL by default for caches.
  - Add jitter to reduce synchronized expirations ("thundering herd").

## Memory And Eviction (Production Defaults)

- Always set `maxmemory` and leave headroom for replication buffers, AOF rewrite, and fragmentation.
- Choose eviction policy by role:
  - Pure cache: `allkeys-lfu` (or `allkeys-lru`).
  - Mixed critical + cache keys: TTL only for evictable keys and use `volatile-*`.
  - No eviction allowed: `noeviction` plus strict capacity planning and alerts.
- Watch `mem_fragmentation_ratio` and large-key patterns; investigate spikes.

## Performance And Latency Debugging

- Avoid blocking commands on hot paths:
  - Use `SCAN` family instead of `KEYS`.
  - Prefer `UNLINK` over `DEL` for very large keys where supported.
  - Avoid `HGETALL` / `SMEMBERS` / big `LRANGE` on huge structures.
- Reduce round trips:
  - Pipeline where safe.
  - Use Lua for multi-step atomic operations when it replaces chatty client logic.
- Instrument:
  - Use `SLOWLOG`, `LATENCY`, and `INFO` metrics to correlate spikes with persistence forks, eviction, or IO.

Common debugging commands:
```bash
redis-cli INFO memory
redis-cli INFO replication
redis-cli SLOWLOG GET 10
redis-cli LATENCY LATEST
redis-cli --scan --pattern 'app:prod:*' | head
redis-cli MEMORY USAGE 'some:key'
```

## Durability And HA Choices

- Persistence:
  - Cache-only: consider disabling persistence if total loss is acceptable.
  - Common balance: AOF with `appendfsync everysec` (measure latency and disk IO).
  - Measure fork and copy-on-write costs for RDB/AOF rewrites on real dataset sizes.
- HA:
  - Sentinel: good for a single primary with replicas (no sharding).
  - Cluster: use when you must shard; design multi-key operations for hash slots (hash tags `{...}`).
  - Validate client behavior for failover (Sentinel-aware vs cluster client).

## Security Defaults

- Never expose Redis directly to the public internet.
- Use ACLs (one user per service, least privilege, key prefix restrictions).
- Enable TLS when traffic crosses untrusted networks; ensure clients validate certs.
- Consider disabling/renaming dangerous commands in prod (with awareness of tooling impact).

## Production Readiness Checklist
- Memory:
  - `maxmemory` set with headroom; eviction policy chosen intentionally.
  - Alerts on RSS, fragmentation ratio, evictions, and latency spikes.
- Durability/HA:
  - Persistence mode chosen explicitly (none/RDB/AOF/hybrid) and tested.
  - Failover strategy validated (Sentinel-aware clients or Cluster clients).
- Operations:
  - Backups/DR plan exists (what to do if disk corrupts, node dies, cluster reshard needed).
  - Upgrade plan (version drift, rolling restart, compatibility).
- Security:
  - Network restricted, ACLs enforced, secrets not in repos, TLS if required.

## Common Anti-Patterns (Call Out)
- Using Redis as a primary database without an explicit durability/consistency plan.
- Running without `maxmemory`/eviction settings in production.
- Huge keys (multi-MB values or massive collections) on the hot path.
- `KEYS` in production, or large `SMEMBERS/HGETALL/LRANGE` in request paths.
- Using pub/sub for workloads that need durability or replay (use Streams/queues instead).

## Reference

- Use `references/redis-best-practices.md` for the full guide, checklists, and snippets.
- Find topics quickly:
  - `rg -n '^## Data modeling' references/redis-best-practices.md`
  - `rg -n '^## Memory management and eviction' references/redis-best-practices.md`
  - `rg -n '^## Persistence and durability' references/redis-best-practices.md`
  - `rg -n '^## High availability and clustering' references/redis-best-practices.md`
  - `rg -n '^## Performance and latency' references/redis-best-practices.md`
  - `rg -n '^## Security' references/redis-best-practices.md`

## Extended Guidance
Use this when Redis is critical to availability or latency.

## Key Design Checklist (Expanded)
- Prefix keys with service and entity (`svc:user:123`).
- Keep key cardinality measurable and predictable.
- Avoid keys that grow without TTL or retention policy.

## Latency Debugging Commands (Expanded)
```bash
redis-cli INFO
redis-cli SLOWLOG GET 10
redis-cli LATENCY DOCTOR
```

## Persistence Decisions (Expanded)
- Use AOF when durability matters; tune fsync for latency.
- Use RDB snapshots when you can tolerate some data loss.

## Common Failure Modes
- No TTLs for cache keys (memory grows without bound).
- Hot keys concentrate load on a single shard.
- Large value payloads causing fragmentation and slow operations.

## Reference Index
- `rg -n "TTL|expiration" references/redis-best-practices.md`
- `rg -n "SLOWLOG|latency" references/redis-best-practices.md`

## Operational Reminders
- Validate failover behavior quarterly.

## Quick Questions (When Stuck)
- What is the minimal change that solves the issue?
- What is the rollback plan?
- What is the highest-risk assumption?
- What is the simplest validation step?
- What is the known-good baseline?
- What evidence would change the decision?
- What is the user-visible impact?
- What is the operational impact?
- What is the most likely failure mode?
- What is the fastest safe experiment?

## Reference Index (Extra)
- `rg -n "Checklist|checklist" references/redis-best-practices.md`
- `rg -n "Example|examples" references/redis-best-practices.md`
- `rg -n "Workflow|process" references/redis-best-practices.md`
- `rg -n "Pitfall|anti-pattern" references/redis-best-practices.md`
- `rg -n "Testing|validation" references/redis-best-practices.md`
- `rg -n "Security|risk" references/redis-best-practices.md`
- `rg -n "Configuration|config" references/redis-best-practices.md`
- `rg -n "Deployment|operations" references/redis-best-practices.md`
- `rg -n "Troubleshoot|debug" references/redis-best-practices.md`
- `rg -n "Performance|latency" references/redis-best-practices.md`
- `rg -n "Reliability|availability" references/redis-best-practices.md`
- `rg -n "Monitoring|metrics" references/redis-best-practices.md`
- `rg -n "Error|failure" references/redis-best-practices.md`
- `rg -n "Decision|tradeoff" references/redis-best-practices.md`
- `rg -n "Migration|upgrade" references/redis-best-practices.md`
