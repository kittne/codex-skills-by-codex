---
name: fastify
description: >
  Build, review, and operate production Fastify services with strong plugin architecture,
  schema-first validation/serialization, secure request handling, and observable operations.
  Use for API design, performance tuning, lifecycle hooks, deployment hardening, and incident debugging.
---

# Fastify

## Workflow
1. Confirm service SLOs, trust boundaries, and integration contracts.
2. Design plugin/module boundaries and encapsulation strategy.
3. Define request/response schemas and error contracts.
4. Wire lifecycle hooks for auth, validation, and observability.
5. Harden runtime limits, security headers, and backpressure behavior.
6. Validate performance and failure handling under load.
7. Ship with operational dashboards and alert coverage.

## Preflight (Ask / Check First)
- Node.js and Fastify versions.
- Deployment runtime (container, serverless, VM).
- Authn/authz model and session/token strategy.
- Latency and throughput targets.
- Existing incidents (timeouts, memory pressure, error spikes).

## Architecture and Encapsulation
- Use plugins to isolate domains and dependencies.
- Keep handlers thin; move business logic to services.
- Register shared infrastructure through scoped decorators.
- Avoid cross-plugin state leakage.
- Use `onClose` hooks for deterministic resource cleanup.

## Schema-First Contracts
- Define JSON schema for all externally reachable routes.
- Use shared schema IDs for reusable components.
- Keep response schemas explicit to protect serialization boundaries.
- Treat schema drift as an API compatibility event.
- Version contracts intentionally.

### Route Example
```js
fastify.post('/items', {
  schema: {
    body: { type: 'object', required: ['name'], properties: { name: { type: 'string' } } },
    response: { 201: { type: 'object', properties: { id: { type: 'string' } } } }
  }
}, async (req, reply) => {
  const id = await createItem(req.body)
  return reply.code(201).send({ id })
})
```

## Lifecycle Hooks and Cross-Cutting Behavior
- Use `onRequest` for request correlation and coarse checks.
- Use `preValidation`/`preHandler` for auth and policy checks.
- Use `onError` and custom handlers for consistent error envelopes.
- Keep hook behavior deterministic and low-latency.
- Avoid heavy I/O in hot-path hooks.

## Performance and Resilience
- Keep async paths non-blocking; isolate CPU-heavy work.
- Use sensible body size limits and timeout policies.
- Profile serialization and validation hotspots.
- Apply connection and concurrency controls to protect dependencies.
- Stress test backpressure and queueing behavior.

## Security and Operations
- Enforce strict input validation and response shaping.
- Restrict CORS and trusted proxy settings intentionally.
- Apply rate limiting and abuse controls where needed.
- Emit structured logs with request IDs.
- Instrument latency, error rate, and saturation metrics.

## Validation Commands
```bash
pnpm test
pnpm exec eslint .
pnpm exec node --test
pnpm exec autocannon -c 50 -d 20 http://localhost:3000/health
```

## Common Failure Modes
- Bypassing schemas and leaking internal fields.
- Plugin decorators colliding across encapsulation boundaries.
- Blocking calls in async handlers.
- Inconsistent error shapes across modules.
- Missing shutdown cleanup for DB/cache clients.

## Definition of Done
- Plugin boundaries are explicit and testable.
- Schemas cover all route inputs and outputs.
- Error handling and logging are standardized.
- Performance and resilience checks pass target thresholds.
- Security controls and operational telemetry are in place.

## References
- `references/fastify-2026-02-18.md`

## Reference Index
- `rg -n "plugin|encapsulation|decorate" references/fastify-2026-02-18.md`
- `rg -n "schema|validation|serialization" references/fastify-2026-02-18.md`
- `rg -n "hooks|error handler|lifecycle" references/fastify-2026-02-18.md`
- `rg -n "performance|operations|security" references/fastify-2026-02-18.md`
