---
name: security
description: Security analysis and security engineering workflows for software, infrastructure, and cloud systems. Use when triaging/analyzing security findings or incidents, validating exploitability and impact, performing threat modeling/design reviews, writing security reports, recommending remediations with verification steps, or reviewing security controls and secure coding practices (authn/z, input/output handling, secrets, crypto/data protection, dependency/supply chain, infra/CI/CD hardening, monitoring/IR).
---

# Security

## Overview

Use this skill to turn ambiguous security questions into a scoped, evidence-backed assessment and a concrete plan: what happened/exists, why it matters, how to fix it, and how to verify.

This skill replaces the former `security-analysis` and `security-best-practices` skills.

Keep outputs decision-ready:
- Avoid hand-wavy language; state assumptions and confidence.
- Prefer root-cause fixes and defense-in-depth over narrow patches.
- Provide verification steps that a non-security engineer can actually run.

## Intake (Ask First)

Ask only what you need to remove ambiguity, then proceed.

Always capture:
- What is the question: "Is this real?", "How bad is it?", "How do we fix?", "Is the fix sufficient?"
- What is the system boundary: repo/service, endpoint, job, account, environment (prod/stage/dev).
- What is at stake: data classes (credentials/PII/payment/IP), privileged actions, availability impact.
- What evidence exists: alert/scan output, logs, screenshots, code links, configs, timestamps, PoC.
- What constraints exist: time window, ability to test, non-disruption requirements, approval needed.
- Who owns remediation and who decides risk acceptance.

If incident-like:
- Ask for timeframe, detection source, known indicators (IPs, users, tokens, hashes), and containment actions already taken.

If code/design review:
- Ask for threat model assumptions (external attacker vs insider), trust boundaries, and intended authorization model.

## Workflow Decision Tree

1. If the user asks "is this exploitable / how severe / what is the impact / write a report": run **Finding Analysis**.
2. If the user asks "what happened / did we get popped / how far did it go": run **Incident Analysis**.
3. If the user asks "is this design safe / what threats exist / what controls are required": run **Threat Modeling / Design Review**.
4. If the user asks "how do we secure this / review controls / secure coding guidance": run **Controls Review / Hardening**.
5. If the user asks "is this fix enough / how do we prove it": run **Remediation Verification** (often after one of the above).

## Default Output Templates

Use one of these shapes unless the user requests a specific format.

### Finding / Vulnerability Report

```markdown
## Summary
- What: [one sentence]
- Where: [system/component]
- Impact: [who/what affected]
- Urgency: [why now]

## Scope and Assumptions

## Evidence
- [links/log snippets/commands run]

## Exploitability
- Preconditions:
- Attack path (high level):
- Constraints/mitigations:

## Impact
- Data at risk:
- Privilege gained:
- Blast radius:

## Root Cause

## Risk Rating
- Severity:
- Business impact:
- Confidence:

## Remediation Plan
1. Immediate mitigations:
2. Durable fix (root cause):
3. Defense in depth:

## Verification
- Repro steps before fix:
- Tests/validation after fix:
- Monitoring/detections to add:

## Follow-ups
```

### Incident Analysis Report

```markdown
## Summary

## Timeline (UTC)

## Scope and Affected Assets

## Indicators and Evidence

## Root Cause / Initial Access Vector

## Impact Assessment

## Containment / Eradication / Recovery

## Verification and Monitoring

## Lessons Learned / Preventative Actions
```

## Finding Analysis

Use this for: scanner findings, bug reports, suspicious behavior that is not yet confirmed as an incident, and "is this exploitable?" questions.

1. Restate the question as a testable claim.
2. Identify the entry point(s) and trust boundary crossings.
3. Confirm the issue:
   - Prefer a safe reproduction in a non-production environment.
   - If repro is not possible, reason from code/config + known primitives and state assumptions.
4. Determine exploitability:
   - List preconditions (auth required? network position? user interaction? special headers?).
   - Describe the minimal attacker capability required.
   - Note mitigations/compensating controls (WAF, RBAC, network segmentation, rate limits).
5. Determine impact:
   - Describe what the attacker can do (read/modify/delete data, execute code, impersonate users, move laterally).
   - Enumerate affected assets and data classes.
6. Identify root cause:
   - Prefer the "why" (missing authz check, unsanitized sink, unsafe deserialization), not just the symptom.
7. Rate risk with context:
   - Separate technical severity from business impact.
   - State confidence level and evidence gaps.
8. Recommend remediation:
   - Provide an immediate mitigation if exposure is active.
   - Provide a durable fix that removes the primitive.
   - Add defense-in-depth controls where cheap and effective.
