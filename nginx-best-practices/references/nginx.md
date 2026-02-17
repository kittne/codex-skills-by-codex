________________________________________
name: nginx-best-practices description: > Comprehensive, up-to-date guidance for secure and performant NGINX configurations (TLS, reverse proxy, caching, etc). Use for any task creating, updating, auditing, or troubleshooting NGINX setups to apply modern best practices.
________________________________________
NGINX Best Practices Guide
Overview
This guide outlines modern best practices for configuring NGINX as a web server or reverse proxy. It covers secure and performant configuration of NGINX including TLS/SSL settings, reverse proxy and load balancing, caching, rate limiting, logging, and security hardening. It is platform- and language-agnostic, meant to be applied to any NGINX setup (including containerized deployments) to ensure robust, secure, and efficient operation. Use this guide whenever creating, updating, auditing, or troubleshooting NGINX configurations to follow current best practices.
Configuration Structure and Management
•	Organize configuration files: Keep the main settings in the global nginx.conf and use included files (e.g. conf.d/ or sites-available/ and sites-enabled/ on Debian-based systems) for site-specific or service-specific configurations. This separation makes maintenance easier and reduces errors.
•	Use include directives: Leverage include to segment config (for example, include all vhost configs via include /etc/nginx/conf.d/*.conf;). This allows enabling or disabling specific sites or modules by adding/removing files instead of editing one large config.
•	Preserve defaults if secure: NGINX’s defaults are sane in many cases (for example, it defaults to TLS 1.2+ and strong ciphers[1]). Only override settings when necessary. Avoid duplicating default configs unless you need to change them.
•	Backup and version control: Track NGINX config files in version control or keep backups. This helps review changes and quickly revert if a new configuration causes issues. Always test changes in a staging environment if possible before production.
TLS/SSL Configuration
Always enable HTTPS to secure traffic. Key TLS best practices include:
- Enable strong protocols: Use TLS 1.2 and 1.3 only; disable outdated protocols (SSLv3, TLS1.0, TLS1.1) for security[2]. For example:

server {
    listen 443 ssl http2;
    server_name example.com;
    ssl_certificate     /etc/ssl/certs/example.com.crt;
    ssl_certificate_key /etc/ssl/private/example.com.key;
    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache   shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_stapling on;
    ssl_stapling_verify on;
    ...
}
The http2 directive enables HTTP/2 for better performance (multiplexing, header compression). The cipher suite HIGH:!aNULL:!MD5 is a broadly secure set; for the most up-to-date ciphers, you can refer to Mozilla’s SSL configuration generator (e.g. “Intermediate” profile). Enabling ssl_prefer_server_ciphers on; ensures the server’s cipher order is used. OCSP Stapling (ssl_stapling) should be on with ssl_trusted_certificate set to your CA bundle, so clients can verify certificates efficiently[2].
- Redirect HTTP to HTTPS: For each site, have an HTTP server block that sends a 301 redirect to the equivalent HTTPS URL. This ensures all users end up on a secure connection by default. For example:

server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}
- HSTS header: Enable HTTP Strict Transport Security so that browsers remember to only use HTTPS. This is done by adding a response header, e.g.:

add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
This instructs browsers to never downgrade to HTTP for one year (adjust max-age as needed)[3]. Only add the preload flag if you intend to submit the domain to the browser preload list (and be very sure you want to enforce HTTPS permanently).
- Certificate management: Use strong, trusted certificates (2048+ bit RSA or ECC). Automate renewal if using Let’s Encrypt (via certbot or ACME client). Store private keys securely (with limited permissions). Consider using ECDSA certificates for better performance (you can provide both RSA and ECDSA certs for compatibility).
- Perfect Forward Secrecy: Ensure cipher suites providing forward secrecy (ECDHE) are enabled (the “HIGH” ciphers above include these). Also consider generating a Diffie-Hellman parameter (ssl_dhparam) of 2048+ bits and referencing it, if you need to support DHE ciphers.
- TLS Testing: After configuring, test your HTTPS setup with tools like SSL Labs or Mozilla Observatory to confirm protocol support and header configuration. This helps identify any weak ciphers or misconfigurations.
Reverse Proxy and Load Balancing
NGINX excels as a reverse proxy, forwarding client requests to backend application servers. Follow these practices:
- Basic proxy configuration: In your server block for the site, use a location to forward traffic. For example:

# Reverse proxy all traffic to a backend server
location / {
    proxy_pass http://backend;        # upstream server (defined in upstream{} or an IP:port)
    proxy_http_version 1.1;           # use HTTP/1.1 for proxy (required for WebSockets, KEEPALIVE)
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    # ... other proxy settings
}
This configuration passes the request to an upstream named “backend” (which you define with an upstream block or use a direct IP/port). It also preserves important headers: the original Host (so the backend knows which hostname was requested), the client’s IP (X-Real-IP and the standard X-Forwarded-For chain), and the protocol (X-Forwarded-Proto so the backend knows if the client used HTTPS). These headers ensure the backend application can log the real client IP and enforce HTTPS-specific behavior if needed.
- Upstream keepalives: Define upstream servers and enable keepalive connections for efficiency. For example:

upstream backend {
    server 10.0.0.5:8000 max_fails=3 fail_timeout=30s;
    server 10.0.0.6:8000 max_fails=3 fail_timeout=30s;
    keepalive 32;
}
This load balances between two backend servers. max_fails and fail_timeout mark a server down if it fails repeatedly. The keepalive 32 allows up to 32 idle keepalive connections to each upstream, reducing TCP handshake overhead for persistent traffic. By default NGINX uses round-robin; you can choose algorithms like least_conn; for least connections load balancing (or ip_hash if session persistence is needed).
- WebSocket support: If your application uses WebSockets or other protocols that require HTTP upgrade, add the appropriate headers in the proxy configuration. For example, a WebSocket endpoint might be configured as:

location /socket/ {
    proxy_pass http://backend;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    # timeouts can be increased for long-lived connections
    proxy_read_timeout 300s;
}
Here, the Upgrade and Connection "upgrade" headers are critical – they tell NGINX to switch protocols to WebSocket for that connection[4]. Ensure proxy_http_version 1.1 is set (HTTP/2 is not compatible with upstream WebSocket, so NGINX will use 1.1 to talk to backend). Also consider longer proxy_read_timeout for WebSocket or SSE endpoints to avoid disconnecting idle connections.
- Timeouts and buffers: Set proxy timeouts to prevent hangs. For example, proxy_connect_timeout 60s; proxy_send_timeout 60s; proxy_read_timeout 60s; as baseline values (tune as needed). These define how long NGINX will wait for connecting to the backend, for sending a request, and for receiving a response, respectively. Likewise, adjust buffer sizes if dealing with large responses or headers (e.g., proxy_buffers 16 4k; proxy_buffer_size 4k; etc.).
- Connection reuse: By default, NGINX will add Connection: close when proxying HTTP/1.0 clients. In modern setups with HTTP/1.1+, that’s less common. You generally don’t need to manually set Connection "" (empty) unless working around specific issues. The above keepalive and HTTP/1.1 settings suffice for persistent connections.
- Health checks: NGINX (open source) doesn’t have active health checks built-in, but you can approximate by marking a backend down on failures and optionally using passive health checks (via fail_timeout and max_fails as shown). Additionally, you can create a simple health endpoint (as in the example config, a /health location returning 200) on your app and monitor it. Tools like external monitoring or scripts can periodically hit the health URL and alert if unhealthy. (NGINX Plus supports active health checks, but for OSS, consider external solutions or modules if needed.)
Caching Strategies
Caching can drastically improve performance by serving repeated requests from memory or disk instead of hitting the backend every time:
- Static content caching: For truly static files (images, CSS, JS, etc.), configure long expiration times and cache-control headers. For instance:

# Cache static files for 1 year
location ~* \.(css|js|png|jpg|jpeg|gif|ico|svg|woff2?)$ {
    expires 30d;
    add_header Cache-Control "public, immutable";
    access_log off;
}
This tells clients to cache these file types for 30 days (or even 365 days for versioned assets) and marks them as immutable if they won’t change. This offloads repeated requests from your server entirely and improves client load times[5]. Ensure that when you do update these files, you change their filenames (cache-busting) or lower the expiry if needed, since clients will not re-fetch until expiry or filename change.
- Proxy caching (microcaching): NGINX can cache proxied responses. Define one or more cache zones in the http context:

proxy_cache_path /var/cache/nginx/app_cache levels=1:2 keys_zone=app_cache:50m max_size=1g inactive=60m use_temp_path=off;
This sets up a disk cache at /var/cache/nginx/app_cache with up to 1GB storage, 50MB index in memory, and cache items that haven’t been used in 60 minutes expire. In a location or server block, enable caching:

location /api/ {
    proxy_cache app_cache;
    proxy_cache_valid 200 10m;
    proxy_cache_valid 404 1m;
    proxy_cache_use_stale error timeout http_500 http_502 http_503 http_504;
    proxy_ignore_headers Cache-Control X-Accel-Expires;
    proxy_pass http://api_backend;
    ...
}
In this example, successful responses from /api/ are cached for 10 minutes, 404s for 1 minute. If the backend returns an error or times out, proxy_cache_use_stale permits serving a stale cached copy to the client to hide outages. proxy_ignore_headers can ignore certain headers from backend that might disable caching. Important: do not cache any content that is user-specific or sensitive (e.g. logged-in pages or responses containing private data). Use conditions like proxy_no_cache and proxy_cache_bypass with appropriate variables (such as $cookie_session or $http_authorization) to avoid caching personalized content[6]. In the snippet above, you might add proxy_no_cache $http_authorization; to avoid caching authorized requests.
- Client-side caching: Ensure appropriate ETag or Last-Modified headers are set by your application or use NGINX’s etag on; (default is on) so that browsers can do conditional requests. For static files, NGINX will handle Last-Modified based on file timestamp automatically. This avoids sending full responses when content is unchanged (client gets a 304 Not Modified).
- Compression: Enable GZIP (or Brotli if available) to reduce size of text-based files. Gzip is often enabled by default in NGINX installations. Confirm that in nginx.conf you have:

gzip on;
gzip_types text/plain text/css application/javascript application/json application/xml+rss;
(and other types as needed). Do not compress already compressed content (like images or zip files). Enabling compression improves bandwidth usage and loading times for clients.
Rate Limiting
To prevent abuse and basic DDoS mitigation, configure rate limits:
- Request rate limiting: Use the limit_req_zone directive to define a leaky-bucket algorithm rate limit. For example, in the http context:

limit_req_zone $binary_remote_addr zone=api_limit:10m rate=5r/s;
This creates a zone named “api_limit” using 10MB of shared memory, tracking requests per second per IP ($binary_remote_addr as the key) with a limit of 5 requests per second[7]. (10m of shared memory is plenty for thousands of IPs). Then in a specific location that you want to protect (say /api/ or a login URL), apply the limit:

location /api/ {
    limit_req zone=api_limit burst=10 nodelay;
    # ... proxy_pass or other config
}
Here burst=10 allows a small burst beyond the rate (10 requests) with no delay (nodelay will drop immediately if the burst queue is exceeded). You can omit nodelay to instead queue excess requests briefly. The limit_req module will return HTTP 503 for clients that exceed the limit (you can customize the status code). This throttling helps mitigate brute-force or scraping attacks by slowing them down to a manageable rate.
- Concurrent connection limiting: You can also limit simultaneous connections from one IP. For example:

limit_conn_zone $binary_remote_addr zone=conn_limit:10m;
server {
    location /downloads/ {
        limit_conn conn_limit 3;
        # ... config
    }
}
This would limit each IP to 3 concurrent connections to the /downloads/ location[8]. Excess connections are rejected with an error. This is useful for resource-intensive endpoints or to mitigate certain DoS patterns.
- Rate limit considerations: Be careful not to set the limits too low and block legitimate users (especially behind NAT or corporate proxies, where many users share an IP). Adjust the key if needed (for example, combine IP with URL for more granularity, or use a user/account identifier if available and appropriate). Also note that rate limiting by IP can be ineffective against a distributed attack, but it helps against basic cases.
Security Headers and Hardening
Securing NGINX involves both the web traffic and the server itself. Here are essential practices:
•	Add security headers: Protect clients by adding HTTP security headers in your server blocks:
•	Strict-Transport-Security (HSTS): As noted, use max-age=…; includeSubDomains; preload (with caution on preload) to enforce HTTPS[3].
•	X-Frame-Options: Prevent clickjacking by disallowing your pages in iframes. Usually add_header X-Frame-Options "SAMEORIGIN" always; is suitable (allow iframes from the same site, or use DENY to disallow all).
•	X-Content-Type-Options: Use add_header X-Content-Type-Options "nosniff" always; to stop browsers from MIME-sniffing responses and potentially interpreting scripts/styles as a different MIME type than declared.
•	X-XSS-Protection: Although modern browsers have deprecated this header in favor of Content Security Policy, you can still add add_header X-XSS-Protection "1; mode=block" always; to enable basic XSS filtering on older browsers[3].
•	Referrer-Policy: Control what referrer information is sent. A safe default is add_header Referrer-Policy "strict-origin-when-cross-origin" always; (which doesn’t send full referrer details to external sites).
•	Content-Security-Policy (CSP): Consider implementing a CSP to restrict sources of scripts, iframes, etc., which can greatly mitigate XSS. A very strict CSP can break functionality, so this may require tuning for your application. Even a basic default-src 'self' policy is better than none, if applicable. Add with caution and test thoroughly (CSP is complex and usually app-specific).
Ensure you include the always flag on add_header for these, so they are set even on error responses. These headers significantly improve your site’s security posture[9][10].
•	Hide version details: Set server_tokens off; in nginx.conf to prevent NGINX from advertising its version in error pages or headers. This makes it slightly harder for attackers to detect vulnerable versions (though security through obscurity helps only marginally, it’s easy to do).
•	Disable unwanted modules: If you compile NGINX from source or use modules, only include what you need. For instance, if you don’t use ngx_http_autoindex_module (directory listings) or others, disabling them reduces potential attack surface. In packaged installations this is less flexible, but you can at least turn off autoindex in config (autoindex off; by default it’s off).
•	Time-out slow clients: Mitigate Slowloris-style attacks (clients that hold connections open) by setting timeouts. For example, in the http or server context:

 	client_body_timeout 10s;
client_header_timeout 10s;
keepalive_timeout 65s;
send_timeout 10s;
 	The first two directives will close connections if the client stops sending data (body or headers) for more than 10 seconds[11]. send_timeout does similar while sending response to a slow client. These values balance being low enough to drop slow malicious connections, but high enough not to interrupt legitimate users on very slow connections.
•	Access control: Restrict access to sensitive paths or admin interfaces by IP whitelisting or authentication. For instance, to lock down a /admin area to only an internal subnet:

 	location /admin {
    allow 192.168.10.0/24;
    deny all;
    auth_basic "Admin Area";
    auth_basic_user_file /etc/nginx/.htpasswd;
    proxy_pass http://backend_admin;
}
 	The above example uses both IP allow/deny and basic auth as defense in depth. Even if you don’t use basic auth, using allow/deny directives can block unwanted scanners from even hitting an admin endpoint[12]. Similarly, if you expose status metrics (e.g. stub_status or /metrics for Prometheus), restrict those to localhost or monitoring systems only[13][14].
•	Fail2Ban integration: Utilize fail2ban (or similar intrusion prevention) to ban abusive clients at the firewall level. Fail2ban can monitor NGINX logs and automatically add iptables rules to block IPs that trigger certain patterns (e.g., too many 4xx/5xx errors, repeated exploitation attempts). For example, after a number of failed login attempts or 404 errors, fail2ban can ban that IP for a period of time. This can significantly mitigate brute force and bot attacks by automatically banning offending IPs[15]. Be sure to configure appropriate fail2ban “jail” rules for your NGINX access or error logs.
•	Logging and monitoring: Use custom log formats to capture useful data (for example, log the upstream response time, $remote_addr, $request_uri, etc.). Monitor logs for anomalies (many 5xx errors, sudden spikes in requests). Set up monitoring on NGINX’s own metrics if needed (like requests per second, active connections). Ensure logs are rotated to prevent disk fill-ups.
•	Run workers with least privilege: By default on Linux, NGINX master process runs as root (to bind low ports) but worker processes run as a less-privileged user (often “nginx” or “www-data”). Keep this configuration – do not run everything as root. In container scenarios, you can run NGINX as a non-root user entirely by using high unprivileged ports or Linux capabilities, but that can be complex. At minimum, ensure the default user nginx; (or www-data) is in nginx.conf for worker processes.
•	Web application firewall (WAF): For higher security needs, consider adding a WAF. There are NGINX modules like ModSecurity or cloud services that can inspect and filter malicious HTTP requests (e.g., SQL injection, XSS attempts). This is an advanced topic, but worth noting if you need to enforce complex rules.
Preserving Client IP
In a typical setup, NGINX will see the real client’s IP in $remote_addr (for direct connections). However, when NGINX itself is behind another proxy or load balancer (such as in cloud deployments, or behind Cloudflare/AWS ELB), you need to ensure the actual client IP is preserved and visible to NGINX and your app:
•	Using X-Forwarded-For: If NGINX is behind a proxy that adds an X-Forwarded-For header (common for AWS ELB or CDN services), enable the Real IP module in NGINX. This module will replace $remote_addr with the client’s IP from a trusted header. For example:

 	http {
    set_real_ip_from 10.0.0.0/8;    # trust this subnet (e.g., your load balancer)
    set_real_ip_from 192.168.0.0/16;
    set_real_ip_from 172.16.0.0/12;
    real_ip_header X-Forwarded-For;
    real_ip_recursive on;
    ...
}
 	The set_real_ip_from lines define which source IPs are trusted (requests from those IPs are assumed to be your proxy and the X-Forwarded-For from them is accepted). The real_ip_header specifies which header carries the real client IP. real_ip_recursive on; is useful if there are multiple proxies in chain – it will take the last non-trusted address in the list[16][17]. After this, $remote_addr and $binary_remote_addr in logs and limit modules will reflect the actual client.
•	PROXY protocol: Alternatively, some load balancers (e.g., AWS Network LB) use the PROXY protocol to send client IP. NGINX can be configured to accept this (listen 443 proxy_protocol; and real_ip_header proxy_protocol;). This is only if your environment uses it – if so, ensure NGINX is compiled with --with-http_realip_module and configured appropriately.
•	Ensure backend sees real IP: The earlier mentioned proxy_set_header X-Real-IP $remote_addr; (after applying real_ip module, this becomes the client IP) and X-Forwarded-For propagation means your upstream application receives the client IP. Always confirm that your app isn’t just seeing the proxy’s IP. For example, in an API behind NGINX, log the client IP to verify it’s working. If not, adjust real_ip settings as above.
Containerized NGINX Best Practices
When running NGINX in containers (Docker, Kubernetes, etc.), incorporate these best practices for security and reliability:
- Use official images and pin versions: Base your deployment on the official NGINX Docker image (or distribution-provided images) for a trustworthy source. Pin the image to a specific version/tag rather than latest to avoid untested changes on redeploy[18]. For example, use nginx:1.25.2-alpine (or whichever version is current stable) instead of nginx:latest. This guarantees consistency across environments and prevents surprise breaks from upstream changes. Regularly update the image to pull in security fixes, but do so in a controlled manner.
- Run as non-root if possible: By default, NGINX in Docker runs as root (to bind port 80/443). However, running as root in a container is still a risk. If you can, run it as a non-root user: e.g., use an image that supports dropping privileges or modify the Dockerfile to use USER nginx (after ensuring permissions). You might need to use higher ports and have the orchestrator map them to 80/443, or grant NET_CAP_BIND_SERVICE capability. Running as non-root mitigates the impact of a container compromise.
- Read-only filesystem: Make container filesystems read-only whenever feasible. NGINX does not need to write to most of its filesystem except for caching or logs (which can be redirected). You can mount volumes for /var/cache/nginx and /var/log/nginx and then run the rest of the container as read-only. Setting the root filesystem of the container to read-only greatly reduces the attack surface (an attacker can’t tamper with binaries or config if they somehow get in)[19]. Many orchestrators allow enabling this via a securityContext or docker run flag (--read-only).
- Health checks: Define a HEALTHCHECK for your NGINX container. For example, in Docker you might add to your Dockerfile:

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s \  
  CMD wget -qO- http://localhost/health || exit 1
(assuming you have a /health endpoint or even just hitting “/” or “/nginx_status” if enabled). In Kubernetes, set up liveness and readiness probes similarly (HTTP check on a path or TCP socket check). This ensures the container orchestrator can detect if NGINX is not functioning (and restart or stop routing to it).
- Resource limits: In containers, configure CPU/memory limits to prevent NGINX from exhausting host resources under high load. NGINX is efficient, but under extreme load or DoS it could use a lot of memory for buffers or many worker processes. Setting reasonable limits protects the host. Also consider using worker_processes auto; in nginx.conf to scale workers to available CPUs (the default in official image).
- Logging in containers: It’s common to direct NGINX logs to stdout/stderr (the official image does this by default for access and error log). Ensure your logging configuration in nginx.conf is compatible with container logging (or symlink /var/log/nginx/access.log to /dev/stdout). This way, Docker/K8s can capture logs. When using fail2ban with containers, note that fail2ban usually runs on the host or a privileged container, reading logs – you might instead use cloud-native solutions or NGINX’s built-in limit modules as described.
- Environment configuration: Avoid baking secrets (like TLS private keys) into images. Use Kubernetes secrets or mounted volumes for certificates and keys. Also externalize configuration where needed (for example, mount your nginx.conf and site config into the container, or use environment variables with a templating entrypoint if you need dynamic config on startup). This makes your deployments more flexible and secure.
Validation and Testing Checklist
After applying configuration changes or setting up a new NGINX server, go through this checklist to validate everything is working and secure:
1.	Syntax Check: Run nginx -t to test the configuration for syntax errors or typos before reloading or restarting NGINX[20]. This catches mistakes without downtime. Example output: “syntax is ok” and “test is successful.” Fix any errors indicated before proceeding.
2.	Reload NGINX Safely: Instead of full restart, use zero-downtime reload if NGINX is already running. For instance, nginx -s reload or systemctl reload nginx (on systems with systemd) will apply the new config without dropping connections. Ensure the reload actually succeeded (check systemctl status nginx or error log for any issues loading the new config).
3.	Verify HTTP/HTTPS Reachability: Use tools like curl from the server and an external location to ensure the site is serving content:
4.	curl -I http://your-domain should return a 301 redirect to HTTPS (if HSTS is enabled, you might test in a fresh environment or with a tool that ignores HSTS to see the redirect).
5.	curl -k -I https://your-domain (using -k to ignore cert for test or better, provide CA path) should return a 200 OK or expected result and show the security headers (check that Strict-Transport-Security, etc., appear as configured).
6.	In a browser, navigate to the site and ensure the SSL lock icon shows with no certificate warnings. Check the certificate details (correct domain, not expired, proper chain). Also inspect response headers in browser dev tools for the security headers.
7.	TLS Configuration Test: Use OpenSSL to simulate client handshake, e.g.: openssl s_client -connect your-domain:443 -servername your-domain and examine the output. It should show that only TLSv1.2/1.3 are accepted, the chosen cipher is strong, and OCSP stapling is working (look for "OCSP Response Status: successful" if you configured ssl_stapling). Alternatively, run an external SSL scan (Qualys SSL Labs or similar) to get a detailed report of your TLS config.
8.	Functional Tests: Browse to the site or API and exercise its functions. If it’s a reverse proxy, ensure that routes are correctly forwarded to backends. Look at NGINX’s error log (/var/log/nginx/error.log) for any runtime errors or warnings, and the access log to confirm requests are logging with correct status codes and client IPs.
9.	Caching behavior: If caching is configured, test it. Request a resource twice and see if the second response includes your cache indicator (e.g., an X-Cache-Status header or just faster response). You can also check NGINX’s cache directory to see if files are being written. For static file caching, check the Cache-Control and Expires headers on responses to ensure they match what you set.
10.	Rate limit and access control tests: Deliberately exceed your rate limit (e.g., send more than 5 requests/sec to a limited endpoint) and confirm NGINX returns 503 Service Unavailable or the status you expect. For connection limits, you might use a tool or script to open multiple connections. Also try accessing any URL that should be restricted (like /admin from an unauthorized IP) to ensure it’s blocked (should get 403 Forbidden or an auth prompt, etc., as configured).
11.	WebSocket test (if applicable): Use a WebSocket client (for example, wscat or a small script) to ensure you can establish a WebSocket connection through NGINX. If using wscat: wscat -c wss://your-domain/socket/ (for a wss endpoint) and verify you can send/receive messages. If issues, revisit the Upgrade header config or timeouts.
12.	Monitor and tune: Finally, monitor the NGINX instance over time. Use curl http://localhost/nginx_status (if you enabled stub_status) or tools like nginx-prometheus-exporter for metrics. Check that your worker process count is appropriate and CPU/memory usage are within limits. Adjust worker_connections (default 1024, often adequate) if you expect many simultaneous clients – remember max connections = worker_processes * worker_connections (though for reverse proxy each client may also use a connection to upstream).
By following the above checklist, you validate that your NGINX is not only configured correctly, but also performing as intended (serving content, enforcing security, and handling load). Always perform these checks after changes to catch issues early.
✅ Do’s and ❌ Don’ts Summary
To conclude, here’s a quick rundown of do’s and don’ts when configuring NGINX:
✅ DO:
•	Use HTTPS (TLS) with strong settings – enable HTTP/2, TLS 1.2/1.3, and strong ciphers[2].
•	Implement redirects and HSTS – ensure all HTTP is redirected to HTTPS, use HSTS to enforce it.
•	Tune performance – enable keepalives, use worker_processes auto, and optimize worker_connections for your needs. Use sendfile, gzip compression, and caching of static content for speed.
•	Use caching and rate limiting wisely – cache static and even dynamic content (where safe) to reduce load, and rate limit critical paths to mitigate abuse.
•	Preserve client IPs – pass through or reconstruct the real client IP for accurate logging and to allow upstream services to apply IP-based rules.
•	Add security headers – instruct browsers to protect your users (HSTS, XFO, CSP, etc.) and disable server tokens to not leak versions.
•	Set timeouts and limits – protect against slow clients and resource exhaustion by setting sensible timeouts (client_body_timeout, etc.)[11] and connection limits.
•	Harden the container/OS – run NGINX with least privilege, consider read-only filesystems and dropping root in containers[19], and keep the OS patched.
•	Monitor and log – separate access and error logs, monitor them, and use tools or fail2ban to automatically ban malicious actors[15]. Regularly review logs for anomalies.
•	Test configuration changes – always nginx -t before reloading, and use canary deployments or staging environments to test major changes in a safe manner.
❌ DON’T:
•	Don’t use weak or outdated crypto – avoid old SSL/TLS protocols, weak ciphers, or self-signed certificates in production (users will see warnings). Do not disable TLS 1.3 without cause (it offers better security and performance).
•	Don’t leave HTTP open – serving unencrypted traffic or allowing content on both HTTP/HTTPS without redirect can expose users to attacks. Also avoid mixed content (serve all resources over HTTPS).
•	Don’t ignore buffer and body size limits – the default client_max_body_size (1M) might be too low for file uploads; set it appropriately. Conversely, don’t set it unreasonably high without knowing impact.
•	Don’t cache sensitive data – never cache pages that include user-specific info, authentication tokens, or personal data. Also, don’t use proxy_cache on login or payment pages.
•	Don’t overload one server block – split configs for clarity. For example, don’t cram 50 upstreams and dozens of locations in a single file without comments. This makes maintenance hard.
•	Don’t use the latest Docker image tag in production[18] – it can change unexpectedly. Pin to a version and update intentionally.
•	Don’t run containers with unnecessary privileges – avoid running NGINX (or any service) as root in a container if not needed. Do not mount sensitive host directories into the container unless required.
•	Don’t forget to update – running outdated NGINX or not applying updates can leave you vulnerable. Stay on a maintained release and patch regularly (including renewing certs).
•	Don’t test on production – avoid making config changes live on prod without testing. NGINX can often reload without dropping connections, but a faulty config could still cause an outage if not caught by -t or staging tests.
•	Don’t assume defaults are always optimal – while NGINX defaults are good, your use-case may require tuning (e.g., file upload sizes, timeout values for very slow backends, etc.). Review and adjust critical directives for your scenario instead of sticking blindly to defaults.
Resources
For further reading and reference, see the following official documentation and guides:
•	NGINX Official Documentation – 【31†Nginx Official Documentation (Configuration)†nginx.org】 and specifically the admin guide and module references for directives used above.
•	NGINX Web Server Book – a comprehensive guide on NGINX configuration and optimization (covers advanced topics like tuning kernel parameters, etc.).
•	Mozilla SSL Configuration Generator – 【34†Mozilla SSL Configuration Generator†ssl-config.mozilla.org】 for recommended TLS settings and cipher lists, updated as standards evolve.
•	NGINX Rate Limiting Blog – 【10†Rate Limiting with NGINX (NGINX Blog)†blog.nginx.com】 for an in-depth discussion on limiting requests and connections (useful to fine-tune your settings beyond basics).
•	Mitigating DDoS with NGINX – techniques to protect against DoS attacks, including timeouts and limits (NGINX blog)[11][8].
•	NGINX Beginner’s Guide – Covers basic setup, and the official NGINX Admin Guide for performance tuning.
•	Docker Security (OWASP Cheat Sheet) – if deploying in containers, see OWASP’s Docker security guide for general container hardening practices.
•	WebSocket Proxying Docs – 【18†NGINX WebSocket Proxying†nginx.org】 official doc on how NGINX handles WebSocket upgrade, for more nuanced scenarios.
By adhering to the best practices above and referencing the documentation for specific needs, you can confidently configure NGINX to be secure, efficient, and reliable in a modern production environment.
________________________________________
[1] Configuring HTTPS servers
https://nginx.org/en/docs/http/configuring_https_servers.html
[2] [3] [5] [6] [13] [14] [20] nginx-configuration - Agent Skill by aj-geddes | SkillsMP
https://skillsmp.com/skills/aj-geddes-useful-ai-prompts-skills-nginx-configuration-skill-md
[4] WebSocket proxying
https://nginx.org/en/docs/http/websocket.html
[7] [8] [11] [12] Mitigating DDoS Attacks with NGINX – NGINX Community Blog
https://blog.nginx.org/blog/mitigating-ddos-attacks-with-nginx-and-nginx-plus
[9] [10] Configure Security Headers • Nginx & Apache »
https://webdock.io/en/docs/how-guides/security-guides/how-to-configure-security-headers-in-nginx-and-apache?srsltid=AfmBOorIGMygUh8KrqogkAaUGou_cfwzJhcCmbIzwlBKJ4nVIPwA8L-3
[15] How To Protect an Nginx Server with Fail2Ban on Ubuntu 20.04 | DigitalOcean
https://www.digitalocean.com/community/tutorials/how-to-protect-an-nginx-server-with-fail2ban-on-ubuntu-20-04
[16] [17] Module ngx_http_realip_module
https://nginx.org/en/docs/http/ngx_http_realip_module.html
[18] Best practices | Docker Docs
https://docs.docker.com/build/building/best-practices/
[19] Security recommendations | NGINX Documentation
https://docs.nginx.com/nginx-ingress-controller/configuration/security/
