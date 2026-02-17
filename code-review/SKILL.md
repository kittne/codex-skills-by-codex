---
name: code-review
description: >
  Perform a comprehensive, language-agnostic code review covering design, code quality, correctness,
  security (OWASP Top 10), performance, testing, and documentation. Use when the user requests a code
  review (e.g. PR feedback); do not trigger for general coding queries.
---

# Code Review

## Workflow
1. Preflight: establish scope (PR intent), risk profile, and how to run/verify changes.
2. High-level pass: identify release blockers (security, correctness, data loss, operational hazards).
3. Detailed pass: review correctness, security, performance, maintainability, and consistency with the codebase.
4. Validation pass: check tests, migrations, docs, config changes, and rollout/rollback.
5. Write up findings by severity with file/line references and concrete fix suggestions.

## Preflight (Fast Intake)
- What is the change trying to accomplish (requirements, user story, bug)?
- What parts of the system are touched (API, DB, auth, infra)?
- What is the risk: data migration, auth changes, backwards compatibility, perf regressions?
- How can we validate locally: commands, test target, feature flags, environments?

## Focus Areas
- Architecture and design: requirements fit, consistency with existing patterns, migration/rollout.
- Correctness and robustness: edge cases, error handling, concurrency/timeouts, data integrity.
- Security: input validation, authn/authz, secrets handling, secure transport, dependency risk.
- Code quality: naming, simplicity, duplication, readability, dead code.
- Performance: algorithmic complexity, query patterns (N+1), resource usage, client-side impact.
- Testing: coverage for changed behavior, failure-path tests, flaky tests, deterministic setups.
- Documentation: user-facing docs, API docs, config changes, changelog/migrations.

## High-Signal Red Flags (Prioritize)
- Authz missing on object-level access (IDOR risk).
- Unsafe string building into SQL/NoSQL/shell/templating contexts.
- Secrets introduced in code/logs/committed config.
- Data migrations without rollback/verification or without handling existing data.
- Concurrency without cancellation/timeouts; potential goroutine/thread/task leaks.
- Retries without jitter/backoff; unbounded queues; missing rate limits.
- Breaking API/schema changes without versioning or coordination.
- Error handling that swallows failures or loses context.

## Severity
- Critical: must fix before merge (vuln, data loss, crash, auth bypass).
- Important: should fix soon (logic risk, missing tests, performance regression, maintainability).
- Suggestion: nice-to-have (refactors, naming, minor cleanup).

## Security Pass (Minimum Checklist)
- Inputs validated server-side; outputs encoded by context.
- Authn/authz enforced per request and per object.
- CSRF/session/cookie settings sane (if web).
- Secrets not logged; config uses secret manager patterns where available.
- Dependencies pinned/locked; risky upgrades called out; supply chain checks noted.

## Output Format
Use this structure and put findings first:

```markdown
## Findings

### Critical
- **<File>:<line>**: <Issue>\n  Impact: <Why it matters>\n  Fix: <Concrete next step>

### Important
- **<File>:<line>**: ...

### Suggestions
- **<File>:<line>**: ...

## Questions
- <Blocking question if any>

## Testing
- <What you ran / what to run>

## Notes
- <Positive callouts or follow-ups>
```

## Review Execution Tips
- Prefer pointing to exact code locations and describing impact + fix.
- Ask targeted questions only when the review is blocked (missing context, unclear requirements).
- Call out what was not reviewed (e.g., "did not run tests", "no DB access").
- If you can run tests/lints locally, do it and report the commands/outcome.

## References
- `references/code-review.md`

## Extended Guidance
Use this when reviewing risky changes or security-sensitive code.

## Deep-Dive Checklists
- Security: authn/authz checks, input validation, secrets handling, dependency risks.
- Performance: hot paths, N+1 queries, unbounded loops, caching validity.
- Reliability: retries, idempotency, timeouts, failure handling.

## Review Artifacts to Request
- Test output and coverage for changed areas.
- Benchmark results (if performance claims are made).
- Migration plan and rollback steps for schema changes.

## Common Failure Modes
- Approving without running or understanding tests.
- Missing backward compatibility for public APIs.
- Silent error handling that hides operational issues.

## Reference Index
- `rg -n "Security checklist|OWASP" references/code-review.md`
- `rg -n "Performance|scalability" references/code-review.md`
- `rg -n "Testing|coverage" references/code-review.md`

## Review Questions (When In Doubt)
- What is the smallest failure mode if this breaks?
- Is there a rollback plan?
- Are migrations backward compatible?
- Are logs/metrics sufficient to diagnose issues?

## Layered Review Checklist
- API layer: authn/authz, rate limits, input validation.
- Data layer: indexes, migrations, and rollback.
- Background jobs: idempotency, retries, dedupe.
- Infra: secrets handling, config defaults, permissions.

## Reference Index (Expanded)
- `rg -n "Correctness|behavior" references/code-review.md`
- `rg -n "Documentation|changelog" references/code-review.md`
- `rg -n "Risk|severity" references/code-review.md`

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
- `rg -n "Checklist|checklist" references/code-review.md`
- `rg -n "Example|examples" references/code-review.md`
- `rg -n "Workflow|process" references/code-review.md`
- `rg -n "Pitfall|anti-pattern" references/code-review.md`
- `rg -n "Testing|validation" references/code-review.md`
- `rg -n "Security|risk" references/code-review.md`
- `rg -n "Configuration|config" references/code-review.md`
- `rg -n "Deployment|operations" references/code-review.md`
- `rg -n "Troubleshoot|debug" references/code-review.md`
- `rg -n "Performance|latency" references/code-review.md`
- `rg -n "Reliability|availability" references/code-review.md`
- `rg -n "Monitoring|metrics" references/code-review.md`
- `rg -n "Error|failure" references/code-review.md`
- `rg -n "Decision|tradeoff" references/code-review.md`
- `rg -n "Migration|upgrade" references/code-review.md`
