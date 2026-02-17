---
name: socket-io
description: >
  Design, secure, scale, and operate production Socket.IO systems. Use when defining transport
  strategy, authentication/authorization middleware, rooms and namespaces, delivery guarantees,
  horizontal scaling with adapters, performance tuning, reliability patterns, load testing, and
  incident runbooks for real-time Node.js services.
---

# Socket.IO

## Workflow
1. Confirm workload shape, latency SLOs, and reliability requirements.
2. Choose transport strategy and proxy/load balancer behavior.
3. Implement connection and event-level security controls.
4. Define room/namespace topology and message contract conventions.
5. Establish delivery semantics (at-most-once vs stronger guarantees).
6. Select scaling adapter strategy and session affinity model.
7. Tune performance with measured data (not defaults-by-folklore).
8. Ship with test coverage, operations runbooks, and upgrade policy.

## Preflight (Ask / Check First)
- Expected concurrent connections and event throughput profile.
- Need for polling fallback vs WebSocket-only operation.
- Auth model and event-level authorization requirements.
- Critical message reliability requirements (ack/retry/persistence).
- Deployment environment (single host, Kubernetes, multi-region).
- Broker options for adapters (Redis, Streams, managed broker).
- Security constraints (CORS/origin policy, TLS, dependency policy).

## Operating Principles
- Treat Socket.IO as session/routing infrastructure, not durable messaging by default.
- Keep security layered: connect middleware + packet middleware + payload validation.
- Use explicit CORS/origin strategy and remember CORS limitations on WebSocket.
- Keep heartbeat/proxy timeouts aligned to avoid false disconnect storms.
- Prefer measured tuning over speculative parameter changes.
- Keep reconnection and backpressure behavior explicit in client/server logic.

## Baseline Architecture and Defaults
- Start with explicit server config, even when using defaults.
- Keep `transports` intentional (`polling + websocket` or websocket-only).
- Keep `maxHttpBufferSize` bounded and documented per workload.
- Keep `perMessageDeflate` off unless bandwidth gains justify CPU/memory cost.
- Align reverse proxy idle timeouts with heartbeat window.

### Baseline Config Checks
```bash
node -e "console.log(require('socket.io/package.json').version)"
node -e "console.log(process.versions.node)"
```

```bash
rg -n "pingInterval|pingTimeout|maxHttpBufferSize|transports|perMessageDeflate" src
```

## Security Best Practices
- Authenticate at connection time with `io.use()` middleware.
- Authorize and rate-limit per event using `socket.use()` packet middleware.
- Validate payload schema and enforce allowlists per event contract.
- Restrict oversized payloads and expensive event shapes.
- Use TLS for all browser-facing production traffic.
- Track security advisories for Socket.IO, Engine.IO, parser, and `ws` ecosystem.

### CORS and Reachability Guardrails
- Configure explicit origin allowlists; never rely on permissive wildcard defaults.
- Remember CORS only affects browser polling requests.
- Use `allowRequest`/edge controls for true reachability policy.
- Treat WebSocket handshake policy separately from browser CORS assumptions.

### Security Validation Loop
1. Test unauthorized connect attempts.
2. Test unauthorized event attempts for each privileged event.
3. Fuzz payload shape and size boundaries.
4. Verify rate limits under burst and distributed attack patterns.
5. Confirm TLS and origin restrictions in real deployment path.

## Performance and Latency Engineering
- Optimize event framing and avoid excessive tiny packet emission.
- Batch/coalesce high-frequency updates where user experience allows.
- Choose parser/encoding strategy after profiling payload patterns.
- Keep compression policy explicit (`httpCompression`, `perMessageDeflate`).
- Consider WebSocket engine/native addons where throughput demands justify.
- Measure ACK timeout rates and tail latency, not only averages.

### Latency Measurement Pattern
- Use ACK + timeout for round-trip latency measurements.
- Track per-event success, timeout, and retry histogram.
- Distinguish transient network failures from server processing delays.

## Scalability and Topology Patterns
- Use rooms for routing subsets of clients.
- Use namespaces for coarse authorization and domain separation.
- Avoid namespace explosion without clear ownership boundaries.
- For multi-node with polling enabled, enforce sticky sessions.
- Choose adapter pattern by environment and failure tolerance.

### Adapter Selection Guidance
- Single-host multi-process: cluster adapter + sticky strategy.
- Multi-host standard fan-out: Redis adapter (simple pub/sub).
- Multi-host with disconnection tolerance: Redis Streams adapter.
- Managed-cloud broker adapters when infra policy requires.

