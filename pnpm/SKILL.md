---
name: pnpm
description: >
  Design, maintain, and operate pnpm workspaces for reproducible Node.js development and CI.
  Use for workspace layout, dependency policies, filtered task execution, lockfile integrity,
  monorepo installs, and release pipeline hardening.
---

# pnpm

## Workflow
1. Confirm workspace topology and ownership boundaries.
2. Define `pnpm-workspace.yaml` package globs and exclusions.
3. Standardize dependency policies and workspace protocol usage.
4. Harden install and lockfile behavior for CI reproducibility.
5. Optimize filtered script execution for local and CI speed.
6. Validate supply-chain and upgrade hygiene.
7. Verify release and publish paths.

## Preflight (Ask / Check First)
- pnpm version and Node version policy.
- Workspace package count and dependency graph shape.
- CI platform and cache strategy.
- Private registry/auth requirements.
- Current lockfile drift or broken installs.

## Workspace Structure
- Keep package globs explicit and avoid accidental inclusion.
- Exclude examples/tests from production build graphs when appropriate.
- Keep `sharedWorkspaceLockfile` enabled unless there is a documented isolation need.
- Use `workspace:` protocol for internal dependencies to prevent accidental registry drift.
- Keep root scripts thin; delegate to package-level scripts.
- Avoid duplicated toolchains across packages unless isolation is required.

### Example Workspace File
```yaml
packages:
  - "apps/*"
  - "packages/*"
  - "tools/*"
  - "!**/examples/**"
```

## Dependency and Version Policy
- Prefer exact internal links through `workspace:*`.
- Control transitive overrides intentionally.
- Keep peer dependency expectations explicit in shared packages.
- Use dedupe and outdated checks on a regular cadence.
- Gate upgrades with test and typecheck baselines.

## Install and Lockfile Discipline
- Use `pnpm install --frozen-lockfile` in CI.
- Keep `--frozen-lockfile` explicit even though CI defaults are frozen in newer pnpm releases.
- Fail builds on lockfile mutation in protected branches.
- Use `--prefer-offline` only when cache health is monitored.
- Never bypass lockfile checks to “fix CI quickly”.
- Keep one package manager per repo.

## Filtered Execution and Task Hygiene
- Use `--filter` for targeted builds/tests during development.
- Use recursive run for coordinated monorepo actions.
- Keep script names consistent across packages.
- Avoid broad root commands that hide failing packages.
- Capture summary output for CI diagnosis.

### Common Commands
```bash
pnpm install --frozen-lockfile
pnpm -r run build
pnpm --filter ./packages/core... test
pnpm --filter "...[origin/main]" run lint
pnpm audit
```

## CI and Operations
- Cache pnpm store, not `node_modules` snapshots.
- Pin Node and pnpm versions in CI toolchain setup.
- Separate dependency install from build/test stages.
- Use `pnpm fetch` in Docker/CI to prefill the store from the lockfile, then install with `--offline`.
- Ensure private registry auth is scoped and rotated.
- Track install time regressions and cache miss rates.

## Security and Supply Chain
- Enforce lockfile reviews for critical changes.
- Audit vulnerable dependencies and document exceptions.
- Restrict install-time scripts where policy requires.
- Validate integrity of published artifacts before promotion.

## Validation Commands
```bash
pnpm install --frozen-lockfile
pnpm -r run typecheck
pnpm -r run test
pnpm -r run lint
pnpm dedupe --check
```

## Common Failure Modes
- Using mixed package managers in one workspace.
- Ignoring peer dependency warnings until runtime failures.
- Overbroad filters that skip dependent packages.
- Mutable lockfile behavior in CI.
- Hidden transitive version drift from unreviewed overrides.

## Definition of Done
- Workspace boundaries and package globs are intentional.
- Internal dependencies use `workspace:` consistently.
- CI installs are deterministic and lockfile-enforced.
- Filtered commands are documented for core workflows.
- Security and upgrade checks are automated.

## References
- `references/pnpm-2026-02-18.md`

## Reference Index
- `rg -n "workspace|pnpm-workspace.yaml|workspace:" references/pnpm-2026-02-18.md`
- `rg -n "frozen-lockfile|CI|install" references/pnpm-2026-02-18.md`
- `rg -n "filter|recursive|scripts" references/pnpm-2026-02-18.md`
- `rg -n "audit|dedupe|outdated" references/pnpm-2026-02-18.md`
