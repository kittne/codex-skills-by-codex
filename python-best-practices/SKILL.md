---
name: python-best-practices
description: >
  Comprehensive Python best practices for coding, reviewing, and maintaining code. Enforce Python
  3.10+, full type hinting, strict static analysis (Mypy/Pyright), consistent formatting (Black and
  Ruff with 120-char lines, double quotes, sorted imports), robust logging, safe async patterns,
  strong exception handling, secure coding (no secrets in logs/config), and thorough pytest-based
  testing.
---

# Python Best Practices

## Workflow
1. Confirm Python version and project layout.
2. Enforce typing and static analysis (strict).
3. Format and lint consistently (Black + Ruff).
4. Apply logging, error handling, async, and security rules.
5. Add/update tests and validate (pytest; regression coverage).

## Preflight (Ask/Check First)
- Python version target (prefer 3.10+; align with CI/runtime).
- Packaging layout (`src/` layout vs flat; `pyproject.toml` conventions).
- Existing toolchain config:
  - `pyproject.toml` sections for Black/Ruff/Mypy/Pyright.
  - Existing CI steps (lint/typecheck/test).
- Framework context (FastAPI/Django/CLI/batch job) and async usage.

## Version and Structure
- Target Python 3.10+.
- Prefer a `src/` layout for import hygiene when appropriate.
- Keep modules cohesive and avoid circular imports.
- Avoid import-time side effects; keep module import cheap and deterministic.

## Typing and Static Analysis
- Type all public functions and complex internal logic.
- Prefer narrow, explicit types over `Any`.
- Use `mypy --strict` and/or `pyright` when configured.
- Avoid `# type: ignore` unless unavoidable; if used, target specific codes and add a reason.
- Prefer `TypedDict`, `Protocol`, and `Literal` for clearer contracts where applicable.

## Formatting and Linting
- Use Black (line length 120) when the repo uses Black.
- Use Ruff for linting and import sorting; apply autofixes where safe.

Common commands:
```bash
ruff check .
ruff check . --fix
ruff format .
black .
mypy --strict .
pyright
```

## Logging
- Prefer structured, contextual logs over prints.
- Never log secrets or sensitive personal data.
- Log exceptions with stack traces at the appropriate boundary.
- Prefer lazy formatting (`logger.info("x=%s", x)`) over f-strings for expensive values.

## Exceptions
- Fail fast with clear error messages.
- Preserve the original exception via chaining (`raise ... from e`) when wrapping.
- Avoid broad `except Exception` unless at a top-level boundary with logging.
- Do not swallow exceptions silently.

## Async Patterns
- Avoid blocking calls in the event loop.
- Use a single shared client per process when possible (HTTP, DB pools).
- Ensure cancellation/timeouts are respected.
- In FastAPI, avoid sync I/O inside `async def`; use `def` endpoints for sync stacks.

## Security
- Validate all external inputs.
- Keep secrets in a secrets manager or environment (never in code).
- Pin dependencies and scan for known vulnerabilities when the repo supports it.
- Treat logs and error messages as sensitive output channels.

## Testing
- Use pytest with fixtures and clear test names.
- Add regression tests for bug fixes.
- Prefer deterministic tests (avoid time/network flakiness).
- Cover failure paths and boundary conditions for risky code.

## References
- `references/python-best-practices.md`

## Extended Guidance
Use this when setting baseline project standards or troubleshooting difficult bugs.

## Project Defaults Checklist
- Use `pyproject.toml` as the single source of tooling config.
- Pin Python version in `.python-version` or CI matrix.
- Enable Ruff + Black + type checker in CI.
- Keep runtime and dev dependencies separate.

## Type Checking Policy
- Default to strict type checking.
- Avoid `Any` unless unavoidable; justify ignores inline.
- Prefer typed wrappers over `# type: ignore` at call sites.

## Testing Strategy
- Use pytest with fixtures for isolation.
- Favor fast unit tests for pure logic and a small number of integration tests.
- Add regression tests for every bug fix.

## Logging and Observability
- Use structured logging for services.
- Avoid logging secrets or PII.
- Include request IDs in logs for correlation.

## Reference Index
- `rg -n "Type checking|mypy|pyright" references/python-best-practices.md`
- `rg -n "Logging|structlog" references/python-best-practices.md`
- `rg -n "Testing|pytest" references/python-best-practices.md`

## Async and Concurrency
- Avoid blocking calls inside async handlers.
- Use `asyncio.to_thread` for CPU-bound work when necessary.
- Limit concurrent tasks with semaphores.

## Security Defaults
- Validate input at the boundary.
- Prefer parameterized queries and ORM filters.
- Keep secrets out of logs and exceptions.

## Reference Index (Expanded)
- `rg -n "async|await" references/python-best-practices.md`
- `rg -n "security|secrets" references/python-best-practices.md`

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
- `rg -n "Checklist|checklist" references/python-best-practices.md`
- `rg -n "Example|examples" references/python-best-practices.md`
- `rg -n "Workflow|process" references/python-best-practices.md`
- `rg -n "Pitfall|anti-pattern" references/python-best-practices.md`
- `rg -n "Testing|validation" references/python-best-practices.md`
- `rg -n "Security|risk" references/python-best-practices.md`
- `rg -n "Configuration|config" references/python-best-practices.md`
- `rg -n "Deployment|operations" references/python-best-practices.md`
- `rg -n "Troubleshoot|debug" references/python-best-practices.md`
- `rg -n "Performance|latency" references/python-best-practices.md`
- `rg -n "Reliability|availability" references/python-best-practices.md`
- `rg -n "Monitoring|metrics" references/python-best-practices.md`
- `rg -n "Error|failure" references/python-best-practices.md`
- `rg -n "Decision|tradeoff" references/python-best-practices.md`
- `rg -n "Migration|upgrade" references/python-best-practices.md`
