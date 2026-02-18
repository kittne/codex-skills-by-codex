---
name: pyright
description: >
  Configure and operate Pyright for fast static typing in Python projects, including editor alignment,
  strictness rollout, import resolution, stub strategy, and CI enforcement. Use when creating
  `pyrightconfig.json` or `[tool.pyright]`, migrating to stricter typing, or debugging type-analysis
  drift across local and CI environments.
---

# Pyright

## Workflow
1. Detect current Python targets, repo layout, and editor setup.
2. Choose one source of truth: `pyrightconfig.json` or `[tool.pyright]`.
3. Stabilize include/exclude/import resolution and execution environments.
4. Set baseline strictness and roll strict mode by path.
5. Address third-party typing with stubs and typed packages.
6. Wire pinned Pyright checks into CI and enforce no-regression policy.

## Preflight (Ask / Check First)
- Check whether both `pyrightconfig.json` and `[tool.pyright]` exist.
- Check whether VS Code uses Pylance and whether config files override editor settings.
- Check Python versions that must be supported at runtime.
- Check whether `src/` layout or monorepo roots require explicit environments.
- Check whether strict typing is expected globally or incrementally.

## Configuration Source of Truth
- Prefer a single config location to avoid local/CI drift.
- Remember `pyrightconfig.json` takes precedence over `[tool.pyright]`.
- Keep all paths relative to the config file location.
- Avoid absolute or user-home paths in shared configs.
- Use `extends` for organization-wide base policy reuse.

Example baseline:
```json
{
  "include": ["src"],
  "exclude": ["**/__pycache__", "build", "dist"],
  "typeCheckingMode": "standard",
  "pythonVersion": "3.11",
  "pythonPlatform": "All",
  "strict": ["src/core", "src/domain"],
  "stubPath": "typings"
}
```

## Strictness Strategy
- Start with `standard` or `basic` for repository-wide baseline.
- Opt into strict by path using `strict` array or `# pyright: strict`.
- Promote strict coverage directory-by-directory as debt decreases.
- Keep rule overrides narrow and intentional.
- Avoid broad suppression that hides unknown-type spread.

## Import Resolution and Environments
- Define `include` roots explicitly for predictable analysis scope.
- Use `executionEnvironments` in monorepos with mixed roots or versions.
- Use `extraPaths` sparingly and prefer packaging-correct imports.
- Remember excluded files can still be analyzed if imported.
- Use verbose diagnostics temporarily when resolution is unclear.

## Third-Party Typing and Stubs
- Prefer dependencies that ship `py.typed` metadata.
- Add or pin stub packages when libraries lack inline types.
- Keep local custom stubs under `stubPath` with package-accurate structure.
- Treat strict unknown-type errors as signals to improve dependency typing.
- Avoid relying on inferred library code types for critical APIs.

## Editor and CLI Alignment
- Align local CLI and editor engine versions where practical.
- Prefer Pylance for VS Code IDE experience, with Pyright semantics in sync.
- Document one canonical local command for contributors.
- Use `--project` in scripts to force intended config file.
- Use `--outputjson` for machine-readable CI annotation pipelines.

Suggested commands:
```bash
pyright --project pyrightconfig.json
pyright --project pyrightconfig.json --warnings
pyright --project pyrightconfig.json --outputjson
pyright --project pyrightconfig.json --watch
```

## CI and Release Governance
- Pin Pyright versions in CI actions or lockfiles.
- Fail checks on type errors; fail on warnings when maturity allows.
- Keep type checking as a dedicated job for clear triage.
- Run targeted checks on changed packages when full checks are too slow.
- Validate config changes with representative module coverage before merge.

## Gradual Adoption Playbook
- Baseline with standard mode and explicit include roots.
- Strict-enable one high-ownership package first.
- Replace broad ignores with typed protocols, stubs, or wrappers.
- Expand strict coverage based on stable module ownership.
- Keep a backlog of unknown-type hotspots and burn it down incrementally.

## Common Failure Modes
- Defining both config formats and assuming both are active.
- Letting implicit default Python version drift drive noisy changes.
- Using `extraPaths` as a permanent substitute for import hygiene.
- Enabling strict globally before third-party typing is ready.
- Hiding unresolved typing debt with path-wide ignores.

## Definition of Done
- Keep one authoritative Pyright configuration.
- Keep Python version and platform targets explicit.
- Keep strictness rollout incremental and measurable.
- Keep third-party typing strategy explicit with pinned stubs.
- Keep CI deterministic with pinned toolchain and clear failure output.

## References
- `references/pyright-2026-02-17.md`

## Reference Index
- `rg -n "pyrightconfig|pyproject|precedence|extends" references/pyright-2026-02-17.md`
- `rg -n "typeCheckingMode|strict|unknown|warnings" references/pyright-2026-02-17.md`
- `rg -n "include|exclude|executionEnvironments|extraPaths" references/pyright-2026-02-17.md`
- `rg -n "PEP 561|stub|py.typed|import resolution" references/pyright-2026-02-17.md`
- `rg -n "Pylance|CLI|--outputjson|--watch|CI" references/pyright-2026-02-17.md`
- `rg -n "pythonVersion|defaults|release" references/pyright-2026-02-17.md`
