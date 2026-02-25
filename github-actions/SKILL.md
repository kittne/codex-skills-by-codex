---
name: github-actions
description: >
  Design, secure, and optimize GitHub Actions workflows for CI/CD at team and organization scale. Use
  when creating or refactoring workflows, hardening permissions and secrets, adopting OIDC, reducing
  runtime cost, scaling reusable pipelines, enforcing governance rulesets, and improving reliability
  and observability for build, test, and deployment automation.
---

# GitHub Actions

## Workflow
1. Confirm repository trust model, runner model, and release requirements.
2. Define least-privilege token and secret strategy.
3. Design workflow triggers and job graph for safe execution.
4. Standardize reusable workflows and pinned action dependencies.
5. Optimize runtime and cost with caching and concurrency controls.
6. Add observability, governance controls, and enforcement checks.
7. Validate deployment gates, rollback readiness, and policy compliance.

## Preflight (Ask / Check First)
- Check whether repositories are public, private, or mixed visibility.
- Check whether runners are GitHub-hosted, self-hosted, or hybrid.
- Check whether deployments require approvals or environment protections.
- Check branch protection, merge queue, and required-check policies.
- Check compliance requirements for provenance, artifacts, and audit trails.

## Identity, Secrets, and Permissions
- Set top-level `permissions` to minimum required scope.
- Elevate permissions only per job that needs write access.
- Prefer OIDC federation over long-lived cloud credentials.
- For OIDC, grant `id-token: write` at workflow/job level and keep `contents: read` for checkout.
- For reusable workflows outside your org/enterprise, set `id-token: write` in the caller workflow.
- Restrict cloud trust policies by repository, ref, and workflow claims.
- Mask sensitive data and avoid logging secret-bearing structures.

Safe baseline:
```yaml
permissions:
  contents: read
```

OIDC example pattern:
```yaml
permissions:
  contents: read
  id-token: write
```

## Trigger and Trust-Boundary Safety
- Use `pull_request` for untrusted code validation.
- Avoid checking out PR head code in privileged contexts.
- Treat `pull_request_target` and `workflow_run` as high-risk triggers.
- Separate trusted metadata jobs from untrusted test execution.
- Guard manual dispatch and reusable workflow inputs with validation.

## Action Supply-Chain Security
- Pin third-party actions to full commit SHAs.
- Keep official core actions on supported major versions.
- Enforce allowlists and SHA pinning policies at org level where available.
- Automate dependency updates for action versions with review gates.
- Add dependency review and workflow security scanning checks.

## Workflow Architecture and Reuse
- Keep workflows focused: fast feedback, heavy checks, deploy stages.
- Factor common logic into reusable workflows with `workflow_call`.
- Use composite actions for repeated step sequences.
- Keep inputs, outputs, and required secrets explicitly documented.
- Avoid duplicating deployment logic across repositories.

## Performance and Cost Controls
- Use cache keys based on lockfiles, not commit SHAs.
- Add `concurrency` groups to cancel superseded PR runs.
- Cap matrix fan-out when queueing or compute contention appears.
- Prefer Linux runners by default when workload permits.
- Tune artifact retention and cache retention to measured need.

Example concurrency pattern:
```yaml
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
```

## Reliability Patterns
- Make jobs idempotent so reruns are safe.
- Add bounded retry with backoff for flaky external dependencies.
- Keep artifact naming deterministic and collision-resistant.
- Set explicit job timeouts to avoid runaway failures.
- Isolate deployment jobs from optional analysis jobs.

## Observability and Operator UX
- Use workflow command annotations for actionable feedback.
- Group logs for readability and faster incident triage.
- Write job summaries for key outputs and links.
- Export metrics via API or telemetry pipeline for trend analysis.
- Alert on sustained CI degradation and flaky job rates.

## Governance and Policy
- Enforce required workflows or shared gates for baseline controls.
- Align rulesets with branch protection and release requirements.
- Require reviewers for production environment deployments.
- Align merge queue checks with merge-group workflow coverage.
- Audit permission changes and privileged workflow edits.

## Deployment Safety
- Protect production environments with explicit approvals.
- Gate deploys on immutable artifacts from trusted build jobs.
- Keep rollback paths automated and documented.
- Separate build provenance generation from deploy orchestration.
- Validate post-deploy health before marking release complete.

## Anti-Patterns to Block
- Using `permissions: write-all` at workflow scope.
- Running untrusted PR code with secrets or write tokens.
- Referencing mutable action tags for third-party actions.
- Caching every directory without hit-rate validation.
- Allowing long-lived cloud keys when OIDC is possible.

## Definition of Done
- Keep token and secret scope minimal and explicit.
- Keep trust boundaries safe across triggers and runner contexts.
- Keep workflows modular, reusable, and policy-enforced.
- Keep runtime cost and duration optimized with evidence.
- Keep observability, approvals, and rollback controls operational.

## References
- `references/github-actions-2026-02-17.md`

## Reference Index
- `rg -n "permissions|GITHUB_TOKEN|least privilege|OIDC" references/github-actions-2026-02-17.md`
- `rg -n "pull_request_target|workflow_run|untrusted" references/github-actions-2026-02-17.md`
- `rg -n "pinning|full-length SHA|allowlist|Dependabot" references/github-actions-2026-02-17.md`
- `rg -n "cache|concurrency|matrix|artifact retention|cost" references/github-actions-2026-02-17.md`
- `rg -n "reusable workflow|workflow_call|composite action" references/github-actions-2026-02-17.md`
- `rg -n "rulesets|required workflows|merge queue|environments" references/github-actions-2026-02-17.md`
- `rg -n "logs|annotations|job summary|metrics" references/github-actions-2026-02-17.md`
