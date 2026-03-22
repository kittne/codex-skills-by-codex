---
name: nginx
description: >
  Comprehensive guidance for secure and performant NGINX configuration across web and mail proxy
  use cases. Use when creating, updating, auditing, or troubleshooting NGINX for TLS, reverse
  proxying, caching, rate limits, edge hardening, or mail protocols (SMTP/IMAP/POP3) with
  `auth_http`, STARTTLS, and client-IP preservation.
---

# NGINX

## Workflow
1. Identify operating mode (HTTP/reverse proxy, mail proxy, or both).
2. Confirm traffic, TLS, and trust-boundary requirements.
3. Apply secure baseline config (protocols, headers, limits, least exposure).
4. Configure mode-specific routing/proxy behavior.
5. Validate with `nginx -t`, reload safely, and run protocol-specific smoke tests.

## Quick Intake (Ask/Confirm)
- NGINX version and deployment mode (VM/systemd, container, ingress).
- Domains/certificates and TLS termination model.
- Upstream topology and real-client-IP requirements.
- Protocol requirements:
  - HTTP mode: HTTP/2, gRPC, WebSockets/SSE, upload/streaming limits.
  - Mail mode: SMTP/IMAP/POP3 ports, STARTTLS/implicit TLS requirements.
- Security constraints (HSTS/CSP, admin surface restrictions, abuse controls, compliance).

## HTTP / Reverse Proxy Guidance
- Keep config modular (`conf.d/`, `sites-enabled/`, includes).
- Use TLS 1.2/1.3 only; redirect HTTP to HTTPS.
- Prefer `http2 on;` and avoid deprecated `listen ... http2` parameters.
- Disable version leakage (`server_tokens off;`).
- Preserve forwarding headers (`Host`, `X-Forwarded-For`, `X-Forwarded-Proto`).
- Use explicit proxy timeouts; avoid unbounded reads/writes.
- Configure WebSocket upgrade headers only where needed.
- Add `limit_req`/`limit_conn` on public APIs and auth-heavy endpoints.
- Cache only safe, non-user-specific content.
- Do not use removed HTTP/2 server-push directives (`http2_push*`).

Baseline proxy headers:
```nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

## Mail Proxy Guidance
- Confirm `mail` and `mail_ssl` modules are available.
- Treat `auth_http` as mandatory for NGINX mail proxying.
- Define a dedicated `mail {}` context with per-protocol listeners.
- Prefer TLS 1.2/1.3 and use STARTTLS/implicit TLS intentionally by port.
- Do not allow client AUTH on port 25 (MX) unless explicitly required.
- Ensure `auth_http` returns only trusted internal upstream targets.
- Protect the auth backend with shared-secret headers and/or network allowlisting.
- Preserve client IP to upstreams via XCLIENT/PROXY protocol where supported.
- Add brute-force protection and connection limits on exposed mail endpoints.

Minimal mail skeleton:
```nginx
mail {
  server_name mail.example.com;
  auth_http 127.0.0.1:9000/auth;

  ssl on;
  ssl_protocols TLSv1.2 TLSv1.3;

  server {
    listen 993;
    protocol imap;
    ssl on;
  }

  server {
    listen 25;
    protocol smtp;
    smtp_auth none;
    starttls on;
  }
}
```

## Validation
```bash
nginx -t
nginx -T | less
systemctl reload nginx
```

HTTP checks:
- `curl -I http://<host>` returns HTTPS redirect.
- `curl -I https://<host>` returns expected headers/status.

Mail checks:
- `openssl s_client -connect mail.example.com:993`
- `openssl s_client -starttls smtp -connect mail.example.com:587`

## Common Failure Modes
- Missing real IP config behind a CDN/LB.
- Caching user-specific responses.
- Overly broad exposed ports/services.
- Misconfigured STARTTLS or weak TLS policy.
- Mail auth endpoint trusting arbitrary upstream targets.
- Unauthenticated relay exposure or missing rate limits.

## References
- `references/nginx.md`
- `references/nginx-mail.md`

## Reference Index
- `rg -n "proxy_pass|upstream|real_ip" references/nginx.md`
- `rg -n "TLS|HSTS|security headers|limit_req" references/nginx.md`
- `rg -n "SMTP|IMAP|POP3|auth_http|STARTTLS" references/nginx-mail.md`
