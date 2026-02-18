---
name: markdown
description: >
  Enforce consistent, high-quality Markdown formatting and structure across documentation: headings,
  lists, code blocks, links, images, tables, front matter metadata, line length, and formatting
  hygiene.
---

# Markdown

## Workflow
1. Outline the heading structure (one clear H1, then H2/H3 hierarchy).
2. Write content with consistent lists and fenced code blocks.
3. Add links/images/tables using standard Markdown (optimize for raw diff readability).
4. Run any repo-specific format/lint/build steps for docs and preview the rendering.
5. Do a final hygiene pass (spacing, trailing whitespace, broken links/images).

## Headings
- Use a single `#` heading per page.
- Do not skip heading levels (avoid jumping from `##` to `####`).
- Keep headings short and descriptive.

## Lists
- Use `-` for unordered lists consistently.
- For ordered lists:
  - Prefer explicit numbering (`1. 2. 3.`) when steps are procedural.
  - Using `1.` for all items is acceptable when the list is likely to change frequently.
- Avoid deep nesting; split into multiple lists/sections if nesting gets hard to read.

## Code
- Use fenced code blocks and include a language when helpful.

Example:
```bash
rg "TODO" -n
```

- Use inline code for filenames, flags, and identifiers: `--help`, `pyproject.toml`, `src/app.py`.
- Prefer copy/paste-friendly CLI blocks (avoid `$` prompts unless distinguishing command vs output).

## Links and Images
- Use descriptive link text (avoid "click here").
- Prefer relative links for repo-local docs.
- Always include meaningful alt text for images.
- Avoid hotlinking external images when possible.

## Tables
- Use tables for small, structured data (configs, option matrices).
- Keep tables narrow; long prose belongs in paragraphs.
- Use reference-style links inside tables if URLs are long.

## Front Matter
- If the repo uses front matter, keep it valid YAML and minimal.
- Do not invent new metadata keys unless the docs system requires them.
- If front matter provides the page title, do not add an additional H1 in the body.

## Formatting Hygiene
- Wrap long prose at a consistent line length if the repo prefers it.
- Avoid trailing whitespace.
- Keep blank lines between paragraphs and around code blocks.
- Ensure headings have blank lines above and below.
- Avoid raw HTML unless the target renderer requires it.

## Validation And Troubleshooting
- Preview in the target renderer (GitHub, docs site dev server) to catch:
  - Broken code fences and list indentation issues.
  - Broken links/images and malformed tables.
- If a Markdown linter exists, follow it:
  - Look for `.markdownlint.*`, `remark`, `prettier`, or `docs:*` scripts.

## References
- `references/markdown.md`

## Extended Guidance
Use these rules when updating shared docs, READMEs, or public-facing documentation.

## Structure Checklist
- Single H1 per file; use H2/H3 for hierarchy.
- Keep headings short and consistent across files.
- Use parallel wording for sibling headings.
- Avoid heading jumps (H2 -> H4).

## Tables and Code Blocks
- Use tables for small, structured datasets only.
- Align code fences with an explicit info string (language).
- Keep code blocks short and focused.
- Avoid wrapping long command lines; prefer one command per line.

## Links and References
- Prefer relative links to repo paths.
- Keep link text descriptive (avoid "here").
- Avoid raw URLs unless explicitly required.

## Formatting Hygiene
- Use consistent list punctuation and spacing.
- Keep line length reasonable (wrap paragraphs when they exceed ~120 chars).
- Avoid trailing whitespace.

## Validation Commands
```bash
rg -n "TODO|TBD|FIXME" .
```

## Common Failure Modes
- Overusing nested lists that become hard to scan.
- Using inconsistent heading capitalization.
- Mixing prose and commands without code fences.
- Copying content without updating version numbers.

## Reference Index
- `rg -n "Headings|structure" references/markdown.md`
- `rg -n "Lists|ordered lists" references/markdown.md`
- `rg -n "Code blocks|fences" references/markdown.md`
- `rg -n "Tables|alignment" references/markdown.md`

## Example Style Snippet
```markdown
## Setup
1. Install dependencies.
2. Run `npm test`.

### Notes
- Use `--verbose` for detailed logs.
```

## Reference Index (Expanded)
- `rg -n "Links|references" references/markdown.md`
- `rg -n "Formatting|whitespace" references/markdown.md`

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
- `rg -n "Checklist|checklist" references/markdown.md`
- `rg -n "Example|examples" references/markdown.md`
- `rg -n "Workflow|process" references/markdown.md`
- `rg -n "Pitfall|anti-pattern" references/markdown.md`
- `rg -n "Testing|validation" references/markdown.md`
- `rg -n "Security|risk" references/markdown.md`
- `rg -n "Configuration|config" references/markdown.md`
- `rg -n "Deployment|operations" references/markdown.md`
- `rg -n "Troubleshoot|debug" references/markdown.md`
- `rg -n "Performance|latency" references/markdown.md`
- `rg -n "Reliability|availability" references/markdown.md`
- `rg -n "Monitoring|metrics" references/markdown.md`
- `rg -n "Error|failure" references/markdown.md`
- `rg -n "Decision|tradeoff" references/markdown.md`
- `rg -n "Migration|upgrade" references/markdown.md`