9. Define verification:
   - Provide negative tests (prove exploit no longer works).
   - Provide regression tests and monitoring/detections.

For deeper methods, checklists, and templates, read `references/security-analysis.md`.

## Common Finding Primitives (Confirm, Fix, Verify)

Use this as a fast mapping from "what class of bug is this?" to "what must be true for it to be real?" and "what does a durable fix look like?"

- Broken access control / IDOR: Confirm cross-tenant or cross-user access by changing identifiers. Fix with object-level authorization checks on every access path plus policy tests. Verify with negative tests for cross-tenant reads/writes and logs/alerts for abnormal 403/401 patterns.
- SQL/NoSQL injection: Confirm attacker-controlled data reaches a query interpreter. Fix with parameterized queries, strict allowlists for dynamic identifiers, and removal of dangerous operators. Verify with regression tests using representative payloads and query logging/monitoring for anomalies.
- XSS (reflected/stored/DOM): Confirm untrusted data reaches an HTML/JS execution context. Fix with context-aware output encoding, safe templating, and HTML sanitization for rich text; add CSP where practical. Verify by ensuring payloads render inert and by adding tests around sinks.
- CSRF: Confirm a state-changing endpoint can be triggered cross-site with ambient credentials. Fix with CSRF tokens and/or SameSite cookies plus origin checks; avoid GET side effects. Verify with cross-origin request attempts and automated tests.
- SSRF: Confirm user-controlled URLs/hosts can be fetched by backend infrastructure. Fix with strict allowlists, DNS/IP pinning defenses, egress controls, and blocking link-local/metadata ranges. Verify by attempting to reach internal addresses (including metadata IPs) and observing blocks.
- Command injection: Confirm untrusted data reaches a shell or command interpreter. Fix by avoiding shell invocation, using safe argument arrays, and strict allowlists. Verify with payload attempts and unit tests for argument escaping behavior.
- Path traversal / file read: Confirm untrusted path segments can escape intended directories. Fix with path normalization, allowlisted filenames, and storage outside web roots. Verify with traversal payloads and tests on different OS/path separators.
- Unsafe deserialization: Confirm attacker-controlled bytes are deserialized into objects with gadget behavior. Fix by removing unsafe formats, using safe schema-based serialization, and disabling polymorphism/gadgets. Verify by banning vulnerable APIs and adding regression tests.
- Authn/authz logic flaws: Confirm bypass via workflow manipulation (password reset, token exchange, MFA enrollment, account linking). Fix with explicit state machines, step-up auth, and server-side invariants. Verify with integration tests covering alternate paths and replay attempts.
- Token/JWT issues: Confirm missing or incorrect validation (signature, audience, issuer, nonce, expiration) or weak key management. Fix with strict validation, key rotation, and short lifetimes. Verify with negative tests (bad signature, wrong aud/iss, expired token) and telemetry.
- CORS misconfiguration: Confirm cross-origin reads/writes are permitted beyond intent. Fix with strict allowlists, avoid reflecting Origin, and avoid credentialed wildcard patterns. Verify with browser-based tests from untrusted origins.
- Rate limiting / abuse: Confirm brute force or enumeration is feasible at needed scale. Fix with rate limits per identity and per IP, backoffs, and abuse detection. Verify by load tests and alert thresholds.

## Incident Analysis

Use this for: active exploitation, suspicious access, confirmed compromise, or alert-driven investigations.

1. Confirm whether it is an incident:
   - Validate the signal (false positive vs true positive).
   - Identify the detection source and what it can/cannot see.
2. Preserve evidence first when feasible:
   - Avoid destructive actions that erase logs/artifacts unless containment requires it.
   - Normalize time (UTC), note clock drift risks.
3. Scope the blast radius:
   - Identify affected accounts, tokens, hosts, services, data stores, and time window.
   - Identify trust boundary crossings and privileged pivots.
4. Build a timeline:
   - Anchor on high-confidence events (auth logs, admin actions, deploys, config changes).
5. Determine initial access and root cause:
   - Identify the exploited primitive (credential theft, exposed secret, RCE, SSRF to metadata, misconfig).
6. Assess impact:
   - Determine data access/exfil indicators and integrity impact.
   - Distinguish confirmed vs suspected.
7. Recommend containment/eradication/recovery:
   - Rotate/kill credentials and tokens, patch the exploited vector, remove persistence.
   - Ensure changes are coordinated with owners and do not cause avoidable downtime.
8. Verify and harden:
   - Add detections for recurrence.
   - Add preventative controls that would have blocked the path.
9. Write a concise report:
   - Provide what happened, what is known/unknown, what is being done, and what to do next.

For deeper incident analysis guidance, read `references/security-analysis.md`.

