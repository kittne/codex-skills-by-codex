---
name: loki
description: >
  Design and operate Grafana Loki log platforms with efficient label strategy, query performance,
  and retention controls. Use for ingestion pipelines, label cardinality governance,
  LogQL optimization, tenancy boundaries, and production troubleshooting.
---

# Loki

## Workflow
1. Confirm log use cases, retention targets, and compliance requirements.
2. Design label schema with strict cardinality budget.
3. Configure ingestion pipeline and parsing stages.
4. Implement tenant boundaries and access controls.
5. Build performant LogQL query patterns.
6. Tune retention, compaction, and storage costs.
7. Validate incident workflows and query latency.

## Preflight (Ask / Check First)
- Loki version and deployment mode.
- Ingestion rate and expected stream counts.
- Required labels for alerting and triage.
- Multi-tenant isolation needs.
- Current query timeout/error patterns.

## Label Strategy and Cardinality
- Use labels for coarse dimensions only (service, environment, namespace).
- Keep high-cardinality values out of labels (request ID, trace ID, session ID).
- Bound label value growth through ingestion policy.
- Review stream count impact before adding new labels.
- Prefer structured log fields for detailed dimensions.

## Ingestion and Pipeline Rules
- Parse JSON/logfmt only when needed for downstream queries.
- Drop or keep selected fields to reduce noisy payload expansion.
- Normalize key names across services.
- Keep parsing stages deterministic and observable.

## Query Performance
- Start queries with the narrowest label matchers.
- Use exact matches instead of broad regex when possible.
- Filter left-to-right to cut candidate streams early.
- Aggregate with `sum by`/`without` to control output cardinality.
- Profile slow queries and codify better defaults in dashboards.

### Example Query Pattern
```logql
sum by (status) (rate({job="api", namespace="production"} | json [5m]))
```

## Retention and Operations
- Define retention by tenant/classification policy.
- For new installs, use TSDB schema (v13) with 24h index period.
- Retention requires 24h index period plus compactor `retention_enabled=true`.
- Avoid Cassandra for new deployments; it is deprecated as a Loki storage backend.
- Loki 3.0 requires TSDB v13 for structured metadata and removes `shared_store`; migrate before upgrade.
- Loki 3.0 enforces 256KB max line size and reduces label limit per series to 15; budget labels accordingly.
- Monitor ingestion errors, compaction lag, and query tail latency.
- Plan object storage costs with realistic growth models.
- Keep backup strategy for critical config and metadata.
- Rehearse upgrades and schema migrations.

## Security and Multi-Tenancy
- Enforce tenant-aware authN/authZ boundaries.
- Restrict write/read paths by tenant role.
- Redact sensitive fields before ingestion where possible.
- Audit access to sensitive log streams.

## Validation Commands
```bash
curl -fsS "$LOKI/ready"
curl -G -fsS "$LOKI/loki/api/v1/query" --data-urlencode 'query={job="api"}'
```

## Common Failure Modes
- Label cardinality explosion from dynamic values.
- Regex-heavy queries over broad stream selectors.
- Missing retention controls causing storage blowouts.
- Pipeline parsing drift across services.
- Weak tenant boundaries in shared environments.

## Definition of Done
- Label schema is cardinality-safe and documented.
- Ingestion pipeline is deterministic and observable.
- Query performance meets operational latency targets.
- Retention and tenancy policies are enforced.
- Recovery and upgrade procedures are tested.

## Operational Checklist
- Label set review completed for each onboarded service.
- Query templates exist for top incident classes.
- Ingestion error alerts are configured and owned.
- Retention policies are enforced per tenant/environment.
- Slow-query dashboards track p95 and p99 query duration.
- Storage growth budgets are tracked monthly.
- Upgrade compatibility checks are documented.
- Access control policies are tested after permission changes.

## References
- `references/loki-2026-02-18.md`

## Reference Index
- `rg -n "label|cardinality|stream" references/loki-2026-02-18.md`
- `rg -n "query|LogQL|performance" references/loki-2026-02-18.md`
- `rg -n "pipeline|drop|keep|parse" references/loki-2026-02-18.md`
- `rg -n "retention|tenancy|operations" references/loki-2026-02-18.md`
