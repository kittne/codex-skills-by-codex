---
name: beast-mode
description: >
  "Beast Mode" is an instruction-only skill for autonomous, assumption-led problem solving.
  Triggered implicitly by user prompts referencing "beast mode", "thinking beast", or
  "GPT-5 beast mode". Proceed end-to-end without further user input by making best-practice
  assumptions, planning with a Markdown checklist, validating at each step (tests/linters), and
  closing with edge cases, docs updates, and explicit assumption/risk callouts.
---

# Beast Mode

## Core Directives
- Own the task end-to-end without pausing for additional user confirmation.
- Make reasonable, best-practice assumptions when details are missing and record them.
- Validate continuously (tests, linters, type checks, manual verification) and iterate on failures.
- Sweep for edge cases and regressions before declaring done.
- Update docs/comments when behavior or usage changes.

## Workflow
1. Identify the goal, constraints, and the "definition of done" (tests green, feature works, docs updated, etc.).
2. Gather context until execution is unblocked (repo scan, configs, logs, failing commands, minimal repro).
3. State assumptions explicitly and choose safe defaults for missing details.
4. Produce a live Markdown checklist plan (see template below).
5. Execute checklist items sequentially and update the checklist as you go.
6. Validate after every meaningful change; treat failures as new checklist items and fix immediately.
7. Do a final sweep:
   - Edge cases and failure paths.
   - Re-run full validation (tests/lints/build) if available.
   - Docs/comments updated if behavior or usage changed.
8. Close with:
   - What changed.
   - What was validated and how.
   - Assumptions and remaining risks/unknowns.

## Operating Rules (Do Not Skip)
- Do not stop early: keep going until the end state is truly shippable.
- Do not ask for mid-task confirmation unless:
  - The action is destructive/irreversible in the user's context, or
  - The missing detail materially changes the solution and there is no safe default.
- If blocked by missing info, prefer:
  - Inspecting the repo for the answer, or
  - Making a best-practice assumption and recording it.

## Checklist Template (Use This Shape)

```markdown
- [ ] Reproduce / confirm the issue (or restate the exact goal)
- [ ] Gather context (files, configs, logs, versions)
- [ ] Identify root cause / constraints
- [ ] Implement fix/change (smallest safe increment)
- [ ] Add/adjust tests (regression coverage)
- [ ] Run validation (tests/lint/typecheck/build)
- [ ] Handle edge cases and failure paths
- [ ] Update docs/comments if needed
- [ ] Final verification and summary (assumptions/risks)
```

## Validation Defaults
- Prefer running the repo's existing checks over inventing new ones.
- If no checks exist, add the smallest validation possible (a focused test, a smoke command, or a minimal script) and remove it if it is only temporary scaffolding.
- Record exact commands run and outcomes in the final summary.

## Assumptions And Risks (Always Report)
- Assumptions: anything you decided without explicit user input (versions, platform, expected behavior).
- Risks: what might still be wrong, what could not be validated, and what would falsify your conclusion.

## Checklist Format
- Use a live checklist and update it as items complete:

```markdown
- [ ] Step 1: ...
- [ ] Step 2: ...
- [ ] Step 3: ...
```

## Final Output Expectations
- All checklist items checked.
- Clear notes on:
- Assumptions made.
- Risks/limitations.
- What was validated and how.

## References
- `references/beast-mode.md`

## Extended Guidance
Use this when the task is large, under-specified, or involves multiple subsystems.

## Assumption Log Template
```text
Assumption: <what I'm assuming>
Reason: <why it's reasonable>
Risk: <what could be wrong>
Mitigation: <how to verify or reduce risk>
```

## Validation Ladder (Preferred Order)
1. Unit tests (fast feedback).
2. Integration tests (real interfaces).
3. End-to-end tests or manual smoke checks.
4. Production-safe checks (metrics, logs, health probes).

## Stop Conditions
- All acceptance criteria are met.
- Tests and lint checks pass.
- Docs updates are complete.
- Risks and assumptions are explicitly listed.

## Common Failure Modes
- Making changes without a measurable baseline.
- Stacking multiple refactors without checkpoints.
- Forgetting to validate in the environment where it will run.

## Reference Index
- `rg -n "assumptions|risk" references/beast-mode.md`
- `rg -n "validation|tests" references/beast-mode.md`

## Unknowns Checklist
- Identify missing inputs and make explicit assumptions.
- Note any dependencies on external systems or versions.
- Call out any safety or data-loss risks.

## Output Expectations
- Provide a concise summary and explicit next steps.
- List tests run or note if tests were skipped.

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
- `rg -n "Checklist|checklist" references/beast-mode.md`
- `rg -n "Example|examples" references/beast-mode.md`
- `rg -n "Workflow|process" references/beast-mode.md`
- `rg -n "Pitfall|anti-pattern" references/beast-mode.md`
- `rg -n "Testing|validation" references/beast-mode.md`
- `rg -n "Security|risk" references/beast-mode.md`
- `rg -n "Configuration|config" references/beast-mode.md`
- `rg -n "Deployment|operations" references/beast-mode.md`
- `rg -n "Troubleshoot|debug" references/beast-mode.md`
- `rg -n "Performance|latency" references/beast-mode.md`
- `rg -n "Reliability|availability" references/beast-mode.md`
- `rg -n "Monitoring|metrics" references/beast-mode.md`
- `rg -n "Error|failure" references/beast-mode.md`
- `rg -n "Decision|tradeoff" references/beast-mode.md`
- `rg -n "Migration|upgrade" references/beast-mode.md`
