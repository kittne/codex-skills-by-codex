---
name: prisma
description: >
  Design, query, and operate Prisma ORM safely in production Node.js services. Use when defining
  Prisma schema models, planning migrations and versioning, optimizing query behavior, enforcing
  data integrity/security, and maintaining testing, observability, and upgrade workflows.
---

# Prisma

## Workflow
1. Confirm database engine, Prisma version, and deployment constraints.
2. Model schema with explicit relations, constraints, and naming conventions.
3. Plan migration lifecycle (dev, staging, production) before schema changes.
4. Implement query patterns with performance and transaction discipline.
5. Apply integrity and security controls at app and DB boundaries.
6. Add tests for schema evolution and query correctness.
7. Instrument operational visibility for slow queries and failures.
8. Upgrade Prisma in controlled increments with regression checks.

## Preflight (Ask / Check First)
- Target database (PostgreSQL/MySQL/SQL Server/SQLite/Cockroach) and engine mode.
- Existing migration history quality and rollback expectations.
- Multi-tenant or sharding constraints.
- Data consistency requirements for writes and transaction scope.
- Security requirements for PII, secrets, and query logging.
- Release strategy for migration deployment windows.

## Operating Principles
- Keep Prisma schema as the single source of model intent.
- Enforce constraints in database, not only in application code.
- Treat migrations as reviewed, versioned change artifacts.
- Keep transactions short and explicitly scoped.
- Use query-level optimization before adding caching complexity.
- Keep Prisma Client and schema versions in lockstep.

## Schema Design
- Use explicit relation fields and relation names for clarity.
- Define indexes and unique constraints from real access patterns.
- Prefer explicit join models when relationship metadata is needed.
- Keep enum and default choices backward-compatible where possible.
- Document model ownership and cross-service boundaries.

### Schema Review Checklist
- Primary keys and unique constraints are explicit.
- Required vs optional fields match domain rules.
- Cascading behavior is deliberate (`onDelete`, `onUpdate`).
- Soft-delete strategy is consistent where applicable.
- Indexes support critical read paths.

## Migrations and Versioning
- Use `prisma migrate dev` for local iteration only.
- Use reviewed migration files for shared environments.
- Use `prisma migrate deploy` in CI/CD production paths.
- Test data backfills and long-running DDL separately.
- Keep rollback/forward-fix playbook documented.

### Migration Commands
```bash
npx prisma migrate dev
npx prisma migrate status
```

```bash
npx prisma migrate deploy
npx prisma generate
```

## Query Patterns and Performance
- Select only required fields (`select`, `include` with intent).
- Avoid N+1 by batching and relation-aware query design.
- Use pagination strategy appropriate to dataset size (cursor vs offset).
- Keep transaction boundaries narrow and purposeful.
- Inspect generated SQL and DB plans for expensive operations.

### Query Optimization Loop
1. Capture slow operation with context.
2. Reproduce with representative parameters.
3. Review SQL and DB execution plan.
4. Apply smallest change: index, query shape, batching.
5. Re-measure latency and resource impact.

## Transactions and Consistency
- Use interactive transactions only when needed.
- Keep retries idempotent for transient transaction failures.
- Validate concurrency assumptions under realistic load.
- Use optimistic concurrency where business flow permits.
- Avoid long-lived transactions in request paths.

## Data Integrity and Validation
- Validate business invariants at service boundary and database layer.
- Avoid trusting client-provided relation ownership.
- Keep seed scripts deterministic and environment-safe.
- Verify cascading behavior with explicit tests.
- Protect against accidental mass updates/deletes.

## Security and Privacy
- Do not log secrets or sensitive payload fields.
- Use parameterized Prisma API patterns; avoid raw SQL unless necessary.
- If raw SQL is required, keep parameters bound and reviewed.
- Apply row-level access controls in service logic and DB policy.
- Enforce least privilege for database credentials.

## Testing and CI/CD
- Test schema changes with migration apply + integration tests.
- Keep ephemeral test databases reproducible.
- Validate generated client in CI after schema changes.
- Add regression tests for bug fixes in query/mutation behavior.
- Gate deploys on migration and data-consistency checks.

### Typical Validation Commands
```bash
npx prisma validate
npx prisma format
npx prisma generate
```

```bash
npm run test
npm run test:integration
```

## Observability and Operations
- Capture query latency distribution by operation.
- Monitor connection pool saturation and timeout rates.
- Alert on migration failures and schema drift.
- Track error classes for Prisma runtime and DB exceptions.
- Keep incident runbooks for migration rollback and fail-forward.

## Upgrade and Maintenance Strategy
- Upgrade Prisma and engine components incrementally.
- Review release notes for breaking behavior changes.
- Validate generators/plugins compatibility before rollout.
- Run smoke tests in staging with production-like datasets.
- Maintain checklist for client regeneration and deployment ordering.

## Common Failure Modes
- Implicit relation assumptions creating ambiguous schema intent.
- Migration drift between local and production histories.
- N+1 query behavior hidden behind convenience APIs.
- Raw query usage without safe parameter handling.
- Long transactions causing lock contention under load.
- CI passing without real migration apply validation.

## Definition of Done
- Schema intent, constraints, and relations are explicit.
- Migration strategy is safe, reproducible, and reviewed.
- Critical queries are measured and optimized with evidence.
- Security and privacy controls are enforced.
- Testing and operational monitoring cover production risks.

## References
- `references/prisma.md`

## Reference Index
- `rg -n "Schema design|relations|model" references/prisma.md`
- `rg -n "Migrations|versioning|deploy" references/prisma.md`
- `rg -n "Query patterns|performance|scaling" references/prisma.md`
- `rg -n "Data integrity|validation|security" references/prisma.md`
- `rg -n "Testing|observability|upgrades|operations" references/prisma.md`

## Quick Questions (When Stuck)
- Is this a schema problem, migration problem, or query-shape problem?
- Can we enforce this invariant in the database directly?
- What is the safe deploy order for this schema + app change?
- Do we need a backfill, and can it run without locking risk?
- Which metric confirms this query optimization actually worked?
