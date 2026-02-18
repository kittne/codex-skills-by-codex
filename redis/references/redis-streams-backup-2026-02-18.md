# Redis Streams + Backup/Recovery Reference (2026-02-18)

## Context7 Sources
- Library: `/redis/redis-doc`
- Streams docs for `XREADGROUP`, `XPENDING`, `XINFO GROUPS`, `XAUTOCLAIM`.

## Streams Operational Guidance
- Use consumer groups for coordinated processing and replay support.
- Inspect backlog health with `XPENDING` and group/consumer details.
- Reclaim stale pending entries with `XAUTOCLAIM` and bounded batch sizes.
- Acknowledge only after successful processing (`XACK`).
- Keep stream length bounded via retention/trim policy.

## Backup and Recovery Guidance
- Select persistence policy from required durability (`RDB`, `AOF`, or both).
- For Streams durability, validate persistence and failover behavior under load.
- Back up persistence artifacts and test end-to-end restore frequently.
- Include consumer lag and PEL size in operational dashboards.

## Source URLs
- <https://github.com/redis/redis-doc>
- <https://redis.io/docs/latest/develop/data-types/streams/>
