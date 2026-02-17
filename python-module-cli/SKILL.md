---
name: python-module-cli
description: Build, package, and test Python command-line interfaces implemented as installable modules/packages. Use for designing CLI UX contracts (subcommands, flags, stdout/stderr, exit codes, JSON output, color/NO_COLOR), choosing argparse vs Click/Typer, configuring `pyproject.toml` entry points, supporting `python -m` via `__main__.py`, config precedence (flags/env/config), logging/verbosity, safe filesystem behavior, and CLI test strategy.
---

# Python Module CLI

## Default Architecture

- Keep domain logic importable and testable (no CLI parsing in core code).
- Keep the CLI layer thin: parse args, assemble config, call core, format output, return an exit code.
- Provide a single `main(argv=None) -> int` and a single top-level exit path.
- Support `python -m <package>` via `__main__.py` (useful for dev/debug and environments without entry point scripts).

## Quick Intake (Ask/Confirm)
- Target Python version and packaging system (`pyproject.toml`, `src/` layout).
- Expected users: humans, automation/CI, or both.
- Output contract: human text vs machine-readable (`--json` / `--format json`).
- Config sources: flags/env/config file and precedence rules.
- Safety requirements: destructive actions, filesystem writes, secrets handling.

Minimal `main` pattern:

```python
# src/mytool/cli.py
from __future__ import annotations

import sys


def main(argv: list[str] | None = None) -> int:
    argv = sys.argv[1:] if argv is None else argv
    # parse args, call core logic, print output
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

Minimal `python -m` support:

```python
# src/mytool/__main__.py
from __future__ import annotations

from .cli import main

raise SystemExit(main())
```

## Build Workflow (Use This Order)

1. Define the public contract.
   - Decide commands/subcommands and required args.
   - Standardize flags across commands (`-h/--help`, `--version`, `-v/--verbose`, `-q/--quiet`).
   - Decide output contracts (human vs machine) and keep them stable.
   - Define and document exit codes (keep them stable once published).
2. Pick one parsing approach.
   - Prefer stdlib `argparse` unless Click/Typer’s multi-command ergonomics are needed.
   - Avoid mixing frameworks unless there is a clear, justified reason.
3. Implement output separation and formatting.
   - Write primary output to stdout.
   - Write diagnostics, logs, progress, and human guidance to stderr.
   - Add a structured output mode (`--json` or `--format json`) when the tool outputs “data”.
4. Implement errors and exit codes deliberately.
   - Return non-zero on failure; keep exit code meanings predictable.
   - Treat usage errors as a distinct category (commonly exit code `2`).
   - Avoid tracebacks by default; gate behind `--debug/--traceback` or higher verbosity (`-vv`).
5. Implement configuration and paths.
   - Use precedence: CLI flags > environment variables > config file > defaults.
   - Use `platformdirs` for config/cache/data locations; avoid hardcoding OS paths.
   - Provide `--config PATH` for explicit config selection.
   - Avoid storing secrets in plaintext config by default; prefer env vars or secret managers.
6. Package as a modern Python project.
   - Prefer a `src/` layout for packaged CLIs.
   - Define entry points in `pyproject.toml` via `[project.scripts]` (no raw scripts on PATH).
   - Keep runtime dependencies minimal; document `pipx` for end-user installs.
7. Test at multiple layers.
   - Unit-test core logic directly (no subprocess).
   - Test parsing/defaulting/validation separately.
   - Add end-to-end tests that run the CLI and assert: exit code, stdout, stderr, and JSON stability.

## Exit Codes (Default Convention)
- `0`: success
- `1`: runtime failure (unexpected error, external dependency failure)
- `2`: usage error (invalid args, missing required inputs)

If the repo already uses a different convention, follow it and document it.

## Decision Rules (Fast Defaults)

- Keep `--help` and `--version` fast: avoid heavy imports, network calls, or expensive initialization unless needed.
- Make color TTY-aware and honor `NO_COLOR`; add `--color auto|always|never` only when color matters.
- Avoid surprising filesystem writes (especially into CWD); require explicit `--output` for files and support `--dry-run` for mutating operations.
- Never print secrets; audit debug logs too.

## Testing Strategy (CLI-Specific)
- Prefer unit tests for core behavior (fast, deterministic).
- Add subprocess E2E tests for:
  - `--help` / `--version` output.
  - Exit codes on success/failure.
  - Stdout/stderr separation.
  - JSON output stability (exact keys, stable types).
- Ensure tests don't depend on network or global filesystem state unless explicitly mocked.

## Reference

- Use `references/python-module-cli-best-practices.md` for deeper guidance, templates, and checklists.
- Jump to topics quickly with search:
  - `rg -n 'Architecture: keep CLI code thin' references/python-module-cli-best-practices.md`
  - `rg -n 'Choosing an argument parsing approach' references/python-module-cli-best-practices.md`
  - `rg -n 'Configuration:' references/python-module-cli-best-practices.md`
  - `rg -n 'Logging and observability' references/python-module-cli-best-practices.md`
  - `rg -n 'Testing strategy for CLIs' references/python-module-cli-best-practices.md`

## Extended Guidance
Use this when CLIs are part of automation or when the output contract is strict.

## Exit Codes (Expanded)
- `0`: success.
- `1`: runtime failure (unexpected exception, dependency failure).
- `2`: usage error (invalid args, missing inputs).
- `3`: environment/config missing (env var, config file, auth).

## Config Precedence (Example)
```text
Flags > Environment > Config file > Defaults
```

## Testing Checklist (Expanded)
- `python -m <module>` works in a fresh environment.
- `--help` and `--version` are fast and side-effect free.
- Output modes (`--json` or `--format json`) are stable across releases.
- Exit codes are documented and asserted in tests.

## Reference Index
- `rg -n "Exit codes|stderr" references/python-module-cli-best-practices.md`
- `rg -n "Configuration" references/python-module-cli-best-practices.md`

## Operational Checklist
- Document all environment variables.
- Keep stderr free of normal output.

## Quick Questions (When Stuck)
- What is the minimal change that solves the issue?
- What is the rollback plan?
- What is the highest-risk assumption?
- What is the simplest validation step?
- What is the known-good baseline?
- What evidence would change the decision?
- What is the user-visible impact?
- What is the operational impact?
- What is the most likely failure mode?
- What is the fastest safe experiment?

## Reference Index (Extra)
- `rg -n "Checklist|checklist" references/python-module-cli-best-practices.md`
- `rg -n "Example|examples" references/python-module-cli-best-practices.md`
- `rg -n "Workflow|process" references/python-module-cli-best-practices.md`
- `rg -n "Pitfall|anti-pattern" references/python-module-cli-best-practices.md`
- `rg -n "Testing|validation" references/python-module-cli-best-practices.md`
- `rg -n "Security|risk" references/python-module-cli-best-practices.md`
- `rg -n "Configuration|config" references/python-module-cli-best-practices.md`
- `rg -n "Deployment|operations" references/python-module-cli-best-practices.md`
- `rg -n "Troubleshoot|debug" references/python-module-cli-best-practices.md`
- `rg -n "Performance|latency" references/python-module-cli-best-practices.md`
- `rg -n "Reliability|availability" references/python-module-cli-best-practices.md`
- `rg -n "Monitoring|metrics" references/python-module-cli-best-practices.md`
- `rg -n "Error|failure" references/python-module-cli-best-practices.md`
- `rg -n "Decision|tradeoff" references/python-module-cli-best-practices.md`
- `rg -n "Migration|upgrade" references/python-module-cli-best-practices.md`
