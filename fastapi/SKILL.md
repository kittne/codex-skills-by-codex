---
name: fastapi
description: >
  Build, review, and operate production FastAPI services with strong API design, dependency and
  lifespan management, typed request/response contracts, concurrency-safe I/O, security hardening,
  observability, and deployment reliability. Use for architecture decisions, endpoint implementation,
  refactors, incident debugging, performance tuning, and CI/CD readiness of FastAPI APIs.
---

# FastAPI

## Workflow
1. Confirm product requirements, SLOs, and API contract expectations.
2. Define module boundaries: routers, domain services, and infrastructure adapters.
3. Establish lifespan-managed shared resources and request-scoped dependencies.
4. Enforce Pydantic request/response models as external contracts.
5. Choose correct async or sync execution paths for each endpoint.
6. Implement security, observability, and failure handling standards.
7. Validate with tests, load checks, and deployment-safe rollout controls.

## Preflight (Ask / Check First)
- Check Python and FastAPI versions and runtime constraints.
- Check persistence stack and transaction requirements.
- Check whether endpoints are latency-sensitive, throughput-sensitive, or both.
- Check authn/authz model and compliance obligations.
- Check deployment platform, autoscaling behavior, and release cadence.

## Architecture and Project Layout
- Keep routers thin and place business logic in services.
- Separate domain logic from framework-specific transport concerns.
- Group API versions explicitly to preserve backward compatibility.
- Keep infra adapters isolated for DB, cache, queues, and external clients.
- Avoid circular dependencies by enforcing clear layer boundaries.

Suggested layout:
```text
app/
  main.py
  api/v1/endpoints/
  api/deps.py
  core/config.py
  domain/services/
  infra/db/
  infra/cache/
  schemas/
  tests/
```

## Lifespan and Dependency Management
- Initialize shared resources in application lifespan handlers.
- Dispose shared resources deterministically during shutdown.
- Use dependency `yield` patterns for request-scoped cleanup.
- Keep dependency functions small and composable.
- Avoid hidden I/O side effects in dependency graphs.

Example pattern:
```python
@asynccontextmanager
async def lifespan(app: FastAPI):
    app.state.http = httpx.AsyncClient(timeout=5.0)
    try:
        yield
    finally:
        await app.state.http.aclose()
```

## Data Contracts and Serialization
- Define separate input and output schemas.
- Use response models consistently to prevent accidental field leakage.
- Enable strict validation and explicit field constraints.
- Keep internal ORM models out of API boundary payloads.
- Version schema changes intentionally and document deprecations.

## Async, Sync, and Background Execution
- Use `async def` only when downstream stack is non-blocking.
- Keep blocking libraries in sync endpoints or threadpool paths.
- Move long-running work to queue workers, not request lifecycle.
- Treat in-process `BackgroundTasks` as short, best-effort operations.
- Apply timeouts and cancellation-aware patterns for outbound I/O.

## Error Handling and API Reliability
- Standardize error envelopes across endpoints.
- Map domain exceptions to predictable HTTP status codes.
- Avoid returning raw stack traces in client responses.
- Add idempotency semantics where clients may retry writes.
- Keep retry policies bounded and observable.

## Security Baseline
- Enforce authentication and authorization at explicit boundaries.
- Validate all external inputs even after schema parsing.
- Configure CORS minimally; avoid wildcard credentials.
- Apply trusted-host and security-header middleware where appropriate.
- Add rate limiting and abuse controls for high-risk endpoints.

## Persistence and Transactions
- Keep transaction boundaries in service layer, not router layer.
- Use one session per request unless workflow requires explicit units of work.
- Avoid lazy-loading surprises in response serialization paths.
- Keep migration tooling and schema lifecycle in CI.
- Validate connection pool sizing against worker and DB limits.

## Observability and Operations
- Emit structured logs with request correlation identifiers.
- Capture metrics for latency, error rate, saturation, and queue depth.
- Instrument tracing across inbound requests and outbound dependencies.
- Separate business metrics from infrastructure metrics.
- Keep dashboards aligned to SLOs and alert thresholds.

## Testing Strategy
- Write unit tests for domain logic without HTTP transport.
- Write integration tests for endpoint contracts and dependency wiring.
- Snapshot or diff OpenAPI changes for contract stability.
- Add regression tests for every bug fix in production paths.
- Keep test fixtures deterministic and isolated from shared state.

Suggested validation commands:
```bash
pytest -q
pytest tests/integration -q
uvicorn app.main:app --reload
```

## Deployment and Runtime Practices
- Run multiple workers only after validating shared-state assumptions.
- Keep health and readiness checks explicit.
- Use graceful shutdown timeouts aligned to in-flight request profiles.
- Separate deploy and migrate responsibilities to reduce blast radius.
- Use staged rollouts and automatic rollback criteria.

## CI/CD and Governance
- Gate merges on tests, type checks, linting, and API contract checks.
- Keep environment configuration explicit and non-secret by default.
- Inject secrets via platform secret stores, not code or logs.
- Keep release artifacts traceable to source and build inputs.
- Enforce review ownership for API and schema changes.

## Common Failure Modes
- Mixing business logic into dependency functions and routers.
- Using `async def` with blocking DB or HTTP calls.
- Returning ORM entities directly and leaking private fields.
- Treating background tasks as durable job infrastructure.
- Shipping without standardized error and observability contracts.

## Definition of Done
- Keep architecture boundaries explicit and testable.
- Keep lifespans and dependencies deterministic and leak-free.
- Keep API contracts typed, versioned, and regression-tested.
- Keep async/sync execution choices intentional and measurable.
- Keep security, observability, and rollout controls production-ready.

## References
- `references/fastapi-2026-02-17.md`

## Reference Index
- `rg -n "architecture|router|versioning|project structure" references/fastapi-2026-02-17.md`
- `rg -n "lifespan|dependency|yield|startup|shutdown" references/fastapi-2026-02-17.md`
- `rg -n "Pydantic|response model|serialization|validation" references/fastapi-2026-02-17.md`
- `rg -n "async|sync|BackgroundTasks|queue|concurrency" references/fastapi-2026-02-17.md`
- `rg -n "error|logging|metrics|tracing|observability" references/fastapi-2026-02-17.md`
- `rg -n "security|CORS|trusted hosts|rate limiting" references/fastapi-2026-02-17.md`
- `rg -n "testing|deployment|performance|CI/CD|anti-pattern" references/fastapi-2026-02-17.md`
