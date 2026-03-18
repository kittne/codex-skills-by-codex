---
name: keycloak
description: >
  Design, secure, and operate Keycloak OIDC environments for production applications.
  Use for realm/client architecture, redirect and PKCE policy, token/session settings,
  role/group claim mapping, admin API governance, and operational hardening.
---

# Keycloak

## Workflow
1. Confirm identity domains and realm boundaries.
2. Model clients by app type and trust level.
3. Enforce secure redirect URI and flow settings.
4. Configure scopes, protocol mappers, and claims intentionally.
5. Tune token/session lifetimes and logout behavior.
6. Harden admin/API access and key lifecycle.
7. Validate failover, backup, and upgrade procedures.

## Preflight (Ask / Check First)
- Keycloak version and distribution mode.
- Realm topology (single vs multi-realm).
- Public, confidential, and service-account client mix.
- Required claims and role/group model.
- Regulatory and audit constraints.

## Realm and Client Modeling
- Use separate realms only when policy boundaries require isolation.
- Classify clients correctly: public for untrusted runtimes, confidential for server-side apps.
- Keep redirect URIs explicit and narrow.
- Use exact redirect URI matching (including scheme/host case and query string where present).
- Avoid wildcard redirects for sensitive clients; do not register fragment-bearing redirect URIs.
- Keep client authentication methods explicit.

## OIDC Flow Security
- Use authorization code flow by default.
- Enforce PKCE for public clients.
- Restrict legacy or risky grants when not required.
- Keep post-logout redirect URIs validated.
- Require HTTPS and trusted certificates in production.

## Claims and Scope Governance
- Use client scopes to centralize claim policy.
- Keep default scopes minimal; move extras to optional scopes.
- Map roles/groups intentionally and document semantics.
- Avoid claim name collisions across client mappers.
- Test token payload size to prevent downstream issues.

## Token, Session, and Key Management
- Keep short access token lifetimes and bounded refresh behavior.
- Align session policies with user risk profile.
- Rotate signing keys predictably and publish rollover runbook.
- Audit admin token issuance and privileged API usage.
- Keep clock drift monitored across IdP and clients.

## Admin API and Operations
- Use least-privilege service accounts for automation.
- Version and review realm config changes as code.
- Keep backup/restore workflows tested before upgrades.
- Validate client policy controls for redirect enforcement.
- Prefer RFC 7662 token introspection (`/token/introspect`); the legacy `/validate` endpoint is deprecated.

### Basic Verification
```bash
curl -fsS "$ISSUER/.well-known/openid-configuration"
curl -fsS "$ISSUER/protocol/openid-connect/certs"
```

## Security and Incident Response
- Alert on unusual auth failures and token issuance spikes.
- Track realm and client configuration drift.
- Rehearse compromised client secret and key rotation playbooks.
- Protect admin endpoints with network and MFA controls.

## Validation Commands
```bash
curl -fsS "$ISSUER/.well-known/openid-configuration"
curl -fsS "$ISSUER/protocol/openid-connect/certs"
```

## Common Failure Modes
- Broad redirect URI patterns enabling token theft paths.
- Missing PKCE for public clients.
- Overloaded default scopes and claim sprawl.
- Token/session lifetimes disconnected from risk posture.
- Untracked admin changes causing environment drift.

## Definition of Done
- Realm/client model is secure and intentional.
- Flow, redirect, and PKCE policies are enforced.
- Scopes and claims are minimal and documented.
- Key and token lifecycle controls are operationalized.
- Monitoring and recovery runbooks are verified.

## References
- `references/keycloak-2026-02-18.md`

## Reference Index
- `rg -n "redirect|secure-redirect-uris-enforcer|PKCE" references/keycloak-2026-02-18.md`
- `rg -n "client scope|mapper|claim" references/keycloak-2026-02-18.md`
- `rg -n "token|session|key" references/keycloak-2026-02-18.md`
- `rg -n "admin API|operations|hardening" references/keycloak-2026-02-18.md`
