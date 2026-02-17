---
name: nginx-mail-best-practices
description: >
  Apply secure, performant Nginx practices when editing Nginx configs targeting mail capabilities,
  reverse proxies, TLS, rate limits, Docker/compose for Nginx, or edge routing.
---

# NGINX Mail Best Practices

## Workflow
1. Confirm NGINX includes the `mail` and `mail_ssl` modules.
2. Define a `mail {}` context with per-port `server {}` listeners.
3. Configure TLS (implicit TLS and/or STARTTLS) with strong protocols/ciphers.
4. Implement `auth_http` routing and authentication safely (mandatory dependency).
5. Preserve real client IP to upstreams (XCLIENT and/or PROXY protocol) where supported.
6. Harden against relay abuse and brute-force attempts.
7. Validate configuration and test handshakes end-to-end.

## Quick Intake (Ask/Confirm)
- Which protocols/ports are required (SMTP 25, submission 587/465, IMAP 143/993, POP3 110/995)?
- What are the upstream servers (Postfix/Dovecot/Exim/etc.) and what do they support (XCLIENT, PROXY protocol)?
- Where does TLS terminate (at NGINX vs end-to-end to backends)?
- Any multi-domain requirements (mail module has limited SNI; prefer SAN/wildcard certs).
- What is the auth_http service and how is it protected (shared secret header, allowlist)?

## Core Requirements
- `auth_http` is mandatory for the mail module; treat it as a critical dependency.
- Do not expose internal mail hosts directly to the internet; proxy through NGINX.
- Disable client AUTH on port 25 (MX) unless you have a very specific reason.

## TLS
- Prefer TLS 1.2 and 1.3.
- Use implicit TLS on 993/995/465 as appropriate, and STARTTLS on 25/587/143/110 if needed.
- Use certificates that cover all served hostnames (mail module has limited SNI support).

## Auth and Routing
- The auth service should:
- Validate credentials (or deliberately defer to upstream with a documented reason).
- Return only trusted internal upstream targets (never arbitrary hosts/ports).
- Require a shared secret header from NGINX and/or network allowlisting.
- Rate-limit or delay repeated failures (brute-force defense) where possible.

## Minimal Skeleton (Example Shape)

```nginx
mail {
  server_name mail.example.com;
  auth_http 127.0.0.1:9000/auth;

  ssl on;
  ssl_certificate     /etc/ssl/certs/mail.fullchain.pem;
  ssl_certificate_key /etc/ssl/private/mail.key.pem;
  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_ciphers HIGH:!aNULL:!MD5;

  # IMAPS
  server {
    listen 993;
    protocol imap;
    ssl on;
  }

  # SMTP inbound (no AUTH)
  server {
    listen 25;
    protocol smtp;
    smtp_auth none;
    starttls on;
  }
}
```

## Client IP Preservation
- SMTP: prefer XCLIENT when supported by the upstream (e.g., Postfix).
- IMAP/POP3: use PROXY protocol when supported to avoid losing client IP for logging/fail2ban.

## Validation
```bash
nginx -t

# TLS smoke test (example)
openssl s_client -connect mail.example.com:993
```

Additional checks to consider:
- STARTTLS (SMTP submission): `openssl s_client -starttls smtp -connect mail.example.com:587`
- Validate real client IP preservation on backends (logs show real IP, not NGINX IP).

## References
- `references/nginx-mail.md`

## Extended Guidance
Use this when setting up mail proxies in production or when compliance is involved.

## Protocol Selection Checklist
- SMTP for mail submission and relay.
- IMAP for mailbox access; POP3 only for legacy clients.
- Use STARTTLS or implicit TLS; never plaintext on public networks.

## TLS and Auth Defaults
- Prefer modern ciphers and TLS 1.2/1.3.
- Enforce authentication for submission ports.
- Rotate certificates before expiration; monitor for expiry.

## Rate Limiting and Abuse Controls
- Limit connection rate per IP for submission services.
- Add `limit_conn` for backend protection.
- Monitor auth failures and alert on brute-force spikes.

## Validation Commands
```bash
nginx -t
openssl s_client -starttls smtp -connect mail.example.com:587
```

## Common Failure Modes
- Allowing unauthenticated relay.
- Misconfigured STARTTLS resulting in downgrade.
- Exposing mail endpoints publicly without rate limits.

## Reference Index
- `rg -n "SMTP|submission" references/nginx-mail.md`
- `rg -n "IMAP|POP3" references/nginx-mail.md`
- `rg -n "TLS|STARTTLS" references/nginx-mail.md`

## Operational Checklist
- Monitor queue depth and auth failures.
- Validate DNS (MX, SPF, DKIM, DMARC) outside NGINX.
- Keep logs centralized for incident response.

## Reference Index (Expanded)
- `rg -n "rate limiting|limit" references/nginx-mail.md`

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
- `rg -n "Checklist|checklist" references/nginx-mail.md`
- `rg -n "Example|examples" references/nginx-mail.md`
- `rg -n "Workflow|process" references/nginx-mail.md`
- `rg -n "Pitfall|anti-pattern" references/nginx-mail.md`
- `rg -n "Testing|validation" references/nginx-mail.md`
- `rg -n "Security|risk" references/nginx-mail.md`
- `rg -n "Configuration|config" references/nginx-mail.md`
- `rg -n "Deployment|operations" references/nginx-mail.md`
- `rg -n "Troubleshoot|debug" references/nginx-mail.md`
- `rg -n "Performance|latency" references/nginx-mail.md`
- `rg -n "Reliability|availability" references/nginx-mail.md`
- `rg -n "Monitoring|metrics" references/nginx-mail.md`
- `rg -n "Error|failure" references/nginx-mail.md`
- `rg -n "Decision|tradeoff" references/nginx-mail.md`
- `rg -n "Migration|upgrade" references/nginx-mail.md`
