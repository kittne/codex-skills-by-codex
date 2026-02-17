---
name: playwright-testing
description: >
  Skill for Playwright-driven web UI testing, exploration, test generation, execution, and debugging
  (TypeScript or Python). Triggers for tasks involving application testing, validation, test
  generation, or UI automation in web applications.
---

# Playwright Testing

## Workflow
1. Confirm Playwright setup and target app URL.
2. Choose language (TypeScript or Python) based on repo.
3. Record or author tests with stable selectors.
4. Add assertions for key UI states and flows.
5. Run tests and debug failures.

## Quick Intake (Ask/Confirm)
- Target environment (local dev server, staging URL, ephemeral preview).
- Auth needs (anonymous vs logged-in roles) and how to obtain credentials safely.
- State management (seeded DB? test accounts? reset between runs?).
- CI expectations (headless, artifacts, retries, cross-browser).

## Setup
- Install Playwright and browsers.
- Use existing test config when present.

Common setup:
```bash
# Node/TS
npm i -D @playwright/test
npx playwright install --with-deps

# Python
pip install pytest-playwright
playwright install
```

## Test Authoring
- Prefer role-based or data-testid selectors.
- Avoid brittle selectors tied to DOM structure.
- Use `expect` for visible, enabled, or text states.
- Prefer `playwright codegen` for quick prototypes, then harden selectors/assertions.

Rules of thumb:
- Prefer `getByRole`/`get_by_role` and `getByTestId`/`get_by_test_id`.
- Avoid XPath and deep CSS chains unless there is no stable alternative.
- Avoid static sleeps (`waitForTimeout` / `time.sleep`) in favor of web-first assertions.
- Keep tests focused: one primary user outcome per test.

Authentication patterns:
- For large suites, reuse authenticated state via `storageState` (keep out of git).
- Use separate storage states for different roles (admin vs user) to avoid cross-test interference.

## Running Tests
- Use headless runs for CI and headed for local debugging.
- Capture traces/screenshots on failure.

Common commands:
```bash
# TS
npx playwright test
npx playwright test --ui
npx playwright test --debug
npx playwright show-trace trace.zip

# Python (pytest-playwright)
pytest
PWDEBUG=1 pytest -k test_name
```

## Debugging
- Use `--debug`, `--trace`, and `page.pause()`.
- Isolate flaky tests with retries and explicit waits.
- Use the trace viewer to understand timing and locator failures.

Flakiness triage:
- Re-run the single test in isolation; if it only fails in-suite, suspect shared state.
- Ensure assertions are waiting on the right UI signal (URL change, role-visible element, network idle as needed).
- Prefer fixing root cause over increasing timeouts/retries.

## Review Checklist
- Selectors are stable and readable.
- Tests assert user-visible outcomes.
- Failures provide trace/screenshot evidence.
- Tests are isolated (no order dependence).
- CI captures artifacts (HTML report, traces, screenshots) for failures.

## References
- `references/playwright.md`

## Extended Guidance
Use this when test flakiness is high or when scaling test suites.

## Test Design Rules
- Prefer role-based selectors; avoid brittle CSS paths.
- Keep one assertion per user-visible outcome.
- Use fixtures for shared state (auth, seeds, test data).
- Isolate tests; avoid cross-test dependencies.

## Flake Reduction Tactics
- Wait for stable UI states, not arbitrary timeouts.
- Use `expect(...).toBeVisible()` with retries instead of `waitForTimeout`.
- Remove reliance on animation timing; disable animations in test env.

## CI Tips
- Shard tests by file or tag when suites grow.
- Collect trace on failure for debug.
- Keep browser versions pinned in CI for reproducibility.

## Debug Commands
```bash
npx playwright test --debug
npx playwright test --trace on
```

## Reference Index
- `rg -n "Selectors|getByRole" references/playwright.md`
- `rg -n "Fixtures|test.extend" references/playwright.md`
- `rg -n "Tracing|debug" references/playwright.md`

## Coverage Checklist
- Auth flows (login, logout, token refresh).
- Critical paths (checkout, onboarding, settings).
- Error states (404, 500, empty data).

## Reference Index (Expanded)
- `rg -n "CI|sharding" references/playwright.md`

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
- `rg -n "Checklist|checklist" references/playwright.md`
- `rg -n "Example|examples" references/playwright.md`
- `rg -n "Workflow|process" references/playwright.md`
- `rg -n "Pitfall|anti-pattern" references/playwright.md`
- `rg -n "Testing|validation" references/playwright.md`
- `rg -n "Security|risk" references/playwright.md`
- `rg -n "Configuration|config" references/playwright.md`
- `rg -n "Deployment|operations" references/playwright.md`
- `rg -n "Troubleshoot|debug" references/playwright.md`
- `rg -n "Performance|latency" references/playwright.md`
- `rg -n "Reliability|availability" references/playwright.md`
- `rg -n "Monitoring|metrics" references/playwright.md`
- `rg -n "Error|failure" references/playwright.md`
- `rg -n "Decision|tradeoff" references/playwright.md`
- `rg -n "Migration|upgrade" references/playwright.md`
