# Changelog

## 2026-03-22

### Skills updates
- Updated `nginx` HTTP guidance to avoid deprecated `listen ... http2` usage and removed HTTP/2 server-push directives.
- Updated `keycloak` admin/OIDC guidance to avoid the deprecated `/registrations` initiation endpoint and use standards-based `prompt=create`.
- Updated `go` module tooling guidance to track tool dependencies via the Go 1.24+ `tool` directive in `go.mod`.
- Updated `docker` CI tooling guidance to generate SBOM and provenance attestations with `buildx`.
- Updated `turborepo` env guidance to keep strict env mode as the default and treat loose mode as temporary debugging only.

## 2026-03-20

### Skills updates
- Updated `openapi` guidance for OpenAPI 3.2 JSON Schema 2020-12 alignment and `$id`-boundary interoperability references.
- Updated `mysql` operational security notes to remove deprecated `FLUSH PRIVILEGES`-based runbook patterns.
- Updated `redis` persistence defaults to call out `appendfsync everysec` as the baseline AOF policy.
- Updated `prometheus` label and remote-write guidance to reserve `__*` labels and enforce deterministic label-set validity.

## 2026-03-19

### Skills updates
- Updated `filesystem` restore/retention guidance for `restic` append-only retention safety, coupled `forget --prune`, and restore dry-run/overwrite handling.
- Updated `ubuntu` patching guidance to include phased updates in maintenance windows and explicit unattended-upgrades origins policy.
- Updated `typescript` compiler guidance for deprecated-option burn-down and Node 24 `module: "node20"` transition.
- Updated `node.js` testing guidance to add CI deprecation surfacing via `NODE_PENDING_DEPRECATION=1` / `--pending-deprecation`.
- Updated `python` upgrade watchlist to avoid new `typing.AnyStr` usage (deprecated in 3.13).
- Updated `fastify` v5 guidance for `request.socket`, null-prototype params checks, and object-form `listen`.
- Updated `vault` AWS auth guidance to prefer `accesslist`/`denylist` endpoint names over deprecated aliases.

## 2026-03-18

### Skills updates
- Updated `ruff` guidance to reflect modern default notebook lint/format behavior and required explicit `.ipynb` opt-out policy.
- Updated `docker` Compose guidance to prefer Compose Specification and avoid legacy top-level `version:` keys.
- Updated `github` automation security guidance to require explicit `GITHUB_TOKEN` permissions and prefer short-lived GitHub App tokens.
- Updated `keycloak` redirect policy guidance to require exact URI matching and disallow fragment-bearing redirect URIs.

## 2026-03-16

### Skills updates
- Updated `nestjs` migration guidance to replace deprecated middleware wildcard `'(.*)'` with named wildcard path forms for Nest 11 path matching.
- Updated `loki` storage guidance to mark Cassandra backend as deprecated for new deployments.
- Updated `mysql` upgrade guidance to remove `temptable_use_mmap` in MySQL 9.4+ to avoid config/startup failures.
- Updated `postgresql` upgrade notes for PostgreSQL 18 time zone abbreviation precedence changes.

## 2026-03-10

### Skills updates
- Updated `next.js` migration guidance to flag Next.js 16 removal of `unstable_rootParams`.
- Updated `fastapi` migration guidance to avoid `pydantic.v1` on Python 3.14 and plan v1 removal.
- Updated `playwright` setup guidance to require explicit browser install (`npx playwright install --with-deps`) in fresh/CI environments.

## 2026-03-09

### Skills updates
- Updated `nestjs` guidance for Nest 11 middleware ordering and Fastify v5 verification in upgrade testing.
- Updated `opentelemetry-js` migration guidance for deprecated semantic-convention export forms and NodeSDK-first 2.x configuration.
- Updated `mysql` replication/auth notes for `binlog_format` deprecation and `mysql_native_password` retirement timeline.
- Updated `expo-sdk` upgrade guidance to account for Expo Go SDK support-window constraints.
- Updated `go` tooling guidance to run `go mod tidy` after `go get go@...` or `toolchain` bumps.

## 2026-03-08

