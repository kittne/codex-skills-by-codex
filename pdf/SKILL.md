---
name: pdf
description: Use when tasks involve reading, creating, or reviewing PDF files where rendering and layout matter; prefer visual checks by rendering pages (Poppler) and use Python tools such as `reportlab`, `pdfplumber`, and `pypdf` for generation and extraction.
---

# PDF Skill

Use this skill when PDF layout, typography, or page fidelity matters. Always verify by rendering pages, not just by extracting text.

## Goals
- Produce accurate PDF output with predictable layout.
- Validate visuals via rendered images.
- Keep intermediate files organized and disposable.

## Workflow
1. Confirm the task: read, extract, modify, or generate.
2. Identify tooling needs (rendering vs extraction vs generation).
3. Render pages for visual inspection.
4. Apply changes or generate the document.
5. Re-render and verify layout, pagination, and assets.

## Tooling Defaults
- Rendering: `pdftoppm` from Poppler.
- Generation: `reportlab` for new PDFs.
- Extraction: `pdfplumber` for text and tables; `pypdf` for metadata/page ops.

## Rendering Commands
```bash
pdftoppm -png input.pdf output_prefix
```
- Inspect the generated PNGs for margins, text flow, and images.
- Render again after each meaningful change.

## Python Dependencies
Prefer `uv` if available:
```bash
uv pip install reportlab pdfplumber pypdf
```
Fallback:
```bash
python3 -m pip install reportlab pdfplumber pypdf
```

## File Layout Conventions
- Use `tmp/pdfs/` for intermediate outputs.
- Use `output/pdf/` for final deliverables in this repo.
- Keep filenames stable and descriptive.

## Extraction Guidance
- Treat extracted text as lossy.
- Use extraction for content checks, not layout validation.
- For tables, verify manually against rendered pages.

## Generation Guidance
- Use consistent font sizes and margins.
- Keep headers/footers aligned and stable across pages.
- Embed images at correct resolution and aspect ratio.

## Modification Guidance
- For small edits, prefer regenerating the PDF over direct object edits.
- If editing existing PDFs, use `pypdf` for page merges, splits, and metadata updates.
- Re-render pages after any modification to confirm layout integrity.

## Metadata and Accessibility
- Preserve title/author metadata when required.
- If generating reports, ensure headings and logical order are consistent.

## Page Operations
- Use `pypdf` for splitting, merging, or reordering pages.
- Verify page order after any structural change.

## Visual QA Checklist
- No clipped or overlapping text.
- Tables are aligned and readable.
- Images are sharp and correctly placed.
- Page numbers and headers/footers are correct.
- Section transitions look intentional.

## When Layout Is Critical
- Use a visual diff if available.
- Compare before/after page images side by side.
- Call out any layout drift or pagination changes.

## Common Failure Modes
- Text extraction passes but layout is broken.
- Font substitution changes line breaks.
- Margins differ between pages.
- Images are low resolution or incorrectly scaled.
- Page ordering changes during merge/split operations.

## Output Expectations
- Provide the final PDF path.
- Summarize visual checks performed.
- Call out any known limitations.
- Note if any dependencies were missing.

## Definition of Done
- Rendered pages look correct at 100% zoom.
- All required content is present and legible.
- Intermediate artifacts are cleaned or left in `tmp/pdfs/`.
