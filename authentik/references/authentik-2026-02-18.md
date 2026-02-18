# Authentik Reference (2026-02-18)

## Context7 Sources
- Library: `/goauthentik/authentik`
- OAuth2/OIDC provider API and integration docs surfaced via Context7.

## Practical Guidance
- Configure each OIDC integration as a dedicated provider/application pair.
- Keep redirect URI matching strict; regex only when justified.
- Set client type (`public` vs `confidential`) to match threat model.
- Tune `access_token_validity` and `refresh_token_validity` deliberately.
- Verify `/.well-known/openid-configuration` and JWKS endpoints in each environment.

## Operational APIs Mentioned
- `GET /providers/oauth2/`
- `POST /providers/oauth2/`
- `GET /providers/oauth2/{id}/setup_urls/`
- `GET /providers/oauth2/{id}/preview_user/?for_user=...`

## Source URLs
- <https://github.com/goauthentik/authentik>
- <https://goauthentik.io/docs/>