### Skills updates
- Updated `next.js` migration guidance for Next.js 16 `skipProxyUrlNormalize` config rename.
- Updated `fastify` v5 guidance for route `constraints.version` and plugin API style consistency.
- Updated `openapi` guidance to prefer OpenAPI 3.2 when supported and include `$self`/`$id` usage.
- Updated `github-actions` guidance for current core action majors and `node24` runtime metadata for JavaScript actions.
- Updated `typescript` compiler guidance to remove deprecated `importsNotUsedAsValues`.

## 2026-03-07

### Skills updates
- Updated `nestjs` migration guidance for Nest 11 `CacheModule` Redis store migration to `@keyv/redis`.
- Updated `expo-sdk` OTA guidance for SDK 55+ `eas update --environment` behavior and server-side env usage.
- Updated `opentelemetry-js` for JS SDK 2.x runtime/type baselines and removed tracer-provider API patterns.
- Updated `go` module/tooling guidance for explicit `go`/`toolchain` directives and supported release-line CI policy.

## 2026-03-06

### Skills updates
- Updated `next.js` upgrade guidance for Next.js 16 `middleware` to `proxy` rename and export migration.
- Updated `react` React 19 guidance for `act` import source, legacy ref/context removals, and `useActionState`/`useOptimistic` usage.
- Updated `typescript` compiler baseline with TS 5.9+ defaults and `noUncheckedSideEffectImports`.
- Updated `node.js` operational guidance for core `.env` APIs, permission model hardening, and `node:test` first guidance.
- Updated `prisma` Prisma ORM 7 guidance for generated-client import paths and `prisma.config.ts` path pinning.

## 2026-03-05

### Skills updates
- Updated `pnpm` workspace/CI guidance for `sharedWorkspaceLockfile` and explicit `--frozen-lockfile` use.
- Updated `turborepo` env guidance to avoid removed `dotEnv`/`globalDotEnv` keys.
- Updated `mypy` compatibility guidance to require Python target 3.9+.
- Updated `pyright` release guidance to run `--verifytypes` for published libraries.
- Updated `redis` guidance for Redis 8 ACL category changes and Redis 8.6 stream idempotency.
- Updated `grafana` API guidance to prefer UID-keyed datasource endpoints over deprecated numeric ID updates.
- Updated `expo-sdk` release guidance to treat Classic Updates as retired and use EAS Update.
- Updated `opentelemetry-js` semantic convention guidance to migrate deprecated `SEMATTRS_*`/`SEMRESATTRS_*` constants.

## 2026-03-04

### Skills updates
- Updated `node.js` resilience guidance for unhandled rejection policy and invalid `main` fallback deprecation risk.
- Updated `fastapi` contract guidance to treat `pydantic.v1` usage as temporary migration-only state.
- Updated `python` upgrade watchlist to prefer the `type` statement over deprecated `typing.TypeAlias`.

## 2026-03-03

### Skills updates
- Updated `prisma` to note Prisma ORM 7 generate behavior and removed skip flags.
- Updated `prometheus` upgrade guidance for Prometheus 3.0 migration/flag changes.
- Updated `react` migration guidance for React 19 deprecations and ref-as-prop changes.
- Updated `next.js` upgrade guidance for async `params`/`searchParams` in 15+ and sync removal in 16.
- Updated `tailwind` v4 migration guidance to include the upgrade tool and PostCSS plugin change.
- Updated `turborepo` upgrade guidance for `tasks` vs `pipeline` and cache/output key renames.

## 2026-03-01

### Skills updates
- Updated `next.js` caching defaults to reflect Next.js 15 fetch behavior and cache opt-in.
- Updated `paddleocr` pipeline guidance to include textline-orientation module toggles.
- Updated `openai-docs` to flag Assistants API deprecation and sunset date.
- Updated `grafana` to note datasource UID requirement for alerting/proxy APIs.
- Updated `keycloak` to prefer RFC 7662 token introspection and note `/validate` deprecation.
- Updated `loki` to capture Loki 3.0 TSDB requirements and ingestion limits.
- Updated `mypy` to call out PEP 702 deprecation reporting flags.
- Updated `mysql` replication and auth guidance for deprecated options.

## 2026-02-27

### Skills updates
- Updated `fastapi` guidance for `response_model` returning `None` and `yield` dependency re-raise.
- Updated `fastify` guidance for v5 full JSON schema requirements and type provider split.

