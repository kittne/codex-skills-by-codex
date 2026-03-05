---
name: turborepo
description: >
  Build and operate Turborepo monorepos with deterministic task graphs, cache correctness,
  and CI scalability. Use for `turbo.json` design, task dependency modeling, outputs/inputs hashing,
  environment variable handling, remote cache rollout, and pipeline troubleshooting.
---

# Turborepo

## Workflow
1. Confirm monorepo package boundaries and critical tasks.
2. Define task graph dependencies (`dependsOn`) by data flow, not habit.
3. Set precise task inputs/outputs for cache correctness.
4. Configure environment hashing and passthrough values intentionally.
5. Enable and validate local and remote cache behavior.
6. Harden CI execution strategy and change-based targeting.
7. Monitor cache hit rate and task critical-path latency.

## Preflight (Ask / Check First)
- Turborepo version and package manager.
- Build tools in use (Next.js, Vite, Jest, etc.).
- CI provider and artifact/cache controls.
- Environment variables affecting build output.
- Existing cache miss or stale artifact incidents.

## Task Graph Design
- Model dependencies explicitly with `^task` where upstream outputs matter.
- Keep long-running dev tasks uncached.
- Split large tasks into build/test/lint/typecheck for better reuse.
- Avoid implicit cross-package side effects.
- Keep task names consistent across packages.

### Example `turbo.json`
```json
{
  "tasks": {
    "build": { "dependsOn": ["^build"], "outputs": ["dist/**", ".next/**", "!.next/cache/**"] },
    "test": { "dependsOn": ["build"], "outputs": ["coverage/**"] },
    "lint": {},
    "dev": { "cache": false }
  }
}
```

## Cache Correctness Rules
- Declare outputs narrowly so stale files do not masquerade as hits.
- Add non-default inputs when config files drive results.
- Include environment variables that affect task output in `env`.
- Keep secrets out of cache keys unless required for correctness.
- Validate cache correctness before enabling remote cache broadly.

## Environment and Global Dependencies
- Use `globalDependencies` for root configs that impact many tasks.
- Use `globalEnv` only for variables with broad task impact.
- Use task `inputs` for `.env` files; do not rely on removed `dotEnv`/`globalDotEnv` keys.
- Keep per-task env lists minimal and explicit.
- Avoid broad wildcard env strategies that over-invalidate cache.

## CI Strategy
- Use change-aware filters to limit execution scope.
- Fail fast on cache/config mismatch signals.
- Export remote cache credentials securely via CI secrets.
- Ensure cold-start path remains acceptable when cache is unavailable.
- Record task timings and hit rates for regression analysis.
- For Turborepo 2.x+, use `tasks` instead of `pipeline`, expect cache at `.turbo/cache`, and replace `outputMode` with `outputLogs` during upgrades.

### Typical Commands
```bash
pnpm turbo run build test lint
pnpm turbo run build --filter=...[origin/main]
pnpm turbo run test --filter=./packages/api...
pnpm turbo run build --summarize
```

## Security and Operations
- Treat remote cache as shared infrastructure with access controls.
- Scope cache tokens to minimal required permissions.
- Rotate cache credentials and audit usage patterns.
- Keep rollback path for cache incidents documented.

## Validation Commands
```bash
pnpm turbo run build
pnpm turbo run test
pnpm turbo run lint
pnpm turbo run build --filter=...[origin/main]
```

## Common Failure Modes
- Missing outputs causing false cache hits.
- Overly broad env hashing causing cache thrash.
- Hidden dependency edges not captured in `dependsOn`.
- Caching non-deterministic tasks.
- CI using different toolchain versions than local.

## Definition of Done
- Task graph reflects real dependency flow.
- Cache hits are correct and reproducible.
- CI and local runs use aligned toolchains.
- Remote cache is secure, observable, and recoverable.
- Critical-path timing and hit-rate baselines are documented.

## References
- `references/turborepo-2026-02-18.md`

## Reference Index
- `rg -n "tasks|dependsOn|outputs" references/turborepo-2026-02-18.md`
- `rg -n "inputs|env|globalDependencies" references/turborepo-2026-02-18.md`
- `rg -n "remote cache|CI|filter" references/turborepo-2026-02-18.md`
- `rg -n "failure modes|debug" references/turborepo-2026-02-18.md`