### Scaling Risks to Monitor
- Session-affinity misconfiguration causing `Session ID unknown` errors.
- Broker outage causing partial fan-out visibility.
- Recovery-enabled mass reconnect memory spikes.
- OS file descriptor and ephemeral port exhaustion.

## Reliability and Delivery Semantics
- Assume default delivery is at-most-once.
- Use ACK + timeout for critical request/response events.
- Use client retries only for idempotent events with dedup keys.
- Build server-side persistence and replay for stronger guarantees.
- Keep resynchronization path for failed recovery scenarios.

### Stronger Delivery Pattern
1. Assign unique event IDs for deduplication.
2. Persist outbound critical events.
3. Track client last-seen offset.
4. Replay missed events after reconnect/auth.
5. Expire replay window by business policy and storage budget.

## Connection Recovery and Reconnect Storms
- Enable connection state recovery only after load testing.
- Keep `maxDisconnectionDuration` bounded.
- Use `skipMiddlewares` only when auth state safety is proven.
- Guard against reconnect spikes with jitter/backoff and edge throttling.
- Use `socket.conn` signals to implement backpressure policy.

## API Design and Contract Governance
- Use stable event naming conventions (`domain:action`).
- Keep payload schemas versioned and backward-compatible where possible.
- Prefer explicit error envelopes and ACK semantics.
- Separate privileged/admin flows into dedicated namespaces.
- Document room membership and authorization invariants.

### API Contract Checklist
- Event names and payload schemas versioned.
- AuthZ rule for every event documented.
- ACK behavior and timeout expectations defined.
- Idempotency/dedup strategy defined where retries exist.
- Deprecation process defined for event evolution.

## Testing, Load Testing, and CI/CD
- Unit-test handlers and validation logic.
- Integration-test namespaces, rooms, auth, and adapter fan-out.
- Load-test with realistic reconnect/burst scenarios.
- Include failure-injection for broker outage and proxy resets.
- Gate deploys on performance and reliability budgets.

### Typical Validation Commands
```bash
npm run lint
npm run test
npm run test:integration
```

```bash
npm run test:load
npm run build
```

## Operations and Incident Runbook
- Maintain dashboards for connections, emits, ACK timeouts, and disconnect reasons.
- Monitor process memory, event loop lag, and broker health.
- Keep structured logs for connect/disconnect/auth failures.
- Prepare playbooks for reconnect storms, broker degradation, and proxy misconfig.
- Rehearse rollback of transport/adapter changes.

### Incident Triage Sequence
1. Check edge/proxy timeout and upgrade metrics.
2. Check adapter broker health and pub/sub lag.
3. Check auth middleware error-rate spikes.
4. Check memory growth and reconnect burst profile.
5. Apply rate-limit/backoff controls and recover gradually.

## Compatibility and Upgrade Strategy
- Keep Socket.IO and adapter versions aligned intentionally.
- Track protocol and parser changes before upgrades.
- Stage upgrades with canary traffic and rollback plan.
- Validate client compatibility (browser/mobile/Node).
- Patch known advisories quickly with controlled rollout.

## Common Failure Modes
- Assuming CORS protects WebSocket connections.
- Missing per-event authorization despite connect-time auth.
- Enabling retries without idempotency/dedup, causing duplicates.
- Polling enabled in multi-node without stickiness.
- Aggressive heartbeat tuning causing unnecessary churn.
- Recovery configuration causing memory spikes during mass reconnect.
- Over-broad namespaces/rooms creating authorization drift.

## Definition of Done
- Security controls cover connect-time and event-time enforcement.
- Reliability semantics for critical events are explicit and tested.
- Scaling topology and adapter behavior are validated under load.
- Performance budgets and latency metrics are tracked.
- Operations runbooks and rollback paths are current.

## References
- `references/socket.io.md`

## Reference Index
- `rg -n "Baseline architecture|Default operational parameters|Transport options" references/socket.io.md`
- `rg -n "Security best practices|Authentication|authorization|CORS|TLS" references/socket.io.md`
- `rg -n "Performance and latency|perMessageDeflate|wsEngine|heartbeat" references/socket.io.md`
- `rg -n "Scalability|Namespaces|rooms|sticky sessions|adapter" references/socket.io.md`
- `rg -n "Reliability|delivery semantics|ack|retries|recovery" references/socket.io.md`
- `rg -n "API design|testing|load testing|operations|upgrade" references/socket.io.md`

## Quick Questions (When Stuck)
- Is this issue transport/proxy, adapter/broker, or app-level contract design?
- Do we need stronger delivery than at-most-once for this event?
- Are retries safe here without deduplication?
- Is polling fallback worth stickiness complexity in this environment?
- What metric proves this tuning change helped rather than shifted failure mode?
