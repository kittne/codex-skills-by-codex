---
name: planning
description: >
  Generate a comprehensive plan for implementation or project tasks: scope definition, necessary
  research, stepwise action items with dependencies, validation steps, testing strategy, risk
  assessment, and final deliverables. Use when the user requests a development plan, architectural
  blueprint, feature rollout strategy, research plan, or step-by-step execution workflow.
---

# Planning

## Workflow
1. Gather context and constraints from the prompt and any available repo/docs.
2. Ask at most 1-2 blocking questions if planning cannot proceed responsibly.
3. Otherwise, make best-practice assumptions (state them in **Research**) and proceed.
4. Produce the plan using the required template and nothing else.

## Output Rules
- Output the plan only (no preamble or extra commentary).
- Use the exact section names from the template.
- Keep action items atomic and verb-first.
- Include validation, testing, risks, and outputs even if brief.
- Aim for ~6-12 action items unless the user asks for more/less.
- Include at least:
  - 1 validation task (acceptance criteria or review gate).
  - 1 testing task (unit/integration/e2e/manual).
  - 1 risk-mitigation or rollout/rollback task when risk is non-trivial.

## Required Output Template
```markdown
# Plan

<Brief introduction describing the goal and overall approach.>

## Scope
- **In scope:** <What the plan will cover>
- **Out of scope:** <What the plan will not cover>

## Research
- <Key research finding or assumption 1>
- <Key research finding or assumption 2>
- *(or "None needed" if no research is required.)*

## Action items
- [ ] **Task 1:** <Atomic, actionable step>
- [ ] **Task 2:** <Atomic, actionable step>
- [ ] **Task 3:** <Atomic, actionable step>
- [ ] **Task N:** <Atomic, actionable step>

## Task breakdown
| ID | Task | Depends on |
|----|------|------------|
| 1  | <Task 1 summary> | None |
| 2  | <Task 2 summary> | 1 |
| N  | <Task N summary> | <IDs or None> |

## Validation
- <How to validate success (acceptance criteria, review steps, metrics)>

## Testing
- <Unit/integration/e2e/manual testing plan>

## Risks
- <Risk: description - *Mitigation:* ...>
- <Risk: description - *Mitigation:* ...>
- *(or "None" if no significant risks.)*

## Outputs
- <Deliverable/outcome 1>
- <Deliverable/outcome 2>
```

## References
- `references/planning.md`

## Extended Guidance
Use this to improve plan quality when the scope is large or cross-functional.

## Clarification Triggers
- The goal is unclear or contradictory.
- External dependencies (legal, procurement, security review) are required.
- The system boundary is unknown (service ownership, APIs, data stores).

## Assumption Logging
- If you assume a version, environment, or owner, state it explicitly in **Research**.
- Tag assumptions as "high risk" when they could change scope or timeline.

## Plan Quality Checklist
- Tasks are atomic and verb-first.
- Dependencies are explicit and minimal.
- Testing includes at least one negative or edge-case path.
- Risks have mitigations, not just descriptions.

## Example Action Item (Format)
```text
- [ ] **Task 4:** Add audit logging for the new endpoint and verify logs in staging.
```

## Reference Index
- `rg -n "Workflow|Clarify only if needed" references/planning.md`
- `rg -n "Action items|atomic" references/planning.md`
- `rg -n "Risks|mitigation" references/planning.md`

## Example Plan (Mini)
```markdown
# Plan

Ship a small feature with clear validation and rollback.

## Scope
- **In scope:** API endpoint, DB migration, docs update
- **Out of scope:** UI redesign, infra re-architecture

## Research
- Assume PostgreSQL 14 and existing CI pipeline

## Action items
- [ ] **Task 1:** Review current endpoint usage and add metrics
- [ ] **Task 2:** Implement endpoint and add unit tests
- [ ] **Task 3:** Add migration and backfill plan
- [ ] **Task 4:** Update docs and run smoke tests

## Task breakdown
| ID | Task | Depends on |
|----|------|------------|
| 1  | Metrics | None |
| 2  | Endpoint + tests | 1 |
| 3  | Migration | 2 |
| 4  | Docs + smoke | 2 |

## Validation
- Endpoint responds within SLO and passes tests

## Testing
- Unit + integration tests for endpoint

## Risks
- Migration lock time – *Mitigation:* online migration tool

## Outputs
- Endpoint deployed and documented
```

## Risk Prompts (Use When Stuck)
- What is the rollback path?
- What is the worst-case failure mode?
- What is the operational blast radius?
- What is the data loss window?

## Reference Index (Expanded)
- `rg -n "Overview|Scope" references/planning.md`
- `rg -n "Validation|Testing" references/planning.md`
- `rg -n "Outputs|deliverables" references/planning.md`
