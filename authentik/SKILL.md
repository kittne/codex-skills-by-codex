---
name: authentik
description: >
  Design, secure, and operate authentik-based OIDC integrations for production systems.
  Use for application/provider setup, scopes and claims mapping, redirect URI policy,
  token/session lifetime decisions, signing key management, and incident-ready operations.
---

# Authentik

## Workflow
1. Confirm identity architecture, trust boundaries, and app risk profile.
2. Model tenants, applications, and OAuth2/OIDC providers.
3. Configure redirect URI policy and client type correctly.
4. Define scopes, claims/property mappings, and subject strategy.
5. Set token/session lifetimes aligned with threat model.
6. Validate metadata, login/logout flows, and key rotation behavior.
7. Operationalize monitoring, audit logs, and recovery runbooks.

## Preflight (Ask / Check First)
- Authentik version and deployment mode.
- Public vs confidential client usage.
- Required claims and group/role mapping expectations.
- Session and token lifetime requirements.
- TLS, reverse-proxy, and domain model.

## Provider and Application Modeling
- Create one provider per application trust boundary.
- Choose `client_type` correctly (`public` vs `confidential`).
- Keep redirect URI matching strict unless a strong regex case exists.
- Separate human login apps from machine-to-machine access paths.
- Keep issuer mode explicit per provider where needed.

## Scopes, Claims, and Subject Design
- Minimize default scopes; add optional scopes intentionally.
- Use property mappings to control claim release.
- Preview claims per user before production enablement.
- Prefer stable subject strategy for downstream account linking.
- Avoid overloading ID tokens with unnecessary claims.

## Token and Session Security
- Keep access token validity short.
- Scope refresh token lifetime to business need.
- Require strong signing key hygiene and rotation procedures.
- Validate logout/end-session endpoints for all relying parties.
- Protect client secrets in secret stores, never in repo config.

## Integration Guardrails
- Validate OIDC discovery endpoint and JWKS availability.
- Test callback and logout URLs in every environment.
- Use strict redirect validation for internet-facing clients.
- Ensure clock sync across IdP and relying parties.
- Explicitly document offline_access usage and revocation behavior.

### Operational API Checks
```bash
curl -H "Authorization: Bearer $TOKEN" "$AUTHENTIK/providers/oauth2/"
curl -H "Authorization: Bearer $TOKEN" "$AUTHENTIK/providers/oauth2/<id>/setup_urls/"
curl -H "Authorization: Bearer $TOKEN" "$AUTHENTIK/providers/oauth2/<id>/preview_user/?for_user=<uid>"
```

## Operations and Incident Readiness
- Monitor auth failure spikes and unusual grant patterns.
- Keep audit log retention aligned with compliance needs.
- Rehearse key compromise and client secret rotation procedures.
- Document break-glass admin access and recovery paths.

## Validation Commands
```bash
curl -fsS "$ISSUER/.well-known/openid-configuration"
curl -fsS "$JWKS_URI"
```

## Common Failure Modes
- Regex redirect URIs that are overly broad.
- Long-lived tokens with weak revocation controls.
- Inconsistent subject mapping across environments.
- Claim overexposure through default mappings.
- Missing recovery plan for signing key issues.

## Definition of Done
- Provider/app model matches trust boundaries.
- Redirect and scope policies are least-privilege.
- Claims are reviewed and deterministic.
- Token/session settings match security policy.
- Monitoring and recovery procedures are documented and tested.

## Operational Checklist
- OIDC metadata endpoints are monitored for availability.
- Redirect URI policy is reviewed at each app onboarding.
- Client secret rotation cadence is documented and tested.
- Scope and claim reviews run before production releases.
- Signing key rollover drill is executed periodically.
- Audit log retention and access policy are enforced.

## References
- `references/authentik-2026-02-18.md`

## Reference Index
- `rg -n "providers/oauth2|setup_urls|preview_user" references/authentik-2026-02-18.md`
- `rg -n "redirect|client_type|issuer" references/authentik-2026-02-18.md`
- `rg -n "claims|property mapping|scope" references/authentik-2026-02-18.md`
- `rg -n "token|session|signing key" references/authentik-2026-02-18.md`
