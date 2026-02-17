---
name: performance-optimization
description: >
  Guide performance optimization across frontend, backend, and database layers. Trigger when the
  user is reviewing, debugging, or planning to improve performance (speed, latency, throughput, or
  memory usage). Avoid using if the query is not about performance.
---

# Performance Optimization

## Workflow
1. Define the goal and the metric (p95 latency, throughput, memory, Core Web Vitals, cost).
2. Establish a baseline (before measurements).
3. Profile to find bottlenecks (do not guess).
4. Apply targeted optimizations (small, reversible steps).
5. Re-measure and compare to baseline (quantify wins and trade-offs).
6. Roll out safely and monitor for regressions.

## Quick Intake (Ask/Confirm)
- What metric is failing and where is it measured (RUM vs synthetic vs APM)?
- What is the target (SLO) and who is impacted?
- What changed recently (deploys, schema, traffic, feature flags)?
- What environment is representative (prod-like data, device/network class)?

## Principles
- Measure first, optimize second.
- Prefer simple, high-impact fixes (N+1 queries, caching, payload size).
- Protect correctness: add tests/guards when changing performance-critical code.
- Optimize the critical path (80/20); avoid micro-optimizing cold code.

## Frontend Checklist
- Reduce payload size (minify, compress, eliminate unused code/CSS).
- Reduce requests (bundle wisely, preload critical assets).
- Avoid main-thread long tasks; use workers where appropriate.
- Use browser profiling (DevTools, Lighthouse, WebPageTest).
- Validate on realistic devices/networks (throttle; mid-tier mobile).

## Backend Checklist
- Remove obvious hot-path inefficiencies (algorithmic complexity, repeated work).
- Reduce I/O: batch calls, reuse connections, add pooling.
- Add caching where safe (define invalidation strategy).
- Add timeouts and concurrency limits.
- Avoid retry storms (use backoff + jitter; cap retries).

## Database Checklist
- Identify slow queries (logs, EXPLAIN/ANALYZE).
- Add indexes for real access patterns.
- Fix N+1 patterns via joins/eager loading/batching.
- Avoid over-fetching (pagination, select only needed columns).

## Validation
- Benchmark/measure in representative environments.
- Track regressions in CI (performance tests) when the repo supports it.
- Monitor in production (APM, RUM, error rate, saturation signals).

## Common Pitfalls (Call Out)
- "Optimizing" without a baseline (no proof of improvement).
- Moving the bottleneck (faster app, slower DB) without noticing.
- Caching without invalidation (stale data, cache stampede).
- Adding complexity for marginal wins (maintainability regression).

## Communication Template (Before/After)

```text
Baseline: <metric> was <X> (p95/avg) in <env>, impacting <users>.
Change: <what you changed> (why it targets the bottleneck).
Result: <metric> is now <Y> (Δ% improvement). Trade-offs: <cpu/mem/cost>.
Validation: <tests/profiles/load test> and <monitoring>.
Rollout: <flag/canary> and rollback plan <...>.
```

## References
- `references/performance-optimization.md`

## Extended Guidance
Use this when the problem is complex, performance data is noisy, or multiple layers are involved.

## Evidence Collection Checklist
- Capture baseline p50/p95/p99 before changing anything.
- Record workload shape (QPS, concurrency, payload size).
- Identify the path (endpoint, job, query, component) and isolate it.
- Capture resource metrics (CPU, memory, disk, network).
- Record environment (prod/stage) and config flags.

## Profiling and Tracing Workflow
1. Form a hypothesis about the bottleneck.
2. Collect a trace or profile (CPU, memory, IO).
3. Validate with a targeted change.
4. Re-measure with the same workload.

## Optimization Priority Rules
- Fix correctness and excessive work before micro-optimizations.
- Reduce data movement before reducing CPU.
- Cache only after you know the key and invalidation strategy.

## Common Failure Modes
- Optimizing without baselines (no proof of improvement).
- Measuring with a different workload than production.
- Adding caching without invalidation or TTLs.
- Collapsing multiple changes into one commit (hard to attribute impact).

## Example Hypothesis Log
```text
Hypothesis: Slow p95 caused by N+1 DB queries on /api/orders.
Evidence: Trace shows 32 sequential queries per request.
Change: Batch query with IN clause.
Result: p95 dropped from 900ms -> 210ms at 50 QPS.
```

## Reference Index
- `rg -n "Profiling|tracing" references/performance-optimization.md`
- `rg -n "Caching|invalidation" references/performance-optimization.md`
- `rg -n "Database|indexes" references/performance-optimization.md`
- `rg -n "Frontend|Core Web Vitals" references/performance-optimization.md`

## Profiling Commands (Examples)
```bash
rg -n "p95|p99|latency" logs/*.log
```

## Change Discipline
- Make one performance change per commit when possible.
- Capture before/after metrics and note confidence.

## Reference Index (Expanded)
- `rg -n "baseline|measurement" references/performance-optimization.md`
- `rg -n "bottleneck|hypothesis" references/performance-optimization.md`

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
- `rg -n "Checklist|checklist" references/performance-optimization.md`
- `rg -n "Example|examples" references/performance-optimization.md`
- `rg -n "Workflow|process" references/performance-optimization.md`
- `rg -n "Pitfall|anti-pattern" references/performance-optimization.md`
- `rg -n "Testing|validation" references/performance-optimization.md`
- `rg -n "Security|risk" references/performance-optimization.md`
- `rg -n "Configuration|config" references/performance-optimization.md`
- `rg -n "Deployment|operations" references/performance-optimization.md`
- `rg -n "Troubleshoot|debug" references/performance-optimization.md`
- `rg -n "Performance|latency" references/performance-optimization.md`
- `rg -n "Reliability|availability" references/performance-optimization.md`
- `rg -n "Monitoring|metrics" references/performance-optimization.md`
- `rg -n "Error|failure" references/performance-optimization.md`
- `rg -n "Decision|tradeoff" references/performance-optimization.md`
- `rg -n "Migration|upgrade" references/performance-optimization.md`