## Threat Modeling / Design Review

Use this for: new systems, major changes, auth model changes, new integrations, and "is this safe by design?" questions.

1. Define the boundary and assets:
   - List components, data stores, identities, and external dependencies.
2. Draw the data flows and trust boundaries:
   - Identify where untrusted input enters and where privileged actions occur.
3. Identify abuse cases:
   - Focus on the highest-value assets and the easiest attack paths.
4. Enumerate threats (use STRIDE as a scaffold):
   - Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege.
5. Map existing controls and gaps:
   - Identify missing controls and weak assumptions.
6. Recommend mitigations:
   - Prioritize by risk reduction and implementation cost.
7. Define verification:
   - Tests, configuration checks, and telemetry required to ensure controls remain effective.

For deeper threat modeling practices, read `references/security-analysis.md`.

## Controls Review / Hardening

Use this for: secure-by-default guidance, security-focused code/design reviews, and control gap assessments.

1. Restate the threat model:
   - Identify likely attacker types and realistic access.
2. Apply secure defaults:
   - Enforce least privilege.
   - Enforce deny-by-default.
   - Add defense in depth to avoid single points of failure.
3. Review control families (pick only what is relevant):
   - Input/output: validate on server, encode outputs by context, use parameterized queries.
   - Authn: MFA for privileged paths, safe OAuth/OIDC flows, strong password hashing.
   - Authz: object-level checks, consistent policy enforcement, prevent IDOR.
   - Sessions: secure cookie flags, rotation, timeouts, logout invalidation.
   - Secrets: never commit, use a vault/secret manager, rotate and audit.
   - Data protection: TLS everywhere, at-rest encryption, key management, minimize retention.
   - Dependencies: lockfiles/pinning, SBOM/SCA, patching cadence, provenance.
   - Infra/CI/CD: environment isolation, least-privilege service accounts, hardened baselines.
   - Monitoring/IR: centralized logs, alerting, playbooks, tabletop drills.
4. Make recommendations actionable:
   - Provide specific config/code changes and owners.
   - Include verification steps for each recommendation (tests, queries, dashboards, alerts).

For detailed control guidance and checklists, read `references/security-best-practices.md`.

## Remediation Verification

Use this whenever a fix is proposed or implemented.

1. Confirm the fix addresses the root cause (not just the specific payload).
2. Validate negative paths:
   - Attempt the exploit again (safely) and prove it fails.
   - Add regression tests that would fail if the bug reappears.
3. Validate defense-in-depth:
   - Confirm relevant authz checks exist at every boundary.
   - Confirm rate limits, logging, and safe defaults are in place where appropriate.
4. Validate operational readiness:
   - Confirm monitoring/detections cover the relevant signals.
   - Confirm incident response steps are documented for this class of issue.

## Risk Rating (Use a Simple, Explicit Rationale)

Avoid arguing about a single number. Explain the drivers:
- Exposure: internet-facing vs internal, reachable surface area, auth required.
- Exploitability: complexity, prerequisites, reliability, detectability.
- Impact: data sensitivity, privilege gained, integrity/availability impact, blast radius.
- Compensating controls: WAF, segmentation, strong authz, monitoring, rapid rotation.

If the user requests a formal scoring system (e.g., CVSS), provide it, but still explain what it means operationally.

## Reference Docs (Use Efficiently)

The bundled references are intentionally comprehensive. Prefer targeted searches over reading the whole file.

Use `references/security-analysis.md` for:
- Evidence collection, integrity, and incident/finding analysis templates.
- Prioritization and risk-rating guidance.
- Common failure modes and checklists.

Use `references/security-best-practices.md` for:
- Secure coding, authn/z, secrets, crypto/data protection, CI/CD, and monitoring best practices.
- OWASP-aligned vulnerability prevention guidance.

Example targeted searches:
```bash
rg -n "chain of custody|timeline|triage" skills/security/references/security-analysis.md
rg -n "Broken access control|IDOR|CSRF|XSS|SSRF" skills/security/references/security-best-practices.md
```

## Output Expectations (Use by Default)

- Executive summary: what it is, impact, urgency, and who should act.
- Evidence-backed narrative: what you observed, how you concluded it, and confidence level.
- Risk rating: rationale, affected assets/users/data, and exposure conditions.
- Remediation plan: prioritized steps, owners, and expected timelines.
- Verification: how to prove the fix works and did not regress (tests/detections).
- Follow-ups: prevention, monitoring, and process improvements.

## Extended Guidance
Use this when risk acceptance or compliance requirements apply.

## Risk Acceptance Checklist
- Document the business rationale and the decision owner.
- Define compensating controls and monitoring.
- Set a review date for re-evaluation.
