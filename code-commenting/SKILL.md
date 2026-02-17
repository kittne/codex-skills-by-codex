---
name: code-commenting
description: >
  Provides guidance for writing and reviewing high-quality code comments in any programming language.
  Use whenever code comments are added, updated, or reviewed. Prioritize minimal, accurate comments
  that explain intent ("why") over implementation ("what"), follow language doc-comment standards,
  and avoid stale comments, commented-out code, and secrets.
---

# Code Commenting

## Workflow
1. Improve code clarity first (naming, structure, refactors) before adding comments.
2. Decide whether a comment is needed using the "Comment Decision Rules" below.
3. Add comments that explain intent, constraints, and non-obvious trade-offs (the "why").
4. Use standard doc-comment formats for public APIs and important interfaces.
5. Review for staleness, redundancy, and risk (commented-out code, secrets, misleading claims).
6. Keep comments maintained: update or remove when code changes.

## Principles
- Prefer self-documenting code: comments are not a substitute for clarity.
- Comment the "why" (intent, constraints, context), not the "what".
- Treat comments as code: they need review, tests (when applicable), and maintenance.

## Comment Decision Rules

Add a comment when it answers a question a reviewer would reasonably ask:
- "Why is it done this way (and not the obvious way)?"
- "What constraint/invariant must remain true?"
- "What external behavior/spec/compat quirk is being respected?"
- "What breaks if this changes?"

Prefer refactoring over commenting when:
- The code is unclear due to naming/structure (rename/simplify first).
- The comment would be long because the code is tangled (extract helpers, reduce branching).

Avoid comments that will rot quickly:
- Restating the code.
- Describing temporary states without a tracking reference.
- Explaining basic language syntax.

## When to Comment
- Non-obvious constraints: performance trade-offs, invariants, ordering requirements.
- Workarounds: explain why the workaround exists and when it can be removed.
- External interfaces: protocol quirks, compatibility notes, upstream bugs.
- Public APIs: behavior, inputs/outputs, side effects, error modes, usage examples.

## When Not to Comment
- Restating the code.
- Explaining obvious language features.
- Compensating for poor naming (rename instead).
- Leaving blocks of commented-out code (use git history).

## Doc Comments
- Use the language's standard style (docstrings, JSDoc, Javadoc, rustdoc, godoc).
- Document behavior and contracts, not line-by-line implementation.

Include (when relevant):
- Inputs/outputs and units (bytes vs chars, ms vs seconds, UTC vs local).
- Side effects (I/O, DB writes, network calls).
- Error modes (exceptions, return codes, partial failures).
- Concurrency/thread-safety and reentrancy assumptions.

Quick templates:

Python (docstring):
```python
def fn(arg: str) -> int:
    \"\"\"Do X for reason Y.

    Args:
        arg: What it is and constraints.

    Returns:
        What it returns and invariants.

    Raises:
        ValueError: When inputs violate constraints.
    \"\"\"
```

TypeScript (JSDoc):
```ts
/**
 * Does X to satisfy constraint Y.
 * @param arg What it is and constraints.
 * @returns What it returns.
 */
export function fn(arg: string): number {}
```

Go (godoc):
```go
// Fn does X to satisfy constraint Y.
// It returns Z. It is safe for concurrent use by multiple goroutines.
func Fn(arg string) (int, error) { return 0, nil }
```

## Comment Style (Consistency Rules)
- Write in clear, concise English (unless the repo uses another shared language).
- Prefer full sentences for non-trivial comments.
- Use consistent tags: `NOTE:`, `TODO:`, `FIXME:`, `SECURITY:`, `PERF:`.
- For TODO/FIXME, include a tracking reference when possible (`TODO(#1234): ...`).
- Avoid embedding authorship/dates in comments (git history already tracks this).

## Review Checklist
- Comments explain intent/constraints rather than repeating code.
- Comments match current behavior (no contradictions).
- No secrets, tokens, credentials, or sensitive data in comments.
- No large commented-out blocks or TODOs without a tracking reference.
- Public APIs have doc comments that describe behavior and error modes.
- Complex primitives are explained: regexes, algorithms, protocol handling, magic constants.

## References
- `references/code-commenting.md`

## Extended Guidance
Use this when commenting high-risk logic or public APIs.

## Comment Decision Rules
- Prefer refactoring over comments for unclear code.
- Add comments for invariants, safety boundaries, or non-obvious constraints.
- Document "why" and "what could go wrong" rather than restating code.

## Doc Comment Templates (Minimal)
```text
// <FuncName> <verb> <noun> to <purpose> (why it exists).
// Invariants: <what must stay true>.
// Errors: <how callers should handle failures>.
```

## Common Failure Modes
- Comments that drift from code after refactors.
- Comments that reveal secrets or internal hostnames.
- Duplicated comments across similar functions.

## Reference Index
- `rg -n "Doc comment|docstring" references/code-commenting.md`
- `rg -n "Anti-patterns" references/code-commenting.md`

## Maintenance Checklist
- Remove comments that no longer match code behavior.
- Update examples when public APIs change.

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
- `rg -n "Checklist|checklist" references/code-commenting.md`
- `rg -n "Example|examples" references/code-commenting.md`
- `rg -n "Workflow|process" references/code-commenting.md`
- `rg -n "Pitfall|anti-pattern" references/code-commenting.md`
- `rg -n "Testing|validation" references/code-commenting.md`
- `rg -n "Security|risk" references/code-commenting.md`
- `rg -n "Configuration|config" references/code-commenting.md`
- `rg -n "Deployment|operations" references/code-commenting.md`
- `rg -n "Troubleshoot|debug" references/code-commenting.md`
- `rg -n "Performance|latency" references/code-commenting.md`
- `rg -n "Reliability|availability" references/code-commenting.md`
- `rg -n "Monitoring|metrics" references/code-commenting.md`
- `rg -n "Error|failure" references/code-commenting.md`
- `rg -n "Decision|tradeoff" references/code-commenting.md`
- `rg -n "Migration|upgrade" references/code-commenting.md`
