---
name: nestjs
description: >
  Build and review production-grade NestJS backend systems. Use when designing module architecture,
  request lifecycle concerns, validation, configuration/logging, authentication/authorization,
  persistence patterns, performance/scalability, deployment strategy, and testing/observability.
---

# NestJS

## Workflow
1. Confirm runtime baseline (Node version, Nest version, transport style, deployment model).
2. Define module boundaries and domain-driven structure.
3. Implement request pipeline with validation and error discipline.
4. Establish config management, logging, and environment strategy.
5. Apply authentication, authorization, and HTTP security controls.
6. Choose persistence patterns and transactional boundaries.
7. Optimize performance and scalability with measured evidence.
8. Finalize testing, CI/CD, observability, and migration runbooks.

## Preflight (Ask / Check First)
- NestJS version, adapter (Express or Fastify), and target throughput.
- API style (REST, GraphQL, events, hybrid) and compatibility constraints.
- Security model (JWT, sessions, OAuth2/OIDC) and token lifecycle requirements.
- Database and ORM choice (Prisma, TypeORM, Sequelize, custom).
- Deployment target (containers, serverless, VM/Kubernetes).
- SLOs for latency, availability, and error budgets.

## Operating Principles
- Keep modules cohesive and avoid cross-module coupling.
- Treat DTO validation as mandatory input boundary defense.
- Use guards/interceptors/pipes deliberately by concern.
- Prefer structured logs with request correlation IDs.
- Keep configuration typed, validated, and environment-aware.
- Design for graceful shutdown and health-aware orchestration.

## Architecture and Project Structure
- Organize by feature/domain, not by technical layer only.
- Keep controllers thin; business logic belongs in services/use cases.
- Isolate infrastructure dependencies behind interfaces/providers.
- Use shared modules carefully to avoid "god" dependencies.
- Keep public contracts stable and versioned intentionally.

### Suggested Layout
- `src/modules/*` for domain features.
- `src/common/*` for cross-cutting utilities.
- `src/config/*` for validated configuration.
- `src/infrastructure/*` for external adapters.
- `test/*` for integration and e2e harness.

## Request Lifecycle and Cross-Cutting Concerns
- Validate all external inputs with DTOs and pipes.
- Use exception filters for normalized API error responses.
- Use interceptors for logging, serialization, and timing.
- Apply guards for authn/authz; keep policy explicit.
- Keep middleware focused on transport-level concerns only.
- On Nest 11+, remember middleware from global modules runs before imported-module middleware.

### Validation Snippet
```ts
export class CreateUserDto {
  @IsEmail()
  email!: string;

  @IsString()
  @MinLength(8)
  password!: string;
}
```

## API Design and Contracts
- Version APIs intentionally (URI/header strategy) before growth.
- Keep OpenAPI docs generated from source annotations.
- Separate read/write models where complexity warrants.
- For GraphQL, protect resolver N+1 patterns and complexity.
- Make error codes stable and machine-consumable.

## Configuration, Logging, and Errors
- Use centralized config module with schema validation.
- Keep secrets in environment/secret manager, never hardcoded.
- Use structured logs (JSON) with request and trace identifiers.
- Normalize exception mapping to stable API responses.
- Avoid leaking internals in error payloads.

### Runtime Checks
```bash
npm run start:dev
npm run lint
npm run test
```

```bash
npm run test:e2e
npm run build
```

## Security and Authentication
- Apply secure headers, CORS policy, and body size limits.
- Use appropriate auth strategy per client type and risk profile.
- Keep token issuance, refresh, revocation strategy explicit.
- Enforce RBAC/ABAC in guards and domain services.
- Rate limit sensitive routes and monitor abuse patterns.

### Auth Strategy Guidance
- JWT for stateless APIs with careful rotation policy.
- Sessions for web-first apps needing server-side control.
- OAuth2/OIDC for delegated identity and enterprise SSO.

## Persistence and Data Access
- Keep repositories/services transactional where consistency matters.
- Use migrations as code with reviewed rollback path.
- Avoid leaking ORM types across module boundaries.
- Manage connection pools and query timeouts explicitly.
- Track slow queries and optimize with plan-level evidence.

## Performance and Scalability
- Choose Fastify when throughput and low overhead are priorities.
- Use caching for expensive read paths with invalidation strategy.
- Offload long-running work to queues/background workers.
- Enable compression and response shaping where appropriate.
- Scale horizontally with health checks and stateless services.

## Microservices and Real-Time Patterns
- Use message contracts with explicit versioning.
- Keep idempotency in event handlers and retries.
- For WebSockets, enforce auth and backpressure limits.
- Separate transport concerns from core business logic.

## Testing, CI/CD, and Observability
- Maintain balanced coverage across unit, integration, and e2e.
- Mock external systems at boundaries, not everywhere.
- Keep CI pipeline deterministic with required quality gates.
- Instrument metrics, traces, and logs with consistent labels.
- Define deployment SLOs and rollback triggers.

### Testing Baseline
```bash
npm run test
npm run test:cov
npm run test:e2e
```

## Migration and Upgrade Strategy
- Upgrade Node and Nest in staged, compatible increments.
- Verify library compatibility before framework upgrades.
- For Nest 11+, migrate `CacheModule` Redis setups to `@keyv/redis`-based stores.
- For Nest 11 with Fastify adapter, validate Fastify v5 compatibility and route behavior in e2e tests.
- Run canary deployment for risky runtime changes.
- Maintain migration docs for module or transport refactors.

## Common Failure Modes
- Over-coupled modules causing fragile refactors.
- Missing DTO validation leading to runtime faults/security gaps.
- Inconsistent error mapping across controllers.
- Auth checks in controllers only, bypassing service boundaries.
- Unbounded queue/retry behavior causing cascading failures.
- CI tests passing but no production-like integration validation.

## Definition of Done
- Architecture boundaries and request pipeline are explicit.
- Validation, auth, and logging are consistently implemented.
- Database and performance behavior are measured and acceptable.
- CI/CD and observability pipelines are production-ready.
- Upgrade and rollback runbooks are current.

## References
- `references/nestjs.md`

## Reference Index
- `rg -n "Architecture and project structure|request lifecycle" references/nestjs.md`
- `rg -n "DTO|validation|pipes|guards|interceptors" references/nestjs.md`
- `rg -n "Configuration|logging|error" references/nestjs.md`
- `rg -n "Security|authentication|authorization" references/nestjs.md`
- `rg -n "Persistence|ORM|database" references/nestjs.md`
- `rg -n "Performance|scalability|microservices|WebSockets" references/nestjs.md`
- `rg -n "Testing|CI/CD|observability|migration" references/nestjs.md`

## Quick Questions (When Stuck)
- Is this concern in the right layer (controller/service/module)?
- Do DTO and guard boundaries fully protect this endpoint?
- Can this performance issue be proven with tracing/metrics?
- Should this be async background work instead of request-path work?
- What rollback step exists if this deployment fails under load?
