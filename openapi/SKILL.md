---
name: openapi
description: >
  Design, evolve, and govern OpenAPI contracts for production HTTP APIs with compatibility,
  security, and tooling reliability. Use for schema modeling, operation design, reusable components,
  version strategy, linting, and spec-driven delivery workflows.
---

# OpenAPI

## Workflow
1. Confirm API audience, lifecycle stage, and compatibility requirements.
2. Define resource model and operation semantics before schema expansion.
3. Build reusable component schemas and parameter conventions.
4. Encode auth, error formats, pagination, and idempotency patterns.
5. Establish linting and diff gates for contract changes.
6. Validate generated clients/servers against real traffic scenarios.
7. Govern versioning and deprecation policy with explicit timelines.

## Preflight (Ask / Check First)
- OpenAPI target version (3.2 preferred when tooling supports it, otherwise 3.1).
- Consumer types (internal services, third parties, SDK generation).
- Backward-compatibility policy.
- Auth schemes and compliance constraints.
- Existing spec quality and tooling stack.

## Contract Design Rules
- Keep operations resource-oriented and predictable.
- Use stable `operationId` values for SDK safety.
- Standardize error envelope schemas.
- Keep enums and defaults explicit.
- Avoid ambiguous polymorphism unless required by business model.

## Schema and Components
- Use `components/schemas` for shared domain models.
- Reuse common parameters (pagination, correlation IDs, locales).
- Use composition intentionally (`allOf`, `oneOf`, `anyOf`) with discriminator strategy.
- In 3.1, model nullability with JSON Schema (`type: ["string", "null"]`) instead of `nullable`.
- In 3.2, require at least one of `paths`, `components`, or `webhooks` in root documents.
- Set `jsonSchemaDialect` or `$schema` explicitly when using a non-default dialect.
- In 3.2, use `$self` and schema `$id` deliberately when distributing reusable multi-document components.
- Keep numeric/string formats explicit to reduce codegen drift.
- Prefer narrow, explicit object schemas over permissive maps.

### Components Example
```yaml
components:
  schemas:
    Error:
      type: object
      required: [code, message]
      properties:
        code: { type: string }
        message: { type: string }
```

## Security and Auth
- Define security schemes centrally in components.
- Apply least-privilege scopes per operation.
- Keep auth challenge/error responses documented.
- Ensure redirect/callback/security-sensitive fields are strongly constrained.
- Verify OAuth/OIDC flow correctness in examples.

## Compatibility and Versioning
- Treat field removals and required additions as breaking.
- Use additive changes by default.
- Maintain explicit deprecation metadata and migration notes.
- Gate merges using semantic diff checks.
- Version only when compatibility policy requires it.

## Tooling and Governance
- Enforce lint rules for naming, tags, descriptions, and schema style.
- Validate examples with contract tests where possible.
- Keep codegen pinned and reviewed for template upgrades.
- Track spec ownership by domain team.
- Publish changelogs for contract consumers.

## Validation Commands
```bash
pnpm exec spectral lint openapi.yaml
pnpm exec openapi-format openapi.yaml --check
pnpm exec openapi-diff previous.yaml openapi.yaml
pnpm test
```

## Common Failure Modes
- Unstable `operationId` values breaking generated clients.
- Underspecified error responses.
- Overuse of `anyOf/oneOf` without discriminator discipline.
- Security schemes defined but not applied to operations.
- Breaking changes merged without consumer coordination.

## Definition of Done
- Spec is valid, lint-clean, and diff-gated.
- Reusable components are consistent across operations.
- Security schemes and scopes are explicit.
- Compatibility impact is documented and approved.
- Consumers can generate and use clients without manual patching.

## References
- `references/openapi-2026-02-18.md`

## Reference Index
- `rg -n "components|schemas|allOf|oneOf" references/openapi-2026-02-18.md`
- `rg -n "securitySchemes|oauth2|apiKey" references/openapi-2026-02-18.md`
- `rg -n "operationId|compatibility|version" references/openapi-2026-02-18.md`
- `rg -n "lint|diff|governance" references/openapi-2026-02-18.md`
