---
name: tailwind
description: >
  Implement, configure, and review Tailwind CSS usage in modern frontend codebases. Use when setting
  up Tailwind v4/v3 builds, theme tokens, responsive/state variants, component patterns,
  accessibility, performance, migration strategy, and team guardrails for maintainable utility-first UI.
---

# Tailwind

## Workflow
1. Confirm Tailwind version and build integration path (Vite, PostCSS, CLI, framework adapter).
2. Validate content scanning behavior and where classes are discovered.
3. Establish token/theming strategy before scaling component work.
4. Implement utilities first, then extract stable abstractions deliberately.
5. Validate responsive behavior, interaction variants, and accessibility defaults.
6. Measure CSS output and fix scan/safelist mistakes.
7. Apply formatting/linting guardrails for long-term maintainability.
8. Ship with migration notes if upgrading major versions.

## Preflight (Ask / Check First)
- Tailwind major version (`v3` or `v4`) and target browser support policy.
- Build toolchain (Vite, webpack, Next.js, custom PostCSS pipeline).
- Monorepo or multi-package content scanning requirements.
- Existing design tokens and whether they are centralized.
- Current pain points: missing styles, class sprawl, inconsistent dark mode, bundle size.
- Required accessibility constraints (contrast, focus indicators, reduced motion).

## Operating Principles
- Keep class names statically discoverable by the scanner.
- Prefer named tokens over repeated arbitrary values.
- Use utility-first as default; abstract only stable repeated patterns.
- Keep scan scope strict to avoid both missed classes and bloated CSS.
- Treat accessibility behavior as a first-class styling contract.
- Standardize class ordering and review rules in CI.

## Setup and Configuration
- Use official integration path for your bundler.
- For v4, prefer CSS-first setup and modern scanning directives.
- Keep project entry CSS explicit and version-controlled.
- For monorepos, include external sources intentionally.
- Document any non-default scanning behavior in workspace docs.

### Minimal Setup Checks
```bash
npm ls tailwindcss
npm run build
```

```bash
rg -n "@import \"tailwindcss\"|@tailwind" src styles app
```

## Content Scanning and Safelisting
- Keep classes literal in source; avoid dynamic string composition.
- If dynamic states are needed, map inputs to predefined class strings.
- Use controlled safelisting only for known generated classes.
- In shared UI packages, register sources explicitly.
- Exclude generated files and irrelevant directories from scanning.

### Safe Dynamic Mapping Pattern
```ts
const badgeByTone = {
  success: "bg-emerald-600 text-white",
  warning: "bg-amber-500 text-black",
  danger: "bg-rose-600 text-white",
} as const;
```

## Theming and Tokens
- Define core tokens for color, spacing, type, radius, and shadow.
- Prefer semantic naming (`primary`, `muted`, `danger`) over raw palette usage everywhere.
- Promote repeated arbitrary values into tokens.
- Keep dark mode activation strategy consistent (`prefers-color-scheme`, class, or data attribute).
- Document token lifecycle and deprecation process.

## Architecture and Component Patterns
- Keep utility composition close to markup for readability.
- Extract components when patterns repeat with stable inputs.
- Use `@apply` sparingly and only for stable, low-variance groups.
- Prefer component props/variants over giant conditional class chains.
- Keep layout concerns in containers and content styles in leaf elements.

### Extraction Triggers
- Pattern appears in 3+ places.
- Same class bundle changes together across features.
- Variant API can be expressed clearly (`size`, `intent`, `state`).

## Responsive and Stateful UI
- Follow mobile-first breakpoint strategy consistently.
- Use state variants (`hover:`, `focus-visible:`, `disabled:`) intentionally.
- Prefer `focus-visible` over suppressing outlines.
- Use container queries where component-level responsiveness matters.
- Keep motion and transition effects subtle and meaningful.

## Accessibility Guardrails
- Enforce visible focus indicators for keyboard users.
- Verify contrast ratios against token combinations.
- Provide reduced-motion alternatives where animation exists.
- Ensure semantic HTML and ARIA usage are preserved by component abstractions.
- Validate interactive controls by keyboard and screen reader checks.

## Performance and Delivery
- Confirm production output only contains used utilities.
- Keep safelist small and justified with comments in config/CSS.
- Track CSS artifact size before and after major UI changes.
- Avoid broad wildcards that pull large, unused class sets.
- Use build cache and incremental tooling where available.

### Build Verification Commands
```bash
npm run build
ls -lh dist assets .next | cat
```

```bash
rg -n "inline\(|@source|safelist|content" styles src tailwind.config.*
```

## Tooling and Team Workflow
- Enforce formatter with Tailwind class-order plugin.
- Add lint checks for dynamic class anti-patterns where possible.
- Keep PR checklist for token usage, accessibility, and bundle impact.
- Standardize component variant conventions across teams.
- Review Tailwind version upgrades in a dedicated branch.

## Migration Playbooks
- v3 to v4: validate scanning/config model changes early.
- Confirm plugin compatibility and installation method changes.
- Re-test dark mode activation and custom theme variables.
- Compare pre/post build output and runtime visual behavior.
- Roll out incrementally to reduce regression blast radius.

## Common Failure Modes
- Dynamic class interpolation causing missing production styles.
- Unbounded safelist causing large CSS bundles.
- Token drift due to repeated arbitrary values.
- Overuse of `@apply` leading to hidden coupling.
- Inconsistent dark mode trigger across app shells.
- Accessibility regressions from removed focus styles.

## Definition of Done
- Build is stable and Tailwind integration is deterministic.
- Scanning sources are explicit and validated for monorepo/workspace layout.
- Token usage is consistent and arbitrary values minimized.
- Accessibility checks pass for focus, contrast, and reduced-motion behavior.
- CSS output size is measured and acceptable.
- Migration notes are documented when version behavior changed.

## References
- `references/tailwind.md`

## Reference Index
- `rg -n "^## Project setup|^### Installation" references/tailwind.md`
- `rg -n "JIT|purge|content scanning|@source|inline" references/tailwind.md`
- `rg -n "dark mode|@theme|tokens|theme variables" references/tailwind.md`
- `rg -n "componentization|@apply|utility-first|architecture" references/tailwind.md`
- `rg -n "responsive|variants|container queries" references/tailwind.md`
- `rg -n "accessibility|focus-visible|contrast|reduced motion" references/tailwind.md`
- `rg -n "performance|build output|migration|anti-pattern" references/tailwind.md`

## Quick Questions (When Stuck)
- Is the missing style caused by scan discoverability or incorrect class logic?
- Should this repeated value become a token?
- Is this abstraction reducing complexity or hiding it?
- Can we remove classes instead of adding another variant branch?
- Did this change improve accessibility and bundle size, or only visual output?
