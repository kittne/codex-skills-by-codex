# TypeScript Reference (2026-02-18)

## Context7 Sources
- Library: `/microsoft/typescript`
- Node target mapping: <https://github.com/microsoft/typescript/wiki/Node-Target-Mapping>
- Project references performance notes: <https://github.com/microsoft/typescript/wiki/Performance>
- Compiler option examples from TypeScript repo baselines.

## Practical Guidance
- Keep `strict: true` as baseline.
- Align `target`/`lib` with deployed Node version.
- Use `moduleResolution: nodenext` (or node16) for modern Node compatibility.
- Adopt project references for large monorepos to constrain compiler graph.
- Separate `tsc --noEmit` typecheck from bundler/transpiler execution.

## Suggested Baseline Config
```json
{
  "compilerOptions": {
    "strict": true,
    "moduleResolution": "nodenext",
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  }
}
```

## Version Notes
- Context7 result includes TypeScript versions `v5.9.2` and `v5.8.3`.
- Node mapping guidance references runtime-specific `target/lib/module` selections.

## Source URLs
- <https://github.com/microsoft/typescript/wiki/Node-Target-Mapping>
- <https://github.com/microsoft/typescript/wiki/Performance>
