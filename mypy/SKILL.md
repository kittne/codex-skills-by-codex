---
name: mypy
description: >
  Design and enforce production mypy type-checking workflows for Python services and libraries. Use
  when creating or tuning mypy configuration, planning gradual typing adoption, handling stubs and
  imports, configuring plugins, or stabilizing mypy behavior in local development and CI pipelines.
---

# Mypy

## Workflow
1. Identify runtime Python versions, packaging layout, and current checker usage.
2. Standardize one canonical mypy config and one invocation contract.
3. Stabilize imports and third-party typing before raising strictness.
4. Roll out strictness incrementally by package ownership.
5. Wire deterministic checks into CI with pinned versions.
6. Track suppressions and reduce `Any` escapes over time.

## Preflight (Ask / Check First)
- Check whether the repo uses `mypy.ini`, `.mypy.ini`, `pyproject.toml`, or `setup.cfg`.
- Check whether multiple configs already exist across subdirectories.
- Check target runtime Python versions versus tooling Python versions.
- Check whether the codebase depends on untyped third-party libraries.
- Check whether framework plugins are required for typing fidelity.

## Version and Compatibility Policy
- Pin mypy and stub-package versions together for reproducible checks.
- Set `python_version` explicitly in config.
- Do not target Python 3.8; current mypy requires `--python-version 3.9+`.
- Align tooling runtime with supported mypy runtime requirements.
- Treat Python EOL versions as migration priorities, not permanent targets.
- Review mypy release notes before bumping strict or plugin-heavy setups.
- If using PEP 702 `@warnings.deprecated`, enable `deprecated` error code or `report_deprecated_as_note` explicitly.

## Configuration Architecture
- Keep one canonical config per invocation.
- Remember mypy does not merge discovered configs.
- Use per-module overrides for gradual adoption.
- Use inline `# mypy:` directives only for tightly scoped exceptions.
- Keep config settings readable and explicitly commented when non-obvious.

Example baseline:
```toml
[tool.mypy]
python_version = "3.12"
warn_unused_configs = true
warn_unused_ignores = true
check_untyped_defs = true
files = ["src"]

[[tool.mypy.overrides]]
module = ["tests.*"]
ignore_errors = true
```

## Strictness Ladder
- Start with reliability flags before enabling full strict mode.
- Enforce `disallow_untyped_defs` in mature modules first.
- Expand `disallow_untyped_calls` after third-party typing improves.
- Adopt `strict = true` only where ownership and dependency typing support it.
- Prefer per-module strict expansion over repo-wide noisy transitions.
- Remember `--strict` does not enable `--warn-unreachable`, and its flag set can change by version.

## Import and Stub Strategy
- Prefer typed libraries that ship `py.typed` metadata.
- Install and pin `types-...` stub packages explicitly.
- Avoid global `ignore_missing_imports = true` unless in temporary migration mode.
- Add local stubs for critical internal or missing dependency interfaces.
- Keep `mypy_path` repo-relative and avoid pointing at `site-packages`.

## Plugin Guidance
- Enable plugins only when runtime framework behavior requires it.
- Keep plugin versions pinned with mypy versions.
- Treat plugin APIs as change-prone and test upgrades explicitly.
- Prefer framework-native typing improvements over legacy plugin dependence.
- Remove obsolete plugin dependencies during modernization.

## Performance and Local Developer Loop
- Use `dmypy` for faster iterative checks in large repositories.
- Keep cache enabled and isolate cache directories when needed.
- Run path-scoped checks locally before full-repo CI checks.
- Profile slow modules with verbose mode before broad suppressions.
- Keep type-check jobs separate from tests for clearer diagnostics.

Suggested commands:
```bash
mypy --config-file mypy.ini
dmypy run -- --config-file mypy.ini src
mypy --config-file mypy.ini --show-error-codes
```

## CI and Governance
- Pin mypy and stubs in lockfiles or constraints.
- Run one canonical command in CI and document it in contributor docs.
- Fail CI on new typing regressions.
- Track suppression count and stale ignore usage.
- Require type checks on changed Python modules before merge.

## Migration Playbook
- Establish a passing baseline on a narrow package set.
- Quarantine legacy modules with explicit `ignore_errors` scopes.
- Add ownership-driven strict modules every sprint.
- Replace broad suppressions with typed wrappers and protocols.
- Convert migration progress into measurable CI gates.

## Failure Modes to Catch Early
- Splitting config across files and assuming merge behavior.
- Pinning mypy but leaving stubs unpinned.
- Hiding import issues globally with `ignore_missing_imports`.
- Enabling strict globally before dependency typing is ready.
- Using broad `# type: ignore` without error codes or rationale.

## Definition of Done
- Keep one canonical mypy config and one deterministic command.
- Keep Python target semantics explicit and stable.
- Keep import typing debt visible and shrinking.
- Keep strictness expanding by module ownership with low-noise CI.
- Keep suppressions narrow, justified, and periodically removed.

## References
- `references/mypy-2026-02-17.md`

## Reference Index
- `rg -n "version|Python 3.8|Python 3.9|release" references/mypy-2026-02-17.md`
- `rg -n "config|no config merging|--config-file" references/mypy-2026-02-17.md`
- `rg -n "strict|disallow|warn_unused" references/mypy-2026-02-17.md`
- `rg -n "stub|typeshed|PEP 561|ignore_missing_imports" references/mypy-2026-02-17.md`
- `rg -n "plugin|django|SQLAlchemy|attrs" references/mypy-2026-02-17.md`
- `rg -n "dmypy|cache|performance|existing codebase" references/mypy-2026-02-17.md`
