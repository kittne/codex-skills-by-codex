---
name: context7
description: >
  Provides up-to-date documentation and version guidance for external libraries. Use when working
  with any third-party library (JS, Python, Ruby, Go, .NET, etc.), when the user asks about a
  library-specific API or best practice, when debugging a dependency issue, or when installing or
  upgrading a dependency. Prefer this over guessing or relying on stale knowledge.
---

# Context7

## Workflow
1. Identify the library/framework involved (and the language/ecosystem).
2. Determine the installed version from the repo when possible.
3. Resolve the Context7 library ID.
4. Query Context7 docs for the user's exact question (narrow, API-surface specific).
5. Answer using doc-backed facts and note any version-specific caveats.

## Version Detection (Repo-Local)
- JavaScript/Node: check `package.json`, `package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`.
- Python: check `pyproject.toml`, `requirements*.txt`, `poetry.lock`, `Pipfile.lock`.
- Go: check `go.mod`.
- Rust: check `Cargo.toml` / `Cargo.lock`.
- Ruby: check `Gemfile.lock`.
- .NET: check `*.csproj` / `packages.lock.json`.

If the version cannot be determined, assume the user means the latest docs and phrase the answer as such.

## Tool Usage (MCP)
- Always call `resolve-library-id` first to get the exact library ID.
- Then call `query-docs` with a specific, narrow query.
- Do not call Context7 tools more than 3 times per user question.

## Example Query Pattern

1. Resolve (call `resolve-library-id`):
```yaml
libraryName: "<package/framework name>"
query: "<what the user is trying to do + API names + error strings>"
```

2. Query docs (call `query-docs`):
```yaml
libraryId: "</org/project[/version]>"
query: "<precise question with config keys / function names / expected behavior>"
```

## Response Rules
- Prefer official docs and primary references returned by Context7.
- If docs are ambiguous or missing, ask a targeted follow-up question or state uncertainty.
- If the user's installed version differs materially from the docs, call it out and suggest the relevant migration/upgrade notes.

## References
- `references/context7.md`

## Extended Guidance
Use this section for higher-fidelity Context7 usage when the question is ambiguous, version-sensitive,
or likely to change.

## Decision Rules (When to Query)
- Query Context7 if the user names a library, framework, or API surface.
- Query Context7 if the library version is unknown but likely to matter.
- Query Context7 before suggesting configuration flags or function signatures.
- Do not guess on breaking changes; confirm against docs.

## Version Resolution Tips
- Prefer lockfiles over manifests (lockfiles reflect the resolved version).
- If multiple versions exist (monorepo), use the one in the target package.
- If a toolchain is involved (Next.js, Django), check for related config files too.

## Query Construction Examples
```text
Query: "How do I configure CORS in FastAPI?"
Better: "FastAPI CORS middleware example with allowed origins and credentials"
```

```text
Query: "React state update in a loop"
Better: "React setState functional update in loop, avoid stale state"
```

```text
Query: "Prisma transaction"
Better: "Prisma $transaction interactive example with timeout and rollback behavior"
```

## Failure Modes and Fixes
- Library not found:
  - Try the official repo name (e.g., `vercel/next.js`).
  - Try the package name on the registry (e.g., `@nestjs/core`).
- Docs too broad:
  - Narrow with function names, config keys, error messages.
- Docs conflict with local version:
  - Call out the mismatch and suggest checking migration guides.

## Response Quality Checklist
- Cite the specific API/config key from docs.
- Note version-specific caveats if the project version is known.
- Provide a minimal working example or code snippet if useful.
- If uncertain, ask a single targeted question rather than guessing.

## Reference Index
- `rg -n "Required Tool Sequence|Resolve the library ID" references/context7.md`
- `rg -n "Version Awareness|installed version" references/context7.md`
- `rg -n "Query Tips|Include the relevant API surface" references/context7.md`
- `rg -n "Failure Modes|cannot be resolved" references/context7.md`

## Version Detection Commands (Examples)
```bash
rg -n "\"dependencies\"|\"devDependencies\"" package.json
rg -n "go\\s+\\d+\\.\\d+" go.mod
rg -n "django|fastapi|flask" requirements*.txt
```

## Query Refinement Steps
1. Start with the exact API or error string.
2. Add the config keys or flags involved.
3. Narrow to the specific environment (browser/node/server).
4. Re-query if results are too general.

## Output Consistency Checklist
- Note the library version you used (or that it was unknown).
- Include at least one verified example or usage pattern.
- Call out any breaking changes or deprecations.

## Reference Index (Expanded)
- `rg -n "Resolve the library ID|query docs" references/context7.md`
- `rg -n "Version Awareness|installed" references/context7.md`
- `rg -n "Failure Modes|ambiguous" references/context7.md`

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
- `rg -n "Checklist|checklist" references/context7.md`
- `rg -n "Example|examples" references/context7.md`
- `rg -n "Workflow|process" references/context7.md`
- `rg -n "Pitfall|anti-pattern" references/context7.md`
- `rg -n "Testing|validation" references/context7.md`
- `rg -n "Security|risk" references/context7.md`
- `rg -n "Configuration|config" references/context7.md`
- `rg -n "Deployment|operations" references/context7.md`
- `rg -n "Troubleshoot|debug" references/context7.md`
- `rg -n "Performance|latency" references/context7.md`
- `rg -n "Reliability|availability" references/context7.md`
- `rg -n "Monitoring|metrics" references/context7.md`
- `rg -n "Error|failure" references/context7.md`
- `rg -n "Decision|tradeoff" references/context7.md`
- `rg -n "Migration|upgrade" references/context7.md`
