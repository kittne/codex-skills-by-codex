---
name: agents-md
description: >
  Create, review, and maintain effective AGENTS.md files and adjacent agent-instruction surfaces
  for coding repositories. Use when adding or updating AGENTS.md guidance, designing nested or
  path-scoped agent instructions, auditing repo-specific build/test/safety guidance, or adapting
  agent-facing instruction files for monorepos and other multi-scope codebases.
---

# AGENTS.md

## Workflow
1. Inspect the repo's current instruction surfaces before writing anything: `AGENTS.md`, nested
   `AGENTS.md`, `.github/copilot-instructions.md`, `.github/instructions/*.instructions.md`,
   `CLAUDE.md`, and any docs that already define setup or validation commands.
2. Extract only non-inferable, repo-specific facts:
   - install, build, test, lint, and format commands
   - repo entry points and important directories
   - protected paths, approval gates, and secret-handling rules
   - fast vs slow validation expectations
   - common failure and recovery steps
3. Keep the instruction file short and command-first. Link to deeper docs instead of copying
   architecture or onboarding prose into the instruction file.
4. Scope guidance to the smallest useful boundary. Prefer nested `AGENTS.md` files when packages or
   subtrees have different commands, tooling, or safety rules.
5. Write concrete, checkable rules. State when to ask before running expensive, destructive, or
   networked actions.
6. Validate the documented commands, links, and formatting when feasible.

## Authoring Rules
- Put exact copy/pasteable commands near the top.
- Prefer minimal sections such as quickstart commands, repo map, work policy, validation contract,
  and failure/recovery notes.
- Include security boundaries that matter in practice: secrets handling, protected files,
  permission limits, sandbox or network assumptions, and audit/log locations if the repo has them.
- Use examples and short templates only when they reduce ambiguity.
- Avoid generic coding advice, motivational prose, sensitive internal details, and long summaries
  of facts the agent can learn by reading the repo.
- Treat AGENTS.md as context, not enforcement. Favor rules that humans can review and CI can check.

## Scoping Guidance
- Use one repo-root `AGENTS.md` for universal instructions.
- Add nested `AGENTS.md` files when subprojects need different commands or policies.
- Use adjacent instruction formats only when the target tool requires them; keep the source of truth
  as small and consistent as possible.
- If the repo already has multiple instruction systems, avoid duplicating the same rule in every
  file. Point to the canonical source where possible.

## Done Criteria
- The file starts with the commands the agent will actually need.
- The guidance captures repo-specific constraints that are not obvious from the code alone.
- Expensive or risky actions have explicit approval or escalation guidance.
- Templates or examples are customized for the repo instead of copied verbatim.
- The final document is concise enough to load frequently without wasting context budget.

## References
- `references/agents_md_best_practices.md`

## Reference Index
- `rg -n "^### What the empirical evidence implies|^### A small chart" references/agents_md_best_practices.md`
- `rg -n "^### Recommended semantic sections|^### Workflow design" references/agents_md_best_practices.md`
- `rg -n "^### How major tools load and scope|^### Comparison table" references/agents_md_best_practices.md`
- `rg -n "^### What to put in AGENTS.md for security|^### Privacy and data retention" references/agents_md_best_practices.md`
- `rg -n "^### Three AGENTS.md templates|^#### Minimal template|^#### Standard template|^#### Advanced template" references/agents_md_best_practices.md`
