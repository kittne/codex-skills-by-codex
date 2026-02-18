---
name: ruff
description: >
  Build, tune, and operate Ruff for Python linting, formatting, and editor/CI integration. Use when
  adding or updating Ruff configuration, migrating from Black/Flake8/isort, selecting rule families,
  enforcing fix safety, or debugging lint/format behavior in local development, pre-commit, and CI.
---

# Ruff

## Workflow
1. Detect current Ruff version, Python targets, and existing lint/format tools.
2. Set one canonical Ruff configuration and inheritance strategy.
3. Select rule families in adoption layers to control migration noise.
4. Configure formatter behavior and notebook handling.
5. Integrate editor, pre-commit, and CI with version pinning.
6. Apply safe autofixes first, then isolate unsafe changes.
7. Tune cache and file discovery for monorepo-scale performance.

## Preflight (Ask / Check First)
- Check whether the repo already uses `pyproject.toml`, `ruff.toml`, or `.ruff.toml`.
- Check whether Black, Flake8 plugins, or isort are already enforced in CI.
- Check target Python versions across runtime and tooling.
- Check whether Jupyter notebooks are in scope for linting or formatting.
- Check whether the team wants preview features or strict stability.

## Version and Stability Policy
- Pin Ruff in CI and pre-commit because minor releases can include breaking changes.
- Set `required-version` when deterministic behavior matters across contributors.
- Keep `preview = false` by default.
- Enable preview only in controlled trials and document rollback steps.
- Prefer explicit upgrades with changelog review rather than floating latest.

## Canonical Configuration Pattern
- Keep one root config as the policy source of truth.
- Use `extend` for subproject inheritance in monorepos.
- Avoid duplicated rule lists in child configs unless divergence is intentional.
- Set `target-version` explicitly for stable parsing and formatting behavior.
- Use `src` and `respect-gitignore` to improve import classification and discovery.

Example baseline:
```toml
[tool.ruff]
target-version = "py311"
line-length = 100
src = ["src"]
respect-gitignore = true
required-version = ">=0.15.1"

[tool.ruff.lint]
extend-select = ["I", "UP", "B", "SIM", "RUF"]
ignore = ["E501"]

[tool.ruff.format]
preview = false
```

## Rule Selection Strategy
- Start from Ruff defaults for high-signal correctness checks.
- Add rule families in layers: imports, upgrades, bug-risk, simplifications.
- Add docstring or complexity families only after team policy is clear.
- Use `per-file-ignores` for tests, package `__init__.py`, and generated code.
- Track suppressions and remove stale ignores during cleanup cycles.

## Formatter Strategy
- Choose one formatter owner for the repo.
- Prefer `ruff format` when replacing Black for speed and single-tool consistency.
- Avoid style rules that conflict with formatter ownership.
- Exclude generated files and migration outputs from formatting.
- Define notebook behavior explicitly if `.ipynb` files are tracked.

## Autofix Safety and Change Control
- Run safe fixes first: `ruff check . --fix`.
- Run unsafe fixes only in isolated PRs: `ruff check . --fix --unsafe-fixes`.
- Review runtime and typing-sensitive diffs before merging unsafe fixes.
- Avoid combining policy changes and mass autofix in one commit.
- Keep modernization waves small enough for reviewers to reason about.

## Monorepo and Inheritance Guidance
- Use closest-config behavior intentionally.
- Remember Ruff does not implicitly merge parent configs.
- Use child `extend` references to preserve organization baseline.
- Prefer `extend-select` over `select` in children unless resetting is intentional.
- Keep package-specific deviations narrow and documented inline.

## Editor, Pre-commit, and CI Integration
- Use Ruff language server integration where editor support is available.
- Ensure editor and CI run compatible Ruff versions.
- Set `force-exclude = true` if pre-commit passes explicit paths.
- Run lint and format checks as separate CI steps for clearer failures.
- Emit machine-readable output when CI annotations are required.

Suggested commands:
```bash
ruff check .
ruff check . --fix
ruff format .
ruff check . --output-format=github
```

## Performance Operations
- Keep `.ruff_cache` enabled for incremental speed.
- Set `RUFF_CACHE_DIR` when workspace cache placement matters.
- Restrict analyzed paths with `src`, `exclude`, and focused CI paths.
- Avoid running repository-wide checks when change-scoped jobs are enough.
- Profile slow runs before widening ignore patterns.

## Migration Playbook
- Baseline with `ruff check .` and record current diagnostics volume.
- Introduce minimal rule expansion and fix low-risk issues first.
- Enforce no-regression policy in CI before strict expansion.
- Migrate formatter ownership in a dedicated PR.
- Remove legacy linter plugins only after parity is validated.

## Failure Modes to Catch Early
- Mixing multiple formatters without clear ownership.
- Resetting rule baselines accidentally in child configs.
- Enabling preview features in CI without local parity.
- Applying unsafe fixes without targeted review.
- Letting ignore lists grow without cleanup ownership.

## Definition of Done
- Keep one canonical Ruff config with intentional inheritance.
- Keep lint and format checks deterministic across local, pre-commit, and CI.
- Pin Ruff versions and document upgrade cadence.
- Enforce safe autofix by default and isolated unsafe migrations.
- Keep suppressions small, explicit, and periodically reviewed.

## References
- `references/ruff-2026-02-17.md`

## Reference Index
- `rg -n "Version|preview|required-version" references/ruff-2026-02-17.md`
- `rg -n "extend|monorepo|closest config" references/ruff-2026-02-17.md`
- `rg -n "rule|extend-select|per-file-ignores" references/ruff-2026-02-17.md`
- `rg -n "formatter|Black|ruff format" references/ruff-2026-02-17.md`
- `rg -n "cache|RUFF_CACHE_DIR|performance" references/ruff-2026-02-17.md`
- `rg -n "pre-commit|CI|output-format" references/ruff-2026-02-17.md`
