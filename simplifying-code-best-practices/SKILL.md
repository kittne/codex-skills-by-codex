---
name: simplifying-code-best-practices
description: >
  Best practices for simplifying code, consolidating logic, and reducing repetition. Use when
  refactoring for clarity, applying DRY/KISS/YAGNI/Separation of Concerns, introducing reusable
  components, or reviewing code for maintainability and duplication.
---

# Simplifying Code Best Practices

## Workflow
1. Identify duplication, complexity hot spots, and confusing flows.
2. Apply DRY/KISS/YAGNI/Separation of Concerns to guide refactors.
3. Extract reusable modules or functions with clear interfaces.
4. Replace complex conditionals with patterns when appropriate.
5. Validate behavior with tests and update docs.

## Preflight (Avoid Accidental Complexity)
- Confirm what must not change (public API, performance constraints, side effects).
- Prefer small, reversible refactors with tests between steps.
- Do not DRY "similar-looking" code unless the invariants are truly shared.

## Core Principles
- DRY: Single source of truth for logic.
- KISS: Prefer the simplest solution that works.
- YAGNI: Avoid speculative features and over-generalization.
- Separation of Concerns: Keep responsibilities isolated.

## Decision Rules (Fast Defaults)
- If duplication is likely to drift: centralize it.
- If duplication is only superficial and will diverge: keep it separate (intentional WET).
- If an abstraction has only one caller and no clear contract: avoid premature generalization.
- If a conditional is growing without bound: consider a table/strategy/state machine.

## Refactoring Techniques
- Extract method/module for repeated logic.
- Replace long conditionals with Strategy or polymorphism.
- Favor composition over inheritance for flexibility.
- Reduce parameter lists by grouping related data.
- Remove dead code and collapse unnecessary layers.
- Prefer guard clauses/early returns to reduce nesting.
- Replace "boolean parameter" anti-patterns with explicit functions or enums.
- Introduce small domain types to clarify meaning (money, user id, timestamps).

## Modular Design
- Keep modules cohesive and loosely coupled.
- Build reusable utilities and shared components.
- Use clear interfaces and dependency injection where needed.

## Naming and Structure
- Use intention-revealing names.
- Keep functions small and single-purpose.
- Avoid misleading names or “magic” values.
- Centralize constants and document units/semantics (ms vs s, bytes vs chars).

## Tooling
- Use linters, formatters, and static analysis to surface complexity.
- Measure complexity and duplication (CI gates where possible).
- If available, use complexity tools (cyclomatic complexity, code coverage) to target hot spots.

## Team Practices
- Use code review checklists focused on duplication and clarity.
- Encourage refactoring as part of feature work.
- Track refactor follow-ups explicitly (issues/tickets) rather than leaving long-lived TODOs.

## References
- `references/simplifying-code.md`

## Extended Guidance
Use these checklists when the refactor touches shared code or complex business logic.

## Refactor Safety Ladder
1. Add or strengthen tests around current behavior.
2. Extract pure functions (no side effects) first.
3. Simplify conditionals and data flow next.
4. Consolidate duplication only after behavior is stable.
5. Remove dead code last.

## Smell-to-Action Map
- Long function: extract helpers with intention-revealing names.
- Deep nesting: use guard clauses and early returns.
- Repeated validation: centralize into a single validator.
- Boolean parameter flags: split into explicit functions or enums.
- Feature toggles scattered: centralize toggles and document their lifespan.

## Abstraction Checklist (Before You DRY)
- Are the invariants actually shared?
- Will both callers change together?
- Is the interface stable and easy to name?
- Does the abstraction remove more complexity than it adds?

## Validation Checklist
- Unit tests cover old and new paths.
- No behavior change in edge cases (nulls, empty lists, error paths).
- Performance does not regress on hot paths.
- Lint/format passes and new code is consistent with existing style.

## Example Refactor Template
```text
1. Identify duplicated logic in <files/paths>.
2. Extract a pure function: <function name> with a minimal interface.
3. Replace call sites and delete duplicated blocks.
4. Run tests and add a focused regression test.
```

## Common Failure Modes
- Over-generalizing a helper that only has one caller.
- Consolidating logic that is similar but not identical.
- Removing "unused" code that is actually used in dynamic paths.
- Introducing new abstractions without tests.

## Reference Index
- `rg -n "DRY|single source of truth" references/simplifying-code.md`
- `rg -n "KISS|simplicity" references/simplifying-code.md`
- `rg -n "YAGNI|premature" references/simplifying-code.md`
- `rg -n "Separation of Concerns|Single Responsibility" references/simplifying-code.md`
- `rg -n "Refactoring Patterns|Strategy|Composition" references/simplifying-code.md`

## Review Checklist (Fast)
- Is there a simpler approach that is still correct?
- Are responsibilities separated cleanly?
- Can a newcomer understand the flow in one read?

## Reference Index (Expanded)
- `rg -n "Naming|intention" references/simplifying-code.md`
- `rg -n "Tooling|linters|formatters" references/simplifying-code.md`

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
- `rg -n "Checklist|checklist" references/simplifying-code.md`
- `rg -n "Example|examples" references/simplifying-code.md`
- `rg -n "Workflow|process" references/simplifying-code.md`
- `rg -n "Pitfall|anti-pattern" references/simplifying-code.md`
- `rg -n "Testing|validation" references/simplifying-code.md`
- `rg -n "Security|risk" references/simplifying-code.md`
- `rg -n "Configuration|config" references/simplifying-code.md`
- `rg -n "Deployment|operations" references/simplifying-code.md`
- `rg -n "Troubleshoot|debug" references/simplifying-code.md`
- `rg -n "Performance|latency" references/simplifying-code.md`
- `rg -n "Reliability|availability" references/simplifying-code.md`
- `rg -n "Monitoring|metrics" references/simplifying-code.md`
- `rg -n "Error|failure" references/simplifying-code.md`
- `rg -n "Decision|tradeoff" references/simplifying-code.md`
- `rg -n "Migration|upgrade" references/simplifying-code.md`
