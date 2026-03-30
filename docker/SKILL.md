---
name: docker
description: >
  Guide Codex in reviewing or writing Dockerfiles and Docker Compose files for best practices. This
  skill triggers when the user is creating, editing, or optimizing Docker container builds or Compose
  workflows. It covers security, efficiency, and maintainability of container images (Dockerfiles) and
  multi-service setups (Docker Compose), including build optimization, image security scanning, proper
  use of instructions, and secret management.
---

# Docker

## Workflow
1. Clarify target environment (dev vs prod), runtime, and constraints (size, security, build speed).
2. Review Dockerfile for reproducibility (pinning), minimal runtime image, and layer hygiene.
3. Enforce security defaults: non-root, least privilege, no secrets in layers, safe filesystem.
4. Review Compose for networking/exposure, healthchecks, startup order, and resource constraints.
5. Validate and automate: `docker build`, `docker compose config`, lint, scan, and smoke-run.

## Dockerfile
- Use minimal, trusted base images and pin versions/digests.
- Use multi-stage builds to keep runtime images lean.
- Add a `.dockerignore` to reduce build context.
- Combine `RUN` steps and clean package caches in the same layer.
- Prefer `COPY` over `ADD` unless you need `ADD` features.
- Use BuildKit cache mounts (`RUN --mount=type=cache`) for language dependency caches.
- Use BuildKit secrets (`RUN --mount=type=secret`) for build-time credentials.
- Consider `COPY --link` to improve layer cache reuse in large builds.
- Never bake secrets into images; inject at runtime or via build secrets.
- Run as non-root with a dedicated user.
- Add `HEALTHCHECK` for long-running services.

## Dockerfile Review Checklist (High Signal)
- Base image:
  - Prefer official/verified images; avoid full OS images unless needed.
  - Pin to a version and, for production, a digest.
- Layer hygiene:
  - Avoid `apt-get update` in a separate layer; combine install + cleanup.
  - Remove build-only deps or use multi-stage so they never reach the final image.
  - Keep build context small with `.dockerignore`.
- Reproducibility:
  - Pin language/package deps (lockfiles, hashes where supported).
  - Prefer deterministic build steps (avoid downloading unpinned artifacts).
- Security:
  - Do not use `ENV` for secrets.
  - Use `USER` (non-root) in the final stage.
  - Consider `read-only` rootfs at runtime if compatible.
- Correctness/ops:
  - Healthcheck hits a real readiness indicator.
  - Entrypoint/cmd are explicit; avoid shell forms unless needed.

## Docker Compose
- Define resource limits where supported.
- Use healthchecks and `depends_on` with `service_healthy` when needed.
- Segment networks and avoid exposing unnecessary ports.
- Use Compose secrets or external env files (not committed).
- Drop Linux capabilities and use read-only filesystems where possible.
- Pin image tags and avoid `latest` in production.
- Use the Compose Specification (`services:` root) and avoid legacy top-level `version:` keys.
- For Compose builds, use `build.sbom` and `build.provenance` when your supply-chain policy requires attestations.

## Compose Review Checklist (High Signal)
- Exposure:
  - Only publish ports that must be reachable from the host.
  - Prefer internal networks for databases/queues.
- Dependencies:
  - Use healthchecks and explicit readiness gating (avoid startup race conditions).
- Privilege:
  - Drop capabilities (`cap_drop: ["ALL"]` then add back only what is required).
  - Set `read_only: true` and mount writable volumes explicitly when possible.
- Secrets:
  - Use Compose secrets or external env files; never commit them.
  - Prefer file-mounted secrets over environment variables for high-sensitivity values.
- Limits:
  - Define CPU/memory constraints where supported; at least document intent.

## Tooling
- Lint Dockerfiles (e.g., Hadolint).
- Scan images for CVEs (e.g., Trivy, Docker Scout).
- Use BuildKit and `--pull`/`--no-cache` in CI for clean builds.
- Generate SBOM and provenance attestations in CI builds (`docker buildx build --sbom=true --provenance=true`).
- Validate Compose rendering with `docker compose config`.

Common commands:
```bash
# Clean build with updated bases
DOCKER_BUILDKIT=1 docker build --pull --no-cache -t myapp:dev .

# Inspect resulting layers
docker history --no-trunc myapp:dev

# Validate compose config renders as expected
docker compose config

# Scan for vulnerabilities (if available)
trivy image myapp:dev
docker scout cves myapp:dev
```

## Review Checklist
- Base images are pinned and minimal.
- Build layers are clean and reproducible.
- Secrets are not embedded in images.
- Containers run as non-root with limited privileges.
- Compose networking and healthchecks are sane.
- CI includes linting and vulnerability scans.

## References
- `references/docker.md`

## Extended Guidance
Use this when hardening production images or optimizing build performance.

## Multi-Stage Build Checklist
- Build dependencies in a builder stage.
- Copy only the runtime artifacts into the final image.
- Use minimal base images for the runtime stage.

## Security Checklist
- Avoid root in the final image.
- Pin base image tags to a digest where possible.
- Scan images in CI and fail on high/critical CVEs.

## Build Performance Tips
- Maximize cache by ordering stable layers first.
- Use `.dockerignore` to exclude large or sensitive files.
- Prefer `--mount=type=cache` for language package caches when supported.

## Compose Checklist
- Add health checks for dependent services.
- Avoid embedding secrets in `docker-compose.yml`.
- Document required environment variables.

## Reference Index
- `rg -n "multi-stage|builder" references/docker.md`
- `rg -n "security|root|user" references/docker.md`
- `rg -n "compose|healthcheck" references/docker.md`

## Runtime Checklist
- Use `HEALTHCHECK` for critical services.
- Set explicit `USER` in final stage.
- Document exposed ports and volumes.

## Reference Index (Expanded)
- `rg -n "layer caching|.dockerignore" references/docker.md`

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
- `rg -n "Checklist|checklist" references/docker.md`
- `rg -n "Example|examples" references/docker.md`
- `rg -n "Workflow|process" references/docker.md`
- `rg -n "Pitfall|anti-pattern" references/docker.md`
- `rg -n "Testing|validation" references/docker.md`
- `rg -n "Security|risk" references/docker.md`
- `rg -n "Configuration|config" references/docker.md`
- `rg -n "Deployment|operations" references/docker.md`
- `rg -n "Troubleshoot|debug" references/docker.md`
- `rg -n "Performance|latency" references/docker.md`
- `rg -n "Reliability|availability" references/docker.md`
- `rg -n "Monitoring|metrics" references/docker.md`
- `rg -n "Error|failure" references/docker.md`
- `rg -n "Decision|tradeoff" references/docker.md`
- `rg -n "Migration|upgrade" references/docker.md`
