---
name: prompt-builder
description: >
  Guide the design and iterative testing of high-quality, language-agnostic prompts. Use when the
  user asks to write, refine, evaluate, or template prompts (system prompts, agent instructions,
  evaluation prompts, etc.).
---

# Prompt Builder

## Workflow
1. Gather requirements and success criteria (what "good" looks like).
2. Build the prompt (purpose, context, constraints, tool rules, output format).
3. Test against representative cases and edge cases.
4. Refine (Builder -> Tester loop) up to three iterations.

## Quick Intake (Ask/Confirm)
- Purpose: what task should the model perform? what is explicitly out of scope?
- Audience/tone: who will read the output and what style is required?
- Inputs: what data will be provided and how should it be delimited?
- Output format: Markdown/JSON/code/table; required sections; length constraints.
- Tools: allowed tools, when to use them, and when not to.
- Safety: any refusal/confirmation requirements (especially for destructive actions).

## Builder Mode
- Define purpose, persona, and tasks explicitly.
- Provide necessary context and delimit input data.
- Specify output format with examples.
- Include tool usage or refusal criteria if needed.

### Builder Checklist (Use as Structure)
- **Purpose:** one sentence describing the job.
- **Persona (optional):** role + tone + depth expectations.
- **Tasks:** verb-first steps; order matters when it matters.
- **Inputs:** delimit user-provided data clearly (avoid mixing instructions and data).
- **Constraints:** what must be included/excluded; max length; citations; formatting.
- **Tool rules:** triggers for search/docs/code execution; max calls; fallback if tool fails.
- **Output schema:** exact headings/JSON keys; include a minimal example if format-sensitive.
- **Edge cases:** what to do if input is missing/empty/contradictory.

## Tester Mode
- Run through success criteria and sample cases.
- Check for ambiguity or missing instructions.
- Refine wording, structure, or examples.

### Tester Checklist
- Does the prompt remove ambiguity (no "recent", "short", "high quality" without specifics)?
- Are instructions and input data separated with clear delimiters?
- Are failure modes specified (what to do when the answer is unknown)?
- Is the output format deterministic and easy to validate?
- Would this work on at least 2-3 varied example inputs (including an edge case)?

## Prompt Checklist
- Purpose and scope are clear.
- Required inputs are defined and separated.
- Output format is explicit and example-driven.
- Safety and edge cases are addressed.

## Anti-Patterns
- Vague goals or undefined output.
- Mixed instructions and input data.
- Overly negative constraints without guidance.
- Conflicting requirements.

## Review Checklist
- Prompt meets success criteria.
- Format and tone are consistent.
- Edge cases are handled.
- Prompt is concise but complete.

## Minimal Prompt Skeleton

```text
You are <persona>.

Task:
1. ...
2. ...

Constraints:
- ...

Input:
<DELIMITED USER DATA>

Output format:
- ...
```

## References
- `references/prompt-builder.md`

## Extended Guidance
Use this when prompts are used in production or evaluated at scale.

## Prompt Quality Rubric
- Clarity: tasks and constraints are unambiguous.
- Reliability: output format is deterministic and testable.
- Robustness: handles missing/contradictory inputs.
- Efficiency: minimal but sufficient instructions.

## Few-Shot Guidance
- Use few-shot only when behavior is hard to specify.
- Keep examples short and aligned with the exact output format.
- Avoid overfitting to a single example.

## Common Failure Modes
- Mixed instructions and input data in the same block.
- Vague terms like "recent" or "best" without criteria.
- Output schema not specified or inconsistent across examples.

## Reference Index
- `rg -n "Builder Checklist" references/prompt-builder.md`
- `rg -n "Tester Checklist" references/prompt-builder.md`

## Evaluation Checklist
- Does the prompt prevent ambiguous outputs?
- Are edge cases defined explicitly?
- Is the output schema unambiguous and minimal?

## Reference Index (Expanded)
- `rg -n "Anti-Patterns" references/prompt-builder.md`
- `rg -n "Minimal Prompt Skeleton" references/prompt-builder.md`

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
- `rg -n "Checklist|checklist" references/prompt-builder.md`
- `rg -n "Example|examples" references/prompt-builder.md`
- `rg -n "Workflow|process" references/prompt-builder.md`
- `rg -n "Pitfall|anti-pattern" references/prompt-builder.md`
- `rg -n "Testing|validation" references/prompt-builder.md`
- `rg -n "Security|risk" references/prompt-builder.md`
- `rg -n "Configuration|config" references/prompt-builder.md`
- `rg -n "Deployment|operations" references/prompt-builder.md`
- `rg -n "Troubleshoot|debug" references/prompt-builder.md`
- `rg -n "Performance|latency" references/prompt-builder.md`
- `rg -n "Reliability|availability" references/prompt-builder.md`
- `rg -n "Monitoring|metrics" references/prompt-builder.md`
- `rg -n "Error|failure" references/prompt-builder.md`
- `rg -n "Decision|tradeoff" references/prompt-builder.md`
- `rg -n "Migration|upgrade" references/prompt-builder.md`
