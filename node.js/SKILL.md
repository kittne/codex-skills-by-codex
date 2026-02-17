---
name: node-js
description: >
  Build and operate modern Node.js applications with strong architecture, dependency hygiene,
  performance, resilience, observability, and security controls. Use when designing project layout,
  runtime/module strategy, testing and CI, release workflows, and production operations.
---

# Node.js

## Workflow
1. Confirm Node.js runtime baseline and package manager strategy.
2. Define repository structure and module boundaries.
3. Select language/module approach (TypeScript vs JavaScript, ESM vs CJS).
4. Establish dependency and supply-chain controls.
5. Implement testing strategy and CI quality gates.
6. Harden runtime behavior for performance and resilience.
7. Apply OWASP-aligned security controls.
8. Ship with observability and operations runbooks.

## Preflight (Ask / Check First)
- Target Node LTS version and hosting/runtime constraints.
- Service shape: API, worker, CLI, or hybrid.
- Repo model: single package vs monorepo.
- Build tooling and transpilation requirements.
- Expected traffic, latency SLOs, and failure budgets.
- Security/compliance requirements for dependencies and secrets.

## Operating Principles
- Keep boundaries explicit between domain, transport, and infrastructure.
- Prefer deterministic builds with lockfiles and pinned toolchains.
- Keep async flows cancellable and timeout-aware.
- Standardize logging and error contracts across services.
- Measure performance before optimization.
- Document operational decisions near code and pipelines.

## Project Structure and Architecture
- Organize by feature/domain, not utility dumping grounds.
- Keep shared libraries small and versioned intentionally.
- In monorepos, enforce dependency boundaries and ownership.
- Keep entrypoints (`api`, `worker`, `scheduler`) isolated.
- Separate adapters (DB, HTTP, queue) from domain logic.

### Example Layout
- `src/domain/*`
- `src/application/*`
- `src/infrastructure/*`
- `src/interfaces/http/*`
- `src/interfaces/queue/*`
- `test/*`

## Language and Module Strategy
- Prefer modern Node features supported by chosen LTS.
- Choose ESM or CJS deliberately; avoid accidental mixing.
- Prefer TypeScript strict mode for medium/large codebases.
- Keep API surfaces explicit and typed.
- Avoid transpilation complexity without clear benefit.

### Runtime Sanity Checks
```bash
node -v
npm -v
```

```bash
node --check src/index.js
```

## Dependency and Supply-Chain Management
- Commit lockfiles and enforce deterministic install mode in CI.
- Prefer minimal dependency count and regular update cadence.
- Audit direct and transitive dependencies by exploitability.
- Keep private package publishing scoped and credential-protected.
- Document policy for semver ranges and upgrade windows.

## Testing and Release Workflow
- Maintain layered tests: unit, integration, and system/e2e.
- Keep CI fast for PR feedback; run heavy suites on merge/nightly.
- Enforce lint/type/test/build as required gates.
- Use semantic release/tagging conventions consistently.
- Keep changelog and rollback notes for production deploys.

### Typical Validation Commands
```bash
npm run lint
npm run test
npm run build
```

```bash
npm run test:integration
npm audit --omit=dev
```

## Performance and Scalability
- Use async I/O and avoid blocking CPU in request paths.
- Offload CPU-heavy work to workers or external jobs.
- Profile event loop delay and memory under realistic load.
- Apply connection pooling and bounded concurrency.
- Implement backpressure and queue limits for burst traffic.

## Error Handling and Resilience
- Use typed/domain errors with stable mapping to HTTP/queue responses.
- Apply timeouts/retries with jitter and idempotency controls.
- Guard against cascading failure with circuit-breaker patterns.
- Ensure graceful shutdown drains in-flight work.
- Keep fatal vs recoverable error policy explicit.

## Security Controls
- Validate and sanitize all untrusted inputs.
- Keep secrets out of source and logs.
- Enforce authn/authz at boundary and domain levels.
- Apply secure headers and TLS posture for HTTP services.
- Protect dependency chain with scanning and provenance where possible.

## Observability and Operations
- Emit structured logs with correlation IDs.
- Capture metrics for latency, throughput, errors, and saturation.
- Instrument distributed tracing on critical request paths.
- Keep dashboards and SLO alerts tied to user-facing risk.
- Maintain incident runbooks and service ownership maps.

## Deployment and Runtime Operations
- Use immutable artifacts and environment-specific config injection.
- Keep health endpoints meaningful (`liveness`, `readiness`).
- Validate startup probes and shutdown grace windows.
- Track config drift across environments.
- Exercise disaster-recovery drills and failover playbooks.

## Data and Integration Boundaries
- Set explicit DB/query timeouts and pool limits.
- Keep migrations transactional and reversible when possible.
- Make external API retry behavior explicit per endpoint.
- Use idempotency keys where duplicate delivery is possible.

## Common Failure Modes
- Unbounded async concurrency leading to resource exhaustion.
- Event-loop blocking work in hot request paths.
- Mixed module systems causing runtime loader errors.
- CI passing with missing integration coverage.
- Dependency drift causing non-reproducible builds.
- Weak shutdown behavior dropping in-flight work.

## Definition of Done
- Architecture and boundaries are clear and enforced.
- Build/test/release pipeline is deterministic.
- Performance and resilience have measured baselines.
- Security controls and dependency hygiene are active.
- Observability and operational runbooks are production-ready.

## References
- `references/node.js.md`

## Reference Index
- `rg -n "Project structure|architecture|monorepo" references/node.js.md`
- `rg -n "ESM|CommonJS|TypeScript|coding style" references/node.js.md`
- `rg -n "Dependency|supply-chain|npm|Yarn|pnpm" references/node.js.md`
- `rg -n "Testing|CI|release|changelog" references/node.js.md`
- `rg -n "Performance|resilience|error handling" references/node.js.md`
- `rg -n "OWASP|security|observability|deployment" references/node.js.md`

## Quick Questions (When Stuck)
- Is this a runtime bottleneck, architecture issue, or dependency issue?
- Can we reduce complexity by removing a dependency?
- Is this path bounded by CPU, I/O, or external service latency?
- What happens during shutdown under active load?
- Which metric proves this change improved reliability?
