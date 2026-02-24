# Changelog

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
