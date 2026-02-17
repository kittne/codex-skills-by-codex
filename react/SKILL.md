---
name: react
description: >
  Build and review React applications with modern best practices for component design, state
  management, effects and data fetching, performance, testing, accessibility, architecture, and
  delivery quality gates. Use for feature work, refactors, and production hardening in React apps.
---

# React

## Workflow
1. Confirm React/runtime baseline and application architecture constraints.
2. Design component boundaries and prop APIs before implementation.
3. Choose state strategy by scope: local, shared client, and server state.
4. Implement effects and async flows with cancellation/error handling.
5. Optimize rendering and bundle behavior with measured evidence.
6. Validate user behavior with testing pyramid and accessibility checks.
7. Enforce lint/type/build gates in CI.
8. Document migration and operational risks for release.

## Preflight (Ask / Check First)
- React version and rendering mode constraints.
- TypeScript strictness and linting baseline.
- Data layer choice for server state and caching.
- Design system/component library conventions.
- Performance goals for key routes and devices.
- Accessibility and compliance requirements.

## Operating Principles
- Keep components pure and deterministic by default.
- Prefer composition and hooks over inheritance patterns.
- Keep state minimal and colocated to where it is used.
- Separate server-state concerns from local UI state.
- Treat effects as synchronization boundaries, not general logic buckets.
- Optimize after measuring real bottlenecks.

## Component Design and TypeScript
- Use function components and typed props by default.
- Prefer discriminated unions for mode-dependent props.
- Avoid prop surfaces with uncontrolled boolean explosion.
- Keep component responsibilities narrow and explicit.
- Use stable keys for list rendering; avoid index keys on mutable lists.

### Component API Checklist
- Props represent intent, not implementation detail.
- Controlled/uncontrolled behavior is explicit.
- Accessibility attributes and semantics are first-class.
- Error and loading states are modeled clearly.

## State Management
- Start with local state; lift only when necessary.
- Use context for ambient dependencies, not broad mutable state by default.
- Use reducer patterns when transitions are complex.
- Keep derived state computed, not duplicated.
- Escalate to dedicated state tools only when complexity demands.

### State Decision Heuristic
- Local UI concern: `useState`/`useReducer`.
- Cross-feature client concern: context or store.
- Remote data: dedicated server-state cache/query layer.

## Effects, Data Fetching, and Error Handling
- Keep effects idempotent and dependency-accurate.
- Always clean up subscriptions, timers, and abortable async work.
- Handle loading, empty, error, and retry paths explicitly.
- Keep side effects in handlers/effects, not render paths.
- Normalize error boundaries for recoverable UX.

### Effect Safety Pattern
```tsx
useEffect(() => {
  const controller = new AbortController();
  loadData({ signal: controller.signal });
  return () => controller.abort();
}, [loadData]);
```

## Performance and Scalability
- Profile before adding memoization.
- Use `memo`, `useMemo`, and `useCallback` only where measured benefit exists.
- Avoid creating unstable objects/functions in hot paths unnecessarily.
- Split large bundles and defer non-critical code.
- Keep list rendering virtualized for large datasets.

### Performance Validation Commands
```bash
npm run build
npm run test
```

```bash
npm run lint
npm run typecheck
```

## Testing Strategy
- Unit-test pure logic and component edge cases.
- Integration-test feature flows and data boundaries.
- E2E-test critical user journeys and regressions.
- Prefer user-centric assertions over implementation details.
- Keep flaky tests out of required merge gates.

## Accessibility, Styling, and UX Consistency
- Use semantic HTML and roles before custom ARIA.
- Ensure visible focus states and keyboard operability.
- Validate color contrast and reduced-motion behavior.
- Keep form labels and error messaging accessible.
- Align styling patterns with design system tokens.

## Architecture and Code Organization
- Use feature-oriented folders and clear ownership boundaries.
- Keep business logic in hooks/services, not deeply nested UI trees.
- Avoid circular dependencies and utility sprawl.
- Keep route-level composition simple and discoverable.
- Document domain boundaries and extension points.

## Delivery, Security, and Operations
- Keep CI gates for lint, type checks, tests, and build.
- Prevent secrets from entering frontend bundles.
- Validate dependency risk and lockfile integrity.
- Use runtime monitoring for errors and performance web vitals.
- Keep rollback strategy for high-risk releases.

## Migration and Modernization
- Upgrade React and tooling incrementally.
- Validate strict mode behavior and deprecated API usage.
- Remove legacy patterns gradually with codemod/testing support.
- Re-baseline performance and bundle budgets after upgrades.

## Common Failure Modes
- Effects with stale dependencies causing subtle bugs.
- Over-centralized state creating coupling and re-render storms.
- Premature memoization increasing complexity with no gain.
- Tests asserting internals instead of user-visible behavior.
- Accessibility regressions from custom components without semantics.
- Feature delivery without route-level performance verification.

## Definition of Done
- Component and state boundaries are explicit and maintainable.
- Async/effect logic is safe and cancellation-aware.
- Performance, accessibility, and testing checks are passing.
- Delivery pipeline enforces quality and security gates.
- Upgrade and rollback implications are documented.

## References
- `references/react.md`

## Reference Index
- `rg -n "Component design|TypeScript" references/react.md`
- `rg -n "State management|local|shared|server state" references/react.md`
- `rg -n "Side effects|data fetching|error handling" references/react.md`
- `rg -n "Performance|scalability" references/react.md`
- `rg -n "Testing strategy|accessibility|styling|architecture" references/react.md`
- `rg -n "Delivery|security|migration" references/react.md`

## Quick Questions (When Stuck)
- Is this state colocated at the lowest useful scope?
- Should this logic be a custom hook or remain local?
- Can we remove this effect and derive from state instead?
- Which user-facing metric justifies this optimization?
- Do tests and accessibility checks prove this behavior is safe?
