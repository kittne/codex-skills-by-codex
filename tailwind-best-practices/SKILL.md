---
name: tailwind-best-practices
description: >
  Best practices for using Tailwind CSS in modern frontend codebases. Use when implementing,
  configuring, or reviewing Tailwind usage (utility classes, responsive and state variants, theming,
  component abstraction, accessibility, and build performance) across frameworks like React, Vue, or
  Svelte.
---

# Tailwind Best Practices

## Workflow
1. Confirm Tailwind is wired into the build (PostCSS/Vite/Next/etc.) and a global CSS entry loads
   `@tailwind base`, `@tailwind components`, and `@tailwind utilities`.
2. Verify `content` globs only target real template sources (avoid `node_modules` and generated dirs).
3. Enforce deterministic class ordering (Tailwind Prettier plugin or formatter rule).
4. Build UI with utilities first; introduce abstractions when patterns stabilize.
5. Establish theme tokens (colors, typography, spacing) in config, not arbitrary values everywhere.
6. Validate responsive, state, and accessibility behavior.
7. Check CSS output size and fix missing/over-broad `content` or `safelist` entries.

## Authoring
- Prefer semantic structure + utility composition.
- Use variants (`hover:`, `focus:`, breakpoints) to keep styles co-located with markup.
- Prefer stable selectors (avoid styling via DOM structure assumptions).

## Theming and Tokens
- Prefer theme scales (colors/spacing/type) over arbitrary values.
- Extend the theme when adding new tokens; avoid overriding defaults unless intentional.
- Rule of thumb:
  - If you use an arbitrary value more than once, promote it to a named token in `tailwind.config.*`.

## Abstraction
- Do not abstract too early.
- Extract repeated, meaningful UI patterns into components.
- Use `@layer components` and `@apply` sparingly for stable patterns only.
- Prefer component abstraction (React/Vue/Svelte components) over CSS abstraction when possible.

## Performance
- Ensure purge/tree-shaking works via correct `content` paths.
- Avoid overly broad globs that pull in non-template files.
- Keep `safelist` minimal and justified.
- Validate production build CSS size periodically (guard against accidental safelist explosions).

## Accessibility
- Ensure focus states exist and are visible.
- Maintain contrast ratios and readable type sizes.
- Validate keyboard navigation and screen-reader semantics.
- Prefer semantic HTML + ARIA roles; stable `getByRole` selectors help testing and a11y.

## References
- `references/tailwind-css.md`

## Extended Guidance
Use this section when the Tailwind task goes beyond basic setup (design systems, scale, or long-term
maintenance).

## Authoring Checklist (Detail)
- Prefer semantic HTML first; use utilities to style, not to replace structure.
- Keep class lists readable: group by layout, spacing, typography, color, effects, then state variants.
- Use `group` and `peer` intentionally; avoid deeply nested state chains that are hard to debug.
- Choose one responsive strategy per component (mobile-first or desktop-first); avoid mixing.
- Keep layout utilities on containers and content utilities on content.
- Use `min-h` and `min-w` to protect against content clipping.
- Avoid `!important` unless the team explicitly agreed on a pattern.

## Theming Checklist (Detail)
- Define core tokens for colors, spacing, typography, radius, and shadows.
- Use semantic tokens for UI roles (primary, accent, danger, success).
- Prefer named scales over arbitrary values for repeat usage.
- Introduce a token only if it appears in at least two components.
- Document any custom colors that carry meaning (status, brand, chart categories).

## Component Abstraction Checklist
- Extract components when:
  - The pattern is used in 3+ places.
  - There is a stable interface (props) you can name clearly.
  - Changes are likely to be global.
- Avoid `@apply` for deeply variant-heavy patterns; prefer components or `clsx`/`cva`.
- Keep component-level variants explicit (`size`, `intent`, `state`).

## Build and Performance Checklist
- Ensure `content` globs only match real templates and avoid generated files.
- Keep `safelist` minimal and review on each release.
- Track CSS size by artifact (before/after) when adding new components.
- Verify `prettier-plugin-tailwindcss` (or equivalent) is enforced in CI for ordering.

## Minimal `tailwind.config` Example
```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./src/**/*.{ts,tsx,js,jsx,html}"],
  theme: {
    extend: {
      colors: {
        brand: {
          50: "#f5f7ff",
          500: "#304ffe",
          700: "#1a3ae6",
        },
      },
    },
  },
  plugins: [],
};
```

## Example: Class Grouping Convention
```html
<button class="inline-flex items-center gap-2 rounded-md px-3 py-2 text-sm font-medium text-white
  bg-brand-500 hover:bg-brand-700 focus-visible:outline-none focus-visible:ring-2
  focus-visible:ring-brand-500 disabled:opacity-50 disabled:pointer-events-none">
  Save
</button>
```

## Validation Commands (Typical)
```bash
npx tailwindcss -i ./src/index.css -o ./dist/tailwind.css --minify
npx prettier --check .
```

## Common Failure Modes
- `content` globs miss files; styles are missing in production builds.
- `safelist` is overly broad and bloats CSS output.
- Tokens are defined but unused; UI drifts to arbitrary values anyway.
- Components embed responsive logic that conflicts with parent layout logic.

## Reference Index
- `rg -n "Theme configuration|colors|spacing|typography" references/tailwind-css.md`
- `rg -n "Content configuration|purge|JIT" references/tailwind-css.md`
- `rg -n "Responsive design|breakpoints" references/tailwind-css.md`
- `rg -n "Accessibility|focus|contrast" references/tailwind-css.md`
- `rg -n "Performance|build size|safelist" references/tailwind-css.md`

## Token Audit Checklist
- Remove unused tokens after major UI changes.
- Keep typography scale small and consistent.
- Limit custom shadows to a small set of named tokens.

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
- `rg -n "Checklist|checklist" references/tailwind-css.md`
- `rg -n "Example|examples" references/tailwind-css.md`
- `rg -n "Workflow|process" references/tailwind-css.md`
- `rg -n "Pitfall|anti-pattern" references/tailwind-css.md`
- `rg -n "Testing|validation" references/tailwind-css.md`
- `rg -n "Security|risk" references/tailwind-css.md`
- `rg -n "Configuration|config" references/tailwind-css.md`
- `rg -n "Deployment|operations" references/tailwind-css.md`
- `rg -n "Troubleshoot|debug" references/tailwind-css.md`
- `rg -n "Performance|latency" references/tailwind-css.md`
- `rg -n "Reliability|availability" references/tailwind-css.md`
- `rg -n "Monitoring|metrics" references/tailwind-css.md`
- `rg -n "Error|failure" references/tailwind-css.md`
- `rg -n "Decision|tradeoff" references/tailwind-css.md`
- `rg -n "Migration|upgrade" references/tailwind-css.md`
