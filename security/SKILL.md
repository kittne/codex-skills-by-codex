---
name: security
description: Security analysis and security engineering workflows for software, infrastructure, and cloud systems. Use when triaging/analyzing security findings or incidents, validating exploitability and impact, performing threat modeling/design reviews, writing security reports, recommending remediations with verification steps, or reviewing security controls and secure coding practices (authn/z, input/output handling, secrets, crypto/data protection, dependency/supply chain, infra/CI/CD hardening, monitoring/IR).
---

# Security

Use this skill to produce scoped, evidence-backed assessments and concrete remediation plans.

## Core Principles
- State assumptions and confidence.
- Prefer root-cause fixes and defense in depth.
- Provide verification steps that are easy to run.

## Intake (Ask Only What You Need)
- Question type: exploitable, severity, incident, design review, or fix validation.
- System boundary: repo/service, environment, and owner.
- Data at risk: credentials, PII, payments, IP, availability.
- Evidence: logs, scans, code, configs, timestamps, PoC.
- Constraints: time, testing limits, approvals.

If incident-like, ask for timeframe, indicators, and containment actions.
If design review, ask for trust boundaries and intended auth model.

## Workflow Decision Tree
1. Exploitability/severity/report: run **Finding Analysis**.
2. "What happened?": run **Incident Analysis**.
3. Design safety: run **Threat Modeling / Design Review**.
4. Secure-by-default guidance: run **Controls Review / Hardening**.
5. "Is this fix enough?": run **Remediation Verification**.

## Default Output Templates

### Finding / Vulnerability Report
```markdown
## Summary
- What:
- Where:
- Impact:
- Urgency:

## Scope and Assumptions

## Evidence

## Exploitability
- Preconditions:
- Attack path:
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
1. Immediate mitigation:
2. Durable fix:
3. Defense in depth:

## Verification
- Negative tests:
- Regression tests:
- Monitoring to add:
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

## Lessons Learned
```

## Finding Analysis
1. Restate the question as a testable claim.
2. Identify entry points and trust boundary crossings.
3. Confirm safely in non-prod if possible.
4. Determine exploitability (preconditions, attacker capability, mitigations).
5. Determine impact (data, privilege, blast radius).
6. Identify root cause (missing control, unsafe sink, misconfig).
7. Rate risk with confidence and evidence gaps.
8. Recommend remediation (immediate + durable + defense in depth).
9. Define verification steps.

## Incident Analysis
1. Validate the signal (false positive vs true positive).
2. Preserve evidence (avoid destructive changes).
3. Scope blast radius and time window.
4. Build a timeline anchored on high-confidence events.
5. Identify initial access vector and root cause.
6. Assess impact and exposure.
7. Recommend containment/eradication/recovery.
8. Add detections to prevent recurrence.

## Threat Modeling / Design Review
1. Define boundary and assets.
2. Map data flows and trust boundaries.
3. Identify highest-value abuse cases.
4. Enumerate threats (STRIDE as a scaffold).
5. Map existing controls and gaps.
6. Recommend mitigations and verify.

## Controls Review / Hardening
- Enforce least privilege and deny-by-default.
- Validate inputs, encode outputs, and avoid unsafe sinks.
- Secure authn/authz and session handling.
- Protect secrets and sensitive data (in transit and at rest).
- Manage dependencies and patch cadence.
- Harden CI/CD and infrastructure baselines.
- Add monitoring and incident playbooks.

## Remediation Verification
- Reproduce the issue pre-fix where safe.
- Add regression tests for the exploit path.
- Verify telemetry and alerts for recurrence.

## Common Finding Primitives (Fast Mapping)
- Broken access control/IDOR: object-level checks + negative tests.
- Injection (SQL/NoSQL/command): parameterization + allowlists.
- XSS: context-aware encoding + sanitization + CSP.
- CSRF: tokens + SameSite + origin checks.
- SSRF: strict allowlists + network egress controls.
- Auth flow flaws: explicit state machines + step-up auth.
- Token/JWT issues: strict validation + rotation + short TTLs.
- CORS misconfig: strict allowlists and avoid credentialed wildcards.
- Rate limiting/abuse: per-identity limits + monitoring.

## References
- `references/security-analysis.md`
- `references/security-best-practices.md`

## Definition of Done
- Findings are scoped and evidence-backed.
- Remediation steps and verification are clear.
- Assumptions and confidence are explicit.
