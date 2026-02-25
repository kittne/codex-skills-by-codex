---
name: github
description: >
  Apply GitHub best practices across repositories, pull requests, releases, CI/CD, security, and
  collaboration governance. Use when designing repo standards, enforcing review rules, managing
  branch and release policy, automating workflows, hardening supply chain controls, and improving
  onboarding and operational metrics.
---

# GitHub

## Workflow
1. Confirm organization constraints (compliance, branch policy, release cadence, risk tolerance).
2. Standardize repository structure and ownership conventions.
3. Define branching, release, and tagging strategy.
4. Enforce pull request quality gates and review governance.
5. Implement CI/CD workflows with deterministic checks.
6. Enable dependency, secret, and code security protections.
7. Configure access control, audit visibility, and incident response paths.
8. Track developer-experience metrics and iterate on process.

## Preflight (Ask / Check First)
- Repository model: mono-repo, multi-repo, or hybrid.
- Required checks for merge and release readiness.
- Branch model (`trunk-based`, `GitHub Flow`, release branches).
- Compliance requirements (signed commits, review count, audit retention).
- Automation maturity (Actions runners, caching, artifact strategy).
- Existing pain points: PR cycle time, flaky CI, security alert debt.

## Operating Principles
- Keep policy explicit in repository settings and versioned config files.
- Optimize for small, reviewable PRs with clear ownership.
- Treat CI as product-critical infrastructure.
- Shift security left with automated scanning and dependency hygiene.
- Keep onboarding reproducible with templates and docs-as-code.
- Use measurable outcomes, not process theater.

## Repository Design and Standardization
- Standardize README, CONTRIBUTING, and issue/PR templates.
- Use CODEOWNERS for path-based ownership and review routing.
- Keep labels and milestones consistent across repositories.
- Prefer rulesets to enforce protections for default and release branches.
- Document conventions for commit messages and changelog entries.

### Repository Contract Checklist
- `.github/CODEOWNERS`
- `.github/pull_request_template.md`
- `.github/ISSUE_TEMPLATE/`
- `.github/workflows/` with required checks
- `SECURITY.md` and escalation channel

## Branching, Releases, and Versioning
- Choose one branch model and apply it consistently.
- Protect release branches from direct pushes.
- Use semantic versioning policy with explicit exceptions.
- Create annotated tags for releases.
- Gate production releases through signed artifacts and required checks.

### Release Commands
```bash
git tag -a v1.4.0 -m "Release v1.4.0"
git push origin v1.4.0
```

```bash
gh release create v1.4.0 --generate-notes
```

## Pull Request and Review Governance
- Require descriptive PR summaries, risk notes, and test evidence.
- Enforce required reviewers and CODEOWNERS where applicable.
- Block merge on failing required checks.
- Use draft PRs for early feedback on larger changes.
- Keep review SLAs realistic and visible.

### PR Quality Checklist
- Problem statement and scope are explicit.
- Tests added/updated for behavior changes.
- Rollback or mitigation path documented.
- Security implications reviewed when applicable.
- Docs/config updates included when behavior changes.

## Automation with GitHub Actions
- Keep workflows small, composable, and reusable.
- Cache dependencies with strict keys and fallback strategy.
- Separate fast feedback jobs from slow, heavyweight jobs.
- Upload artifacts needed for debugging and release provenance.
- Use environment protections for deployment jobs.

### Actions Guardrails
- Pin action versions by major or full SHA for critical pipelines.
- Prefer OIDC for cloud auth over long-lived secrets.
- Use concurrency controls to prevent conflicting deploys.
- Set job timeouts and fail-fast defaults where sensible.

## Dependency and Supply Chain Management
- Keep lockfiles committed and deterministic.
- Enable dependency update automation with review controls.
- Triage Dependabot alerts by exploitability and blast radius.
- Generate SBOM/provenance artifacts where policy requires.
- Enforce minimal maintainer permissions on publish paths.

## Security, Access Control, and Compliance
- Enforce MFA for organization members.
- Apply least-privilege role assignments and team scopes.
- Enable code scanning, secret scanning, and dependency scanning.
- Audit privileged actions and repository setting changes.
- Define incident runbooks for compromised token or branch events.

### Security Baseline
- Branch protection with required reviews.
- Required status checks for merge.
- Push protection and secret scanning enabled.
- Scoped PAT usage minimized; prefer GitHub App or OIDC.

## Metrics and Onboarding
- Track PR cycle time, review latency, and CI success rate.
- Track deployment frequency and change failure rate.
- Measure onboarding time to first quality PR.
- Monitor alert debt (security, dependency, flaky checks).
- Review monthly and refine process where bottlenecks are visible.

## Collaboration Patterns
- Use Discussions for long-lived decisions and RFC threads.
- Use issue forms for incident and bug report consistency.
- Maintain contribution guides for local setup and validation.
- Keep ownership maps current as teams evolve.

## Common Failure Modes
- Inconsistent branch protections across repositories.
- CI pipelines that are slow, flaky, or non-deterministic.
- CODEOWNERS defined but not enforced in required reviews.
- Dependency updates merged without compatibility testing.
- Security alerts piling up without risk-based triage.
- Onboarding friction due to missing templates and environment docs.

## Definition of Done
- Repository contract and ownership model are in place.
- Branch/release policy is enforced by settings and automation.
- PR quality gates and required checks are reliable.
- Security scanning and access controls are active and reviewed.
- Team metrics are visible and used for iterative improvements.

## References
- `references/github.md`

## Reference Index
- `rg -n "Repository design|standardization|CODEOWNERS" references/github.md`
- `rg -n "Branching|releases|versioning|tag" references/github.md`
- `rg -n "Pull request|review|governance" references/github.md`
- `rg -n "Automation|CI/CD|GitHub Actions" references/github.md`
- `rg -n "dependency|supply chain|Dependabot" references/github.md`
- `rg -n "Security|access control|compliance" references/github.md`
- `rg -n "metrics|monitoring|onboarding" references/github.md`

## Quick Questions (When Stuck)
- What policy can be automated instead of documented-only?
- Which check is truly required versus just traditional?
- Is this release strategy improving safety without blocking delivery?
- Are permissions broader than needed for this workflow?
- What metric will show this process change actually worked?
