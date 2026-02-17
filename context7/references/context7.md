# Context7 Reference

Use Context7 to answer library-specific questions using up-to-date official documentation.

## Required Tool Sequence
1. Resolve the library ID.
1. Query docs using that ID.

Notes:
- Prefer narrow, specific queries.
- Do not call Context7 tools more than 3 times per user question.

## Version Awareness
- If working inside a repo, determine the installed version from dependency files first.
- If the installed version is materially different from what the docs describe, call out the mismatch and adjust guidance.

## Query Tips
- Include the relevant API surface in the query (function/class names, config keys, error strings).
- Ask for examples when needed (e.g., "authentication middleware example").
- If the initial result is too broad, refine and re-query.

## Failure Modes
- If the library cannot be resolved, double-check the name and try a more general search term.
- If docs are missing or ambiguous, ask the user a targeted follow-up question and state uncertainty rather than guessing.
