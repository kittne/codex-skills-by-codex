---
name: documentation-update
description: >
  Update and validate project documentation whenever code changes (new features, API endpoints,
  configuration/CLI updates, etc.) require it. Apply consistent Markdown formatting and use
  language-appropriate doc generation tools (Python, JS/TS, Go, Java, Rust, etc.) to keep docs
  accurate, buildable, and linted.
---

# Documentation Update

## Workflow
1. Identify user-facing changes and which docs are affected (README, `docs/`, API refs, config/CLI tables).
2. Locate the docs toolchain (Sphinx/MkDocs/Docusaurus/TypeDoc/rustdoc/godoc/etc.).
3. Update docs content + examples to match current behavior (names, flags, defaults, outputs).
4. Validate by running the repo's docs build/lint/link checks.
5. Quality pass: consistency, accuracy, and "copy/paste runnable" examples.
6. Summarize what changed, what was verified, and any known gaps.

## What to Update
- README and quickstart guides.
- `docs/` pages and tutorials.
- API references (OpenAPI/Swagger, SDK docs, typedoc/javadoc/rustdoc/godoc).
- Config and env var reference tables.
- Changelog/release notes (only if the repo uses them).

## How to Find the Docs Tooling
- Look for `docs/`, `mkdocs.yml`, `conf.py` (Sphinx), `docusaurus.config.*`, `typedoc.json`.
- Check `package.json` scripts for `docs:*` tasks.
- Check CI config for documentation build/lint steps.

Common build/lint commands to look for:
```bash
# Python docs
sphinx-build -b html docs docs/_build/html
mkdocs build

# JS/TS docs
npm run docs
npm run docs:build
npm run docs:lint
npx typedoc

# Go / Rust
go doc ./...
cargo doc --no-deps
```

## Validation Expectations
- Build docs if the repo has a docs build command.
- Run Markdown lint/format checks if configured.
- Verify code snippets and examples reflect current behavior.
- Prefer treating warnings as errors for doc builds when supported (to prevent bitrot).

## Quality Checklist (Use by Default)
- Completeness:
  - All new/changed user-facing behavior is documented.
  - No "TBD" placeholders or dangling TODOs in published docs.
- Accuracy:
  - Names/paths/flags/defaults match code and `--help` output.
  - Request/response examples match real payloads (and error modes are documented).
- Examples:
  - Examples are copy/paste runnable (or explicitly marked as pseudo).
  - Outputs shown match current behavior (or are clearly illustrative).
- Consistency:
  - Terminology is consistent across pages.
  - Avoid duplicated instructions that will drift; link to a canonical page instead.
- Hygiene:
  - No broken links/images.
  - Markdown renders correctly (fences, headings, tables).

## Templates (Condensed)
New feature section:
````markdown
## <Feature name>

<What it is and why it exists.>

### Usage
- <How to enable/use it>

### Example
```bash
<command>
```
````

API endpoint section:
```markdown
### <METHOD> <PATH>

**Description:** ...

**Request:** ...

**Response:** ...
```

## References
- `references/documentation.md`

## Extended Guidance
Use this when changes affect public APIs, CLIs, or operational runbooks.

## Doc Change Detection Checklist
- New/changed endpoints or CLI flags.
- Config keys added/removed.
- Behavior changes that impact defaults or error codes.
- New environment variables or secrets.

## Documentation QA Checklist
- Examples compile or run as written.
- Version numbers match the codebase.
- Links are valid and relative where possible.
- Commands use the correct shell syntax.

## Common Failure Modes
- Examples drift from actual behavior.
- Missing migration steps for breaking changes.
- Docs updated in README but not in `docs/` (or vice versa).

## Reference Index
- `rg -n "Documentation update workflow" references/documentation.md`
- `rg -n "Command-line|CLI" references/documentation.md`

## Audience Checklist
- Identify primary user type (developer, operator, end user).
- Ensure prerequisites are listed before steps.
- Keep troubleshooting concise and actionable.

## Reference Index (Expanded)
- `rg -n "Formatting|structure" references/documentation.md`

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
- `rg -n "Checklist|checklist" references/documentation.md`
- `rg -n "Example|examples" references/documentation.md`
- `rg -n "Workflow|process" references/documentation.md`
- `rg -n "Pitfall|anti-pattern" references/documentation.md`
- `rg -n "Testing|validation" references/documentation.md`
- `rg -n "Security|risk" references/documentation.md`
- `rg -n "Configuration|config" references/documentation.md`
- `rg -n "Deployment|operations" references/documentation.md`
- `rg -n "Troubleshoot|debug" references/documentation.md`
- `rg -n "Performance|latency" references/documentation.md`
- `rg -n "Reliability|availability" references/documentation.md`
- `rg -n "Monitoring|metrics" references/documentation.md`
- `rg -n "Error|failure" references/documentation.md`
- `rg -n "Decision|tradeoff" references/documentation.md`
- `rg -n "Migration|upgrade" references/documentation.md`
