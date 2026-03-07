---
name: expo-sdk
description: >
  Build and operate React Native mobile apps with Expo SDK and EAS. Use when choosing managed vs bare
  workflow, setting up app config and environments, upgrading SDK versions, structuring app architecture,
  optimizing performance, and handling testing, CI/CD, OTA updates, security, and store release flows.
---

# Expo SDK

## Workflow
1. Confirm product constraints and choose managed/CNG vs bare workflow.
2. Pin Expo SDK and React Native compatibility before feature work.
3. Define app configuration strategy for environments and release channels.
4. Implement architecture boundaries for navigation, state, and modules.
5. Optimize performance (bundle size, assets, startup) with profiling evidence.
6. Build test strategy and EAS pipelines for build, submit, and update.
7. Harden security/privacy defaults (secrets, storage, permissions, manifests).
8. Ship with monitoring and operational runbooks for upgrades and incidents.

## Preflight (Ask / Check First)
- Current Expo SDK version, React Native version, and target platforms.
- Need for custom native modules or build-time native changes.
- Release requirements: store cadence, OTA policy, rollback tolerance.
- Existing CI provider and EAS usage maturity.
- Security requirements for credentials, tokens, and local storage.
- Localization, accessibility, and policy constraints per platform.

## Operating Principles
- Keep dependency versions aligned with Expo SDK compatibility.
- Use `expo install` for Expo-managed dependency resolution.
- Keep environment-specific values in app config and CI secrets.
- Separate app layers: UI, domain logic, data services, platform bridges.
- Treat OTA updates as deployment events with gating and rollback rules.
- Validate store policies and permissions before release branches.

## Workflow Selection and Setup
- Prefer managed/CNG when native customization needs are limited.
- Move to bare only when native control requirements justify complexity.
- Keep project bootstrap reproducible (`create-expo-app`, locked package manager).
- Define profile naming convention for dev/preview/prod early.
- Document platform build credentials and ownership model.

### Setup Verification Commands
```bash
npx expo --version
npx expo doctor
npx expo config --type public
```

```bash
npx expo prebuild --no-install
```

## Dependency Management and Upgrades
- Upgrade Expo SDK intentionally; do not drift package versions.
- Run compatibility checks before and after upgrades.
- Upgrade in small steps and preserve rollback path.
- Re-test native integrations after each SDK upgrade.
- Keep changelog summaries and migration notes for teams.

### Upgrade Checklist
1. Create branch for SDK upgrade.
2. Run `npx expo install --fix` and align peer deps.
3. Regenerate native projects if needed.
4. Run unit/integration/smoke tests on both platforms.
5. Build preview artifacts and validate key flows.

## App Architecture and Organization
- Separate navigation from screen business logic.
- Keep state strategy explicit: local state, server state, global state boundaries.
- Encapsulate platform-specific code behind clean interfaces.
- Use feature-oriented folders for maintainability.
- Keep shared UI primitives consistent across screens.

### Suggested Module Boundaries
- `app/` routes and navigation shells.
- `features/` domain-specific screen logic.
- `components/` reusable UI primitives.
- `services/` API clients and platform adapters.
- `state/` global client state and stores.
- `utils/` pure helpers only.

## Performance Optimization
- Measure startup time and navigation transition smoothness.
- Optimize image and media payload strategy.
- Split heavy modules and defer non-critical work.
- Verify Hermes settings and profiling outputs.
- Watch memory usage on low-end device profiles.

### Performance Commands
```bash
npx expo start --no-dev --minify
npx expo export --platform all
```

```bash
npx expo doctor --fix-dependencies
```

## Testing, CI/CD, Release, and OTA
- Define test pyramid: unit, integration, and device-level smoke coverage.
- Use EAS Build profiles with explicit environment mapping.
- Automate preview builds for QA and stakeholder validation.
- Use EAS Submit with controlled release gates.
- Treat Classic Updates as retired (SDK 49 was last supported); use EAS Update instead of `expo publish`.
- Manage OTA rollout by runtime version and staged channels.
- Pick a `runtimeVersion` policy (`appVersion`, `nativeVersion`, or `fingerprint`) and keep it consistent across builds.
- For SDK 55+, run `eas update --environment <name>` and rely on EAS server env vars (not local `.env` defaults).
- Use `expo-build-properties` in app config when platform SDK/target overrides are required.

### Baseline EAS Flow
1. `eas build --profile preview` for QA validation.
2. `eas build --profile production` for store-ready artifacts.
3. `eas submit` after release approval.
4. `eas update` only for compatible runtime versions.

## Security, Privacy, Accessibility, and i18n
- Keep secrets in EAS/CI secret stores, not source control.
- Use secure storage for sensitive local data.
- Minimize permissions and align with privacy disclosures.
- Validate accessibility labels, focus order, and dynamic type behavior.
- Externalize strings and validate locale fallbacks.

## Observability and Debugging
- Track app crashes, startup failures, and release channel health.
- Correlate client telemetry with backend release versions.
- Keep deterministic reproduction steps per incident.
- Use debug tooling only with clear policy and environment separation.
- Document known device/vendor quirks in runbooks.

## Common Failure Modes
- SDK/package version drift causing build breakage.
- Bare workflow adoption without long-term native ownership plan.
- OTA updates pushed without runtime compatibility checks.
- Secrets leaked via app config or logs.
- Store rejections due to permission/privacy mismatch.
- Performance regressions from unbounded asset sizes.

## Definition of Done
- Workflow choice is justified and documented.
- SDK dependencies are aligned and upgrade path is reproducible.
- Build/release/update pipelines are tested end-to-end.
- Security, privacy, and accessibility requirements are validated.
- Monitoring and rollback runbooks are in place.

## References
- `references/expo-sdk.md`

## Reference Index
- `rg -n "managed|bare|CNG|workflow" references/expo-sdk.md`
- `rg -n "dependency|upgrade|SDK" references/expo-sdk.md`
- `rg -n "architecture|navigation|state" references/expo-sdk.md`
- `rg -n "performance|Hermes|bundle|assets" references/expo-sdk.md`
- `rg -n "EAS|CI/CD|release|submit|update|runtime version" references/expo-sdk.md`
- `rg -n "security|privacy|permissions|secure storage" references/expo-sdk.md`
- `rg -n "accessibility|internationalization|localization" references/expo-sdk.md`

## Quick Questions (When Stuck)
- Is this truly a native requirement or solvable in managed workflow?
- Are dependencies aligned with the current SDK matrix?
- Can this release be rolled back quickly if OTA/store issues occur?
- Are secrets and permissions handled to pass compliance review?
- What is the smallest cross-platform validation needed before shipping?
