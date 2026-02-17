## Redis Best Practices

# Redis Best Practices

This guide is a practical, “use-anytime” reference for working with Redis in both **development** and **production** environments. It is written to be broadly applicable (general → advanced) and is suitable as source material for a coding/deployment skill.

> Scope notes:
>
> * Recommendations assume **modern Redis (6+)** features and defaults where possible (ACLs, improved replication, etc.).
> * Redis is an in-memory system; performance and reliability are tightly coupled to memory and I/O choices.

---

## Table of Contents

* [Core principles](#core-principles)
* [Development vs production](#development-vs-production)
* [Data modeling](#data-modeling)
* [Memory management and eviction](#memory-management-and-eviction)
* [Persistence and durability](#persistence-and-durability)
* [High availability and clustering](#high-availability-and-clustering)
* [Performance and latency](#performance-and-latency)
* [Security](#security)
* [Operational configuration and OS tuning](#operational-configuration-and-os-tuning)
* [Containers and Kubernetes](#containers-and-kubernetes)
* [Monitoring and observability](#monitoring-and-observability)
* [Backups, DR, and upgrades](#backups-dr-and-upgrades)
* [Common anti-patterns](#common-anti-patterns)
* [Checklists](#checklists)

---

## Core principles

1. **Pick the right role for Redis**: cache, session store, ephemeral queueing, rate limiting, pub/sub, leaderboard, etc.
2. **Design keys and data structures intentionally**; model for memory efficiency and latency.
3. **Avoid blocking operations** and large key operations on the main thread.
4. **Choose a persistence + HA strategy aligned with your durability needs**.
5. **Make eviction behavior explicit**; don’t let Redis crash due to OOM surprises.
6. **Instrument everything**: memory, latency, slowlog, hit rate, replication/cluster state.

---

## Development vs production

### Development best practices

* Run Redis with the same major version/features as production whenever possible.
* Use a default `maxmemory` in dev to surface memory bloat early.
* Enable useful diagnostics:

  * `slowlog` with a low threshold
  * periodic `INFO` snapshots
* Use namespaces/prefixes per project to avoid key collisions.

### Production best practices

* Run Redis on dedicated nodes (or dedicated resource pools) when possible.
* Configure `maxmemory` + eviction policy explicitly.
* Choose persistence: none / RDB / AOF / hybrid based on requirements.
* Choose HA strategy: replication + Sentinel vs Redis Cluster (sharding).
* Lock down network access and enable ACLs and TLS where applicable.
* Have a plan for upgrades, backups, and disaster recovery.

---

## Data modeling

### Key naming conventions

Use a consistent, hierarchical pattern:

```
{app}:{env}:{entity}:{id}:{attribute}
```

Examples:

* `billing:prod:user:123:profile`
* `shop:staging:cart:abc123`
* `rate:prod:ip:203.0.113.4`

Best practices:

* Keep keys **short but readable** (every byte matters at scale).
* Use `:` as a conventional delimiter.
* Include environment or tenant scope explicitly (`prod`, `tenant42`).
* Prefer deterministic keys; avoid random keys unless required.

### Choose the right data structure

Redis primitives matter for memory and operations:

* **String**: simplest; good for small values, counters, tokens.
* **Hash**: best for “object-like” data (many fields); usually more memory-efficient than many separate keys.
* **List**: queues, streams of events (but Streams are often better for durable queues).
* **Set**: membership tests, uniqueness.
* **Sorted Set (ZSET)**: leaderboards, ranking, time-ordered indexes.
* **Stream**: durable-ish messaging patterns with consumer groups.

Rules of thumb:

* Prefer **hashes** over “one key per field” for object storage.
* Prefer **sorted sets** for ranking and time windows.
* Prefer **streams** for log/event processing where you need consumer groups.

### Avoid memory bloat

* Store only what you need; avoid large JSON blobs if you can store compact fields.
* Compress large values only if CPU tradeoffs make sense (and you measure).
* Avoid huge single keys (e.g., a 100MB list); shard data by entity/time.
* Use TTLs on cache entries unless they are truly permanent.

### TTL strategy

* For caches, apply TTL by default.
* Use **staggered TTLs** (jitter) to avoid “thundering herd” expirations:

  * base TTL ± random small delta
* Use “soft TTL” patterns when needed:

  * serve stale while recomputing in background (if your app supports it)

---

## Memory management and eviction

### Set maxmemory explicitly

Always configure `maxmemory` in production to avoid host OOM kills.

Guidance:

* Leave headroom for:

  * replication buffers
  * AOF rewrite buffers
  * OS overhead
  * fragmentation

### Choose an eviction policy aligned to your use case

Common policies:

* `noeviction`: writes fail when maxmemory reached (good for “must not evict” workloads).
* `allkeys-lru` / `allkeys-lfu`: evict least recently/frequently used keys among all keys (common for caches).
* `volatile-lru` / `volatile-lfu`: evict among keys with TTL only.
* `volatile-ttl`: evict keys with nearest expiration.

Recommendations:

* **Pure cache**: `allkeys-lfu` or `allkeys-lru`.
* **Mixed** (some keys must not be evicted): use TTL for evictable keys + `volatile-*`.
* **Durable/critical** (no eviction acceptable): `noeviction` + strict capacity planning.

### Track fragmentation

Memory fragmentation can cause Redis to use more RSS than expected.

* Monitor `mem_fragmentation_ratio` in `INFO memory`.
* Plan for overhead; fragmentation isn’t automatically “bad,” but spikes can indicate allocation patterns or big key operations.

---

## Persistence and durability

Redis offers:

* **RDB snapshots**: periodic point-in-time dumps.
* **AOF (append-only file)**: logs each write.
* **Hybrid**: AOF with RDB preamble (faster restarts).

### Selecting persistence

* **Cache-only**: consider disabling persistence (fast, simple) *if* you can tolerate total cache loss.
* **Recoverable cache / session store**: consider RDB or AOF everysec depending on tolerance.
* **Durability-focused**: AOF with `appendfsync everysec` is a common balance.

### AOF best practices

* Use `appendfsync everysec` for most production uses (durability vs performance).
* Monitor AOF rewrite behavior (`BGREWRITEAOF`):

  * ensure disk IO can handle rewrites
  * ensure you have enough disk space for rewrite + existing file
* Avoid `appendfsync always` unless absolutely needed; it can be slow and increases latency.

### RDB best practices

* Configure snapshot frequency based on data criticality and write rate.
* Ensure the RDB directory is on reliable storage.
* Monitor fork time and copy-on-write memory impact (big dataset + snapshot can spike memory usage).

### Persistence pitfalls

* Persistence can increase latency spikes due to fork and I/O.
* Always measure with your dataset size and write patterns.
* If using both AOF and RDB, understand startup priority and file integrity checks.

---

## High availability and clustering

### Replication (primary/replica)

* Use replicas for:

  * failover (with Sentinel)
  * read scaling (if your app can tolerate replication lag)
* Monitor replication lag and replication link health (`INFO replication`).
* Set up proper `replicaof` configurations (or cluster-managed replication).

### Sentinel (automatic failover)

Use Sentinel when:

* you want automatic failover for a single primary with replicas
* you don’t need sharding (single keyspace)

Best practices:

* Run at least **3 Sentinels** on independent nodes.
* Ensure Sentinels can communicate reliably.
* Validate client behavior:

  * clients must discover the new primary (use Sentinel-aware clients)

### Redis Cluster (sharding)

Use Cluster when:

* your dataset exceeds single-node memory/CPU constraints
* you need to scale writes horizontally by sharding keys

Best practices:

* Use at least 3 masters + replicas for production.
* Use hash tags `{...}` carefully to control key co-location (for multi-key operations).
* Remember:

  * multi-key operations require keys in same hash slot
  * some commands behave differently under cluster
* Test failover and resharding procedures.

### Architecture diagram examples

**Sentinel HA:**

```
Clients -> Sentinel-aware client -> [Primary] <-> [Replica]
                          \-> [Sentinel1][Sentinel2][Sentinel3]
```

**Cluster (sharding + HA):**

```
Clients -> Cluster client -> [M1]-[R1]  [M2]-[R2]  [M3]-[R3]
                      (keys distributed by hash slots)
```

---

## Performance and latency

Redis is fast because it’s mostly single-threaded for command execution; avoid blocking that thread.

### Avoid blocking commands

Avoid on hot paths:

* `KEYS` (use `SCAN`)
* `SMEMBERS` on huge sets
* `LRANGE` for huge lists
* `HGETALL` for huge hashes
* `DEL` on very large keys (use `UNLINK` for async freeing where supported)
* `MONITOR` in production

### Prefer SCAN-family

Use:

* `SCAN`, `SSCAN`, `HSCAN`, `ZSCAN`
  instead of `KEYS` or other full scans.

### Reduce round trips

* Use pipelining for multiple operations.
* Use multi/exec carefully for atomic batches (but don’t overuse).
* Consider Lua scripts for multi-step atomic operations to avoid multiple network calls.

### Use client-side caching patterns when appropriate

* Cache frequently used results locally in the service (with TTL).
* Use Redis for shared cache + client-side cache for ultra-low latency.

### Connection management

* Use persistent connections and a connection pool.
* Avoid too many clients:

  * high client count increases memory and CPU overhead
* Use timeouts and keepalive.

### Measure latency

* Use `LATENCY` commands and `SLOWLOG`:

  * identify spikes
  * correlate with persistence forks, eviction, network issues

---

## Security

### Network exposure

* Prefer binding Redis to localhost or private network interfaces only.
* Put Redis behind firewalls/security groups. Redis should not be directly internet-facing.

### Authentication and ACLs

* Use ACLs (Redis 6+) to create least-privilege users:

  * one user per application/service
  * allow only required command categories and key prefixes
* Avoid sharing the default user credentials among services.

### TLS

* If Redis traffic crosses untrusted networks, enable TLS.
* Ensure clients validate certificates.

### Command renaming / disabling

For defense-in-depth, consider renaming or disabling dangerous commands in production environments:

* `FLUSHALL`, `FLUSHDB`, `CONFIG`, `DEBUG`, `SHUTDOWN`, etc.

Be careful: renaming commands can break tooling and client libraries.

### Secrets handling

* Do not hardcode Redis passwords in code.
* Use a secret manager and rotate credentials.

---

## Operational configuration and OS tuning

Redis performance depends heavily on the host OS and kernel settings.

Common production host best practices:

* Disable Transparent Huge Pages (THP) to reduce latency spikes.
* Set `vm.overcommit_memory=1` (commonly recommended for Redis workloads).
* Ensure adequate file descriptors (`ulimit -n`).
* Prefer low-latency storage if using AOF/RDB.
* Avoid swap; or set low swappiness; Redis latency can degrade heavily with swapping.

Always validate these settings in your environment and follow your organization’s OS policy.

---

## Containers and Kubernetes

### Docker/Compose

* Mount a persistent volume if you use persistence.
* Make memory limits explicit and align them with Redis `maxmemory`.
* Avoid CPU starvation; Redis latency suffers under throttling.

### Kubernetes

* Use StatefulSets for stable identity and storage (if persisting).
* Use anti-affinity to separate primaries and replicas.
* Consider PodDisruptionBudgets for availability.
* Ensure network policies restrict access to Redis.
* Avoid running Redis on preemptible nodes unless you can tolerate data loss.

---

## Monitoring and observability

### Key metrics to monitor

From `INFO`:

* `used_memory`, `used_memory_rss`, `mem_fragmentation_ratio`
* `evicted_keys`
* `connected_clients`
* `instantaneous_ops_per_sec`
* `keyspace_hits`, `keyspace_misses` (compute hit rate)
* replication offsets and lag
* persistence stats (AOF rewrite, RDB save times)

### Slowlog

Configure slowlog to capture expensive commands:

* inspect entries regularly
* correlate with client endpoints

### Tooling

* RedisInsight (GUI) for exploration/debugging
* Prometheus exporter (redis_exporter) + Grafana dashboards
* Centralized logging for config changes and operational events

---

## Backups, DR, and upgrades

### Backups

* If using RDB, snapshot files can serve as backup artifacts; copy them off-host.
* If using AOF, back it up carefully (ideally with filesystem snapshots) to preserve consistency.

### Disaster recovery

* Document restore procedures:

  * restore RDB/AOF
  * reattach replicas
  * validate clients reconnect correctly
* Test restore in staging.

### Upgrades

* Plan upgrades:

  * test compatibility with clients and modules
  * rolling upgrade strategy depends on whether you use Sentinel or Cluster
* For Sentinel:

  * upgrade replicas first, then primary
* For Cluster:

  * upgrade in a controlled rolling manner; validate slot map and failover behavior

---

## Common anti-patterns

* Treating Redis as a primary durable database without a durability/DR plan.
* Allowing unbounded key growth (no TTL, no cleanup).
* Using `KEYS` in production.
* Storing huge blobs and frequently rewriting them.
* Too many small keys instead of hashes (increases overhead).
* Not setting `maxmemory` and relying on the OS to kill processes.
* Not monitoring hit rate and eviction; “cache” becomes random under pressure.

---

## Checklists

### Data modeling checklist

* [ ] key naming includes namespace/tenant/environment
* [ ] TTL applied for cache entries by default
* [ ] hashes used for object-like data instead of many keys
* [ ] large-key operations avoided
* [ ] multi-key patterns designed for cluster (hash tags if needed)

### Production readiness checklist

* [ ] `maxmemory` set and eviction policy chosen
* [ ] persistence configured appropriately (or explicitly disabled)
* [ ] HA strategy chosen (Sentinel vs Cluster) and tested
* [ ] ACL users defined per service, least privilege
* [ ] network access restricted (firewalls/policies)
* [ ] monitoring dashboards for memory, latency, evictions, hit rate
* [ ] backup/restore procedure documented and tested
* [ ] upgrade plan documented

---

## Quick reference snippets

### Avoid KEYS

Bad:

```sh
redis-cli KEYS "app:prod:user:*"
```

Better:

```sh
redis-cli --scan --pattern "app:prod:user:*"
```

### Use UNLINK for large keys (async free)

```sh
redis-cli UNLINK app:prod:bigkey
```

### Keyset with sorted set (leaderboard)

```sh
# increment score
ZINCRBY leaderboard 10 user:123

# top N
ZREVRANGE leaderboard 0 99 WITHSCORES
```

---