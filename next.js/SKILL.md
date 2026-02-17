---
name: next-js
description: >
  Build and operate modern Next.js applications with strong routing, data fetching, caching,
  performance, SEO/accessibility, security, and deployment practices. Use when working on App Router
  or Pages Router architecture, rendering strategy, testing, CI/CD, and hosting trade-offs.
---

# Next.js

## Workflow
1. Confirm Next.js version, router mode, and deployment target.
2. Define routing architecture and data boundaries before implementation.
3. Choose rendering and caching strategy per route.
4. Implement mutation and state patterns with minimal complexity.
5. Optimize performance budgets and asset delivery.
6. Validate SEO, accessibility, and security posture.
7. Ship with reproducible CI/CD and observability.
8. Document migration steps for router/runtime/tooling changes.

## Preflight (Ask / Check First)
- App Router, Pages Router, or mixed mode in current codebase.
- Hosting platform constraints and edge/runtime requirements.
- Server-side data sources and caching expectations.
- Authentication/session strategy and sensitive-data boundaries.
- Performance goals (LCP, INP, TTFB) and SEO priorities.
- Monorepo tooling and package manager standards.

## Operating Principles
- Keep server/client boundaries explicit.
- Fetch close to the server boundary by default.
- Choose the simplest state model that satisfies UX needs.
- Prefer route-level caching decisions over global guesses.
- Make security headers and secret handling explicit.
- Enforce quality gates in CI before deployment.

## Architecture and Routing Design
- Prefer App Router for new development unless constraints require Pages Router.
- Keep route segments aligned with ownership and domain boundaries.
- Isolate shared layouts from page-specific data behavior.
- Keep API routes and server actions scoped by domain.
- Document migration boundaries if both routers coexist.

### Routing Decision Notes
- App Router: modern conventions, server components, nested layouts.
- Pages Router: legacy compatibility and established workflows.
- Hybrid: transitional approach with strict ownership to avoid confusion.

## Data Fetching, Mutations, and State
- Pick route-specific fetching strategy based on freshness requirements.
- Keep server-originated data on the server when possible.
- Use client state only for interactive UI concerns.
- Separate mutation side-effects from view rendering concerns.
- Use invalidation/revalidation patterns that match user expectations.

### Data Strategy Checklist
- Define source of truth for each route.
- Document cache behavior and revalidation path.
- Verify error/loading states for all asynchronous branches.
- Ensure mutation paths include optimistic/rollback behavior if needed.

## Caching and Rendering Performance
- Choose rendering mode per route: static, dynamic, or mixed.
- Use caching primitives intentionally with clear invalidation rules.
- Audit large dependencies and avoid accidental client bundle growth.
- Keep image optimization and responsive sizing configured.
- Track route-level web vitals and regressions.

### Performance Commands
```bash
npm run build
npm run start
```

```bash
npx next telemetry disable
npx next info
```

## SEO and Accessibility
- Keep metadata complete and route-appropriate.
- Enforce semantic heading hierarchy and landmark structure.
- Ensure canonical URLs and indexing policy are explicit.
- Validate structured data where business value exists.
- Test keyboard navigation and focus management for key flows.

## Security and Operations
- Configure security headers and CSP where feasible.
- Keep secrets server-side; never expose private keys in client bundles.
- Validate auth/session behavior across rendering boundaries.
- Apply strict input validation on API and action boundaries.
- Document incident response and rollback mechanics.

## Testing Strategy
- Unit test components and utilities with focused scope.
- Add integration tests for route behavior and data boundaries.
- Add e2e checks for critical user journeys.
- Keep visual/regression checks where UI risk is high.
- Gate merges on deterministic test and lint checks.

### Typical Validation Commands
```bash
npm run lint
npm run test
npm run build
```

```bash
npm run test:e2e
```

## CI/CD and Deployment
- Keep builds reproducible and lockfile-driven.
- Promote artifacts through environments rather than rebuilding each stage.
- Use preview deployments for review and QA signoff.
- Configure deployment guards for migrations and risky toggles.
- Define rollback triggers and execution owners.

## Hosting and Runtime Trade-offs
- Compare platform strengths for caching, edge runtime, and observability.
- Validate cold-start and regional latency against user distribution.
- Keep platform-specific behavior documented near deployment configs.
- Avoid coupling core app logic to one provider without abstraction.

## Monorepo and TypeScript Hardening
- Keep package boundaries strict and dependency graph visible.
- Use TypeScript strict mode and shared tsconfig conventions.
- Prevent accidental server-only imports in client components.
- Validate build and test behavior from repo root and app workspace.

## Migration and Upgrade Strategy
- Upgrade Next.js in controlled increments.
- Re-test routing, caching, and middleware behavior after upgrades.
- Verify plugin/tool compatibility before production rollout.
- Keep migration notes for breaking changes and fallback options.

## Common Failure Modes
- Mixed router usage without clear ownership.
- Cache behavior surprises due to undocumented defaults.
- Data fetching in wrong layer causing duplicated requests.
- Large client bundles from server-only dependency leakage.
- SEO regressions from missing metadata on dynamic routes.
- CI success with missing runtime observability coverage.

## Definition of Done
- Routing and data architecture are explicit and documented.
- Rendering and caching behavior are validated against product expectations.
- Performance, SEO, and accessibility checks are passing.
- Security headers and secret boundaries are enforced.
- CI/CD, deploy, and rollback paths are production-ready.

## References
- `references/next.js.md`

## Reference Index
- `rg -n "Architecture and routing|Router selection" references/next.js.md`
- `rg -n "data fetching|mutations|state management" references/next.js.md`
- `rg -n "caching|rendering|performance|Image" references/next.js.md`
- `rg -n "SEO|accessibility" references/next.js.md`
- `rg -n "Security|Testing|CI/CD|Observability" references/next.js.md`
- `rg -n "Deployment|hosting|monorepo|TypeScript|migration" references/next.js.md`

## Quick Questions (When Stuck)
- Does this route need freshness, speed, or strict consistency first?
- Should this data live server-side or client-side?
- Is this cache rule explicit and testable?
- What would break first under load: compute, I/O, or cache invalidation?
- Is rollback possible without data inconsistency?
