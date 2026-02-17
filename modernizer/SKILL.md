---
name: modernizer
description: >
  Plan and guide comprehensive modernization of software projects across languages and
  architectures. Use when the user requests to modernize, refactor, migrate, or update an
  application, codebase, architecture, or tech stack to modern standards. Not for unrelated feature
  development or tasks outside modernization scope.
---

# Modernizer

## Workflow
1. Analyze the current system (architecture, stack, dependencies, pain points, failure modes).
2. Document major features/modules and the required behaviors to preserve (what must not break).
3. Assess cross-cutting concerns:
   - Authn/z, config, logging/metrics/tracing, error handling, data modeling, testing.
   - Infra/CI/CD, deployment, observability, operational runbooks.
4. Checkpoint: summarize current state and confirm understanding and goals with the user.
5. Propose a target architecture and updated tech stack aligned to constraints (time, team, risk, compliance).
6. Produce a phased modernization plan with milestones, dependencies, risks, and validation gates.
7. Plan DevOps/testing improvements to support the target state (CI, IaC, release strategy, rollback).
8. Checkpoint: get user approval on scope and approach before implementation.
9. Scaffold a new structure and documentation for the target state (only when requested/approved).

## Quick Intake (Ask Early)
- What is driving modernization (security, velocity, cost, scale, maintainability)?
- Non-negotiables: timelines, budget, compliance, required uptime, migration constraints.
- What is acceptable change: API compatibility, UI changes, data model changes.
- Current pain points: outages, slow builds, brittle deploys, unowned modules.
- Deployment model: cloud/on-prem, containers, serverless, k8s, etc.

## Modernization Principles (Default Stance)
- Prefer incremental migration over rewrites.
- Preserve required behavior unless explicitly changing it.
- Use feature flags, parallel run, and backward compatibility when risk is high.
- Treat observability and testing as migration enablers, not afterthoughts.

## Deliverable Shapes

Current-state report:
- Architecture overview (components + dependencies).
- Dependency/runtime inventory (versions, EOL risks).
- Key workflows and failure modes.
- Pain points + suspected root causes.

Target-state proposal:
- Architecture diagram (textual is fine).
- Tech stack choices with rationale and trade-offs.
- Migration strategy (strangler, modular monolith, service extraction, etc.).

Phased plan:
- Phases with milestones and "exit criteria".
- Dependencies and parallelization opportunities.
- Rollout/rollback plan per phase.
- Risks with mitigations.

## Deliverables
- Current-state analysis summary.
- Target-state architecture proposal.
- Phased plan (tasks, dependencies, rollout, rollback).
- Updated docs covering setup, architecture, and feature-level changes.
- Optional scaffolded project structure for the modernized system.

## Constraints
- Preserve user-required behavior unless explicitly changing it.
- Avoid unrelated new features.
- Prefer incremental migration with feature flags/backward compatibility when possible.
- Keep changes safe: testing, observability, rollback plan.

## References
- `references/modernizer.md`

## Extended Guidance
Use this when the modernization spans multiple layers (frontend, backend, infra) or when de-risking
production changes.

## Modernization Archetypes
- Dependency refresh (security + compatibility).
- Platform upgrade (language/runtime/framework major versions).
- Architecture shift (monolith to services, service mesh, event-driven).
- Observability + reliability uplift (metrics, tracing, SLOs).
- Data layer refactor (schema changes, storage engine, caching).

## Decision Gates
- Can we isolate risk behind a feature flag or canary?
- Is there a rollback path for each major step?
- Can we ship incremental value every sprint?
- Do we need a compatibility layer or dual-write period?

## Migration Workstreams
- Code changes and refactors.
- Infrastructure and deployment changes.
- Data migrations and backfills.
- Test/quality improvements.
- Documentation and training.

## Compatibility Strategy Checklist
- Maintain backward-compatible APIs where possible.
- Introduce adapters for old interfaces when needed.
- Run dual writes/reads only for the minimum time required.
- Add migration metrics and dashboards.

## Validation and Rollback
- Define success metrics and rollback triggers before rollout.
- Test rollback procedures in staging.
- Keep an inventory of changes that are not easily reversible.

## Common Failure Modes
- Skipping baselines; no “before” metrics exist.
- Attempting a big-bang migration without canaries.
- Underestimating schema migration time or replication lag.
- Leaving compatibility layers permanently.

## Reference Index
- `rg -n "Assessment|current state" references/modernizer.md`
- `rg -n "Modernization strategies|phases" references/modernizer.md`
- `rg -n "Risk|rollback|migration" references/modernizer.md`
- `rg -n "Validation|testing" references/modernizer.md`

## Communication Plan
- Share milestones and blast-radius expectations with stakeholders.
- Publish a change log or migration guide for teams downstream.

## Deliverables Checklist
- Updated architecture diagram.
- Migration plan with rollback steps.
- Updated runbooks and monitoring dashboards.

## Reference Index (Expanded)
- `rg -n "Stakeholders|communication" references/modernizer.md`

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
- `rg -n "Checklist|checklist" references/modernizer.md`
- `rg -n "Example|examples" references/modernizer.md`
- `rg -n "Workflow|process" references/modernizer.md`
- `rg -n "Pitfall|anti-pattern" references/modernizer.md`
- `rg -n "Testing|validation" references/modernizer.md`
- `rg -n "Security|risk" references/modernizer.md`
- `rg -n "Configuration|config" references/modernizer.md`
- `rg -n "Deployment|operations" references/modernizer.md`
- `rg -n "Troubleshoot|debug" references/modernizer.md`
- `rg -n "Performance|latency" references/modernizer.md`
- `rg -n "Reliability|availability" references/modernizer.md`
- `rg -n "Monitoring|metrics" references/modernizer.md`
- `rg -n "Error|failure" references/modernizer.md`
- `rg -n "Decision|tradeoff" references/modernizer.md`
- `rg -n "Migration|upgrade" references/modernizer.md`
