---
name: nginx-best-practices
description: >
  Comprehensive, up-to-date guidance for secure and performant NGINX configurations (TLS, reverse
  proxy, caching, etc). Use for any task creating, updating, auditing, or troubleshooting NGINX
  setups to apply modern best practices.
---

# NGINX Best Practices

## Workflow
1. Identify the traffic model (static, reverse proxy, API gateway, TLS termination).
2. Apply secure TLS configuration and security headers.
3. Configure proxying: upstreams, timeouts, buffering, WebSockets (if needed).
4. Add caching and rate limiting where appropriate.
5. Validate and reload safely.

## Quick Intake (Ask/Confirm)
- NGINX version and deployment style (systemd VM/bare metal, container, Kubernetes ingress).
- Domain(s) and certificate source (Let’s Encrypt/ACME, internal PKI).
- Backend topology (single upstream vs multiple; keepalive needs).
- Protocol needs (HTTP/2, gRPC, WebSockets/SSE, large uploads, streaming).
- Client IP preservation needs (behind LB/CDN? PROXY protocol?).
- Security requirements (HSTS, CSP, admin surface restrictions, WAF).

## Configuration Hygiene
- Keep site-specific configs in included files (`conf.d/`, `sites-enabled/`) where applicable.
- Prefer small, composable includes over a single monolithic config.

## TLS and Security
- Enable TLS 1.2 and 1.3 only.
- Redirect HTTP to HTTPS.
- Enable HSTS only when you are confident HTTPS is permanent.
- Set secure headers when appropriate (CSP, X-Content-Type-Options, etc.).
- Disable version leakage: `server_tokens off;`.

Minimum HTTP to HTTPS redirect:
```nginx
server {
  listen 80;
  server_name example.com;
  return 301 https://$host$request_uri;
}
```

## Reverse Proxy
- Preserve client and scheme headers (`Host`, `X-Forwarded-For`, `X-Forwarded-Proto`).
- Set sensible timeouts; avoid unlimited reads.
- Configure WebSockets upgrade headers only on WebSocket locations.

Baseline proxy headers:
```nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

WebSockets (only where needed):
```nginx
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
```

## Caching and Static Assets
- Cache fingerprinted static assets aggressively.
- Use proxy caching only for safe, non-user-specific responses.

## Rate Limiting and Abuse Controls
- Use `limit_req` and optionally `limit_conn` for public endpoints.
- Prefer allowlists for admin surfaces.

## Logging and Observability
- Ensure access/error logs are enabled and include request IDs when available.
- Log upstream timings for performance debugging.
- If behind a proxy/LB, configure `real_ip_header`/`set_real_ip_from` so logs and rate limits use real client IPs.

## Validation
```bash
nginx -t
nginx -T | less

# Prefer reload over restart for config changes
systemctl reload nginx
```

Post-change smoke checks:
- `curl -I http://<host>` returns a 301 to https.
- `curl -I https://<host>` shows expected headers and status.
- If proxying: verify upstream responses and check `error.log` for warnings/errors.

## References
- `references/nginx.md`

## Extended Guidance
Use this for production hardening or multi-upstream configurations.

## Performance Tuning Checklist
- Use keepalive connections to upstreams where supported.
- Set sensible `proxy_read_timeout` and `proxy_send_timeout`.
- Enable gzip or brotli only if you have measured benefit.

## Example Rate Limit Block
```nginx
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;

server {
  location /api/ {
    limit_req zone=api_limit burst=20 nodelay;
    proxy_pass http://api_upstream;
  }
}
```

## Example Cache Block
```nginx
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=static_cache:10m max_size=1g inactive=60m;

location /assets/ {
  proxy_cache static_cache;
  proxy_cache_valid 200 1h;
  proxy_pass http://assets_upstream;
}
```

## Common Failure Modes
- Missing `real_ip_header` when behind a proxy or CDN.
- Caching user-specific responses by mistake.
- Infinite proxy buffering for streaming workloads.

## Reference Index
- `rg -n "proxy_pass|upstream" references/nginx.md`
- `rg -n "TLS|HSTS|security headers" references/nginx.md`
- `rg -n "caching|proxy_cache" references/nginx.md`

## Deployment Checklist
- Validate config with `nginx -t` before reload.
- Prefer reload over restart for zero-downtime changes.
- Keep `error_log` at `warn` or higher in production.

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
- `rg -n "Checklist|checklist" references/nginx.md`
- `rg -n "Example|examples" references/nginx.md`
- `rg -n "Workflow|process" references/nginx.md`
- `rg -n "Pitfall|anti-pattern" references/nginx.md`
- `rg -n "Testing|validation" references/nginx.md`
- `rg -n "Security|risk" references/nginx.md`
- `rg -n "Configuration|config" references/nginx.md`
- `rg -n "Deployment|operations" references/nginx.md`
- `rg -n "Troubleshoot|debug" references/nginx.md`
- `rg -n "Performance|latency" references/nginx.md`
- `rg -n "Reliability|availability" references/nginx.md`
- `rg -n "Monitoring|metrics" references/nginx.md`
- `rg -n "Error|failure" references/nginx.md`
- `rg -n "Decision|tradeoff" references/nginx.md`
- `rg -n "Migration|upgrade" references/nginx.md`