## 2026-02-26

### Skills updates
- Updated `mysql` replication auth guidance for `caching_sha2_password` replicas.
- Updated `redis` preflight to confirm version and review breaking changes before upgrades.
- Updated `prometheus` rule guidance to convert legacy rules via `promtool`.
- Updated `loki` retention guidance for TSDB v13 schema and compactor requirements.
- Updated `python` upgrade watchlist for 3.13+ deprecations.
- Updated `mypy` strict mode notes to clarify flag coverage.
- Updated `ubuntu` release upgrade guidance for LTS and `-d` usage.

## 2026-02-25

### Skills updates
- Updated `prisma` upgrade notes to include Prisma ORM 7 generator provider change.
- Updated `tailwind` v4 setup guidance for `@import "tailwindcss"` and `@theme`.
- Updated `expo-sdk` OTA guidance for runtime version policies and build properties.
- Updated `playwright-testing` CI setup to explicitly install browsers.
- Updated `github` governance guidance to prefer rulesets for branch protections.
- Updated `github-actions` OIDC permission guidance.
- Updated `docker` build guidance for BuildKit cache/secret mounts and `COPY --link`.
- Updated `postgresql` upgrade notes for MD5 deprecation and checksum defaults.

## 2026-02-24

### Skills updates
- Updated `openapi` guidance for OpenAPI 3.1 JSON Schema dialects and nullability.
- Updated `react` effect guidance for Strict Mode replays and functional state updates.
- Updated `next.js` caching notes for dynamic APIs and Router Cache invalidation.
- Updated `typescript` compiler baseline notes for module resolution compatibility and `verbatimModuleSyntax`.
- Updated `pnpm` CI guidance to use `pnpm fetch` for offline installs.

## 2026-02-22

### Skills updates
- Updated `next.js` caching guidance to clarify `revalidate: 0` usage and avoid conflicting cache options.

## 2026-02-21

### Skills updates
- Updated `fastapi` guidance to prefer lifespan async context managers over deprecated startup/shutdown events.

## 2026-02-20

### Skills updates
- Updated `next.js` caching guidance with current App Router defaults and route segment overrides.

## 2026-02-19

### Skills updates
- Trimmed `radix` skill by one line to stay within the 100-200 line limit.

## 2026-02-18

### Skills updates (first automation run)
- Standardized skill guides to the 100-200 line requirement.
- Refined and updated the following skills:
  - `gh-address-comments`
  - `openai-docs`
  - `pdf`
  - `security-best-practices`
  - `screenshot`
  - `security`
  - `ubuntu`
  - `postgresql`
  - `radix`
  - `socket.io`
- Improved structure and guidance quality to align with skill-creator conventions.
- Validated line-count compliance across all `SKILL.md` files after updates.

### Naming consistency updates
- Renamed skill folders and metadata:
  - `docker-best-practices` -> `docker`
  - `go-best-practices` -> `go`
  - `markdown-best-practices` -> `markdown`
  - `python-best-practices` -> `python`
  - `vault-best-practices` -> `vault`
- Consolidated NGINX skills into one:
  - merged `nginx-best-practices` + `nginx-mail-best-practices` into `nginx`
- Consolidated Tailwind naming to a single canonical skill:
  - removed `tailwind-best-practices` and kept `tailwind`

### Skill expansion and consolidations
- Added 14 new canonical skills:
  - `typescript`
  - `pnpm`
  - `turborepo`
  - `authentik`
  - `keycloak`
  - `fastify`
  - `openapi`
  - `opentelemetry-js`
  - `prometheus`
  - `grafana`
  - `loki`
  - `tesseract-ocr`
  - `paddleocr`
  - `filesystem`
- Added Context7-backed dated references (`2026-02-18`) for each new skill.
- Consolidated legacy backup/streams scope into existing skills:
  - `redis`: merged `redis-streams` + `redis-backup-recovery`, added `references/redis-streams-backup-2026-02-18.md`.
  - `postgresql`: merged `postgresql-backup-restore`, added `references/postgresql-backup-restore-2026-02-18.md`.
- Enforced `SKILL.md` line-count guardrail (100-200 lines) for all touched skills.
- Ran `quick_validate.py` for all new and updated skills (all passing).
