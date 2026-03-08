---
name: typescript
description: >
  Build, review, and operate production TypeScript codebases with strict typing, stable module
  boundaries, modern Node.js resolution, and reliable build/test workflows. Use for tsconfig design,
  monorepo project references, API contract typing, runtime validation boundaries, refactors, and CI
  hardening.
---

# TypeScript

## Workflow
1. Confirm runtime target (Node/browser), module system, and repository layout.
2. Establish strict compiler defaults and shared `tsconfig` inheritance.
3. Separate domain types, transport types, and generated types.
4. Enforce module boundaries and import discipline in packages.
5. Align build, typecheck, lint, and test pipelines with CI.
6. Measure and tune compilation performance for monorepos.
7. Validate public API stability before release.

## Preflight (Ask / Check First)
- TypeScript version and package manager lockfile state.
- Runtime matrix (Node LTS, browser targets, ESM/CJS expectations).
- Monorepo vs single-package structure.
- Publishing requirements (internal-only vs public packages).
- Current strictness level and known `any` escape hatches.

## Compiler Baseline
- Keep `strict: true` enabled in all production packages.
- Use shared base config plus per-package overrides only when justified.
- Prefer modern Node module resolution (`node16`/`nodenext`) to match runtime behavior.
- Avoid `moduleResolution: "bundler"` with `module: "nodenext"`; keep module and resolution compatible.
- Set explicit `target` and `lib` for the deployed runtime, not developer machines.
- Keep emit strategy explicit: `noEmit` for apps checked by bundler, declaration emit for libraries.
- Use `verbatimModuleSyntax` instead of deprecated `preserveValueImports`.
- Remove deprecated `importsNotUsedAsValues`; use `verbatimModuleSyntax` and lint rules for import hygiene.
- Align with TS 5.9+ defaults for new projects (`module: "nodenext"`, `target: "esnext"`, `moduleDetection: "force"`).
- Enable `noUncheckedSideEffectImports` to catch unresolved side-effect imports early.
- Avoid deprecated resolution modes (`classic`, legacy `node10`) and stale baseUrl patterns that are being removed in future majors.

### Recommended Defaults
```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitOverride": true,
    "exactOptionalPropertyTypes": true,
    "noUncheckedIndexedAccess": true,
    "noUncheckedSideEffectImports": true,
    "module": "nodenext",
    "target": "esnext",
    "moduleDetection": "force",
    "moduleResolution": "nodenext"
  }
}
```

## Type Design and Boundaries
- Model domain types first; map transport types at edges.
- Use discriminated unions for state machines and result channels.
- Replace `any` with `unknown` plus explicit narrowing.
- Prefer explicit return types on exported functions.
- Encode invariants with branded types only when misuse risk is real.

## Module and Package Structure
- Use project references for large monorepos to constrain type graph size.
- Keep internal modules private; export stable surfaces through index barrels only when intentional.
- Avoid circular dependencies across packages.
- Use path aliases sparingly and keep runtime resolver alignment verified.
- Keep package boundaries enforceable through lint rules and import constraints.

## Runtime Contract Safety
- Validate untrusted input at process boundaries (HTTP, queues, file ingestion).
- Keep DTO schemas versioned and compatibility-tested.
- Treat generated API/client types as build artifacts, not hand-edited sources.
- Prevent silent `undefined` drift with strict optional handling.

## Build and CI Practices
- Separate `typecheck` and `build` tasks for fast feedback.
- Use incremental builds and project references to reduce CI cost.
- Fail CI on compiler warnings that signal unsafe patterns.
- Keep lockfile deterministic and avoid mixed package managers.
- Run matrix checks when shipping dual ESM/CJS libraries.

## Performance and Developer Experience
- Use incremental compilation and composite projects for large graphs.
- Profile slow type checks before restructuring package topology.
- Avoid massive union/intersection expansions in hot type paths.
- Prefer local inference over globally exported mega-generic helpers.

## Security and Operations
- Do not rely on types for runtime trust boundaries.
- Keep source maps and stack traces policy-aligned for production.
- Enforce dependency update hygiene and deprecation burn-down.
- Audit generated declarations for accidental data exposure.

## Validation Commands
```bash
pnpm exec tsc -b --pretty false
pnpm exec tsc --noEmit
pnpm exec eslint .
pnpm test
```

## Common Failure Modes
- Treating compile-time types as runtime validation.
- Mixing incompatible ESM/CJS settings across packages.
- Overusing `as` assertions to bypass model design.
- Hidden `any` spread from third-party declarations.
- Cross-package import cycles causing unstable builds.

## Definition of Done
- Strict compiler settings are enforced and documented.
- Public types are intentional, stable, and tested.
- Typecheck/build/test gates are green in CI.
- Runtime boundary validation exists for untrusted input.
- Performance hotspots and risky type escapes are addressed.

## References
- `references/typescript-2026-02-18.md`

## Reference Index
- `rg -n "strict|compilerOptions|moduleResolution" references/typescript-2026-02-18.md`
- `rg -n "project references|monorepo|incremental" references/typescript-2026-02-18.md`
- `rg -n "runtime validation|unknown|any" references/typescript-2026-02-18.md`
- `rg -n "CI|typecheck|build" references/typescript-2026-02-18.md`
