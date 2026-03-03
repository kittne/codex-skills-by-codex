# Changelog

## 2026-03-03

### Skills updates
- Updated `prisma` to note Prisma ORM 7 generate behavior and removed skip flags.
- Updated `prometheus` upgrade guidance for Prometheus 3.0 migration/flag changes.
- Updated `react` migration guidance for React 19 deprecations and ref-as-prop changes.

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
