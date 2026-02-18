---
name: security-best-practices
description: Perform language and framework specific security best-practice reviews and suggest improvements. Trigger only when the user explicitly requests security best practices guidance, a security review/report, or secure-by-default coding help. Trigger only for supported languages (python, javascript/typescript, go). Do not trigger for general code review, debugging, or non-security tasks.
---

# Security Best Practices

Use this skill only when the user explicitly requests security best practices or a security review.

## Workflow
1. Identify primary languages and frameworks in scope.
2. Load matching guidance from `references/`.
3. Apply secure-by-default patterns for new code.
4. If asked, produce a prioritized security report.
5. Offer fixes one finding at a time with minimal regression risk.

## Identify Stack
- Inspect the repo for language and framework evidence.
- Focus on core backend and frontend stacks only.
- If the stack is unclear, ask a single clarifying question.

## Reference Files
- Read only the relevant `references/*.md` files.
- File naming pattern:
  - `<language>-general-<stack>-security.md`
  - `<language>-<framework>-<stack>-security.md`
- If no references exist, say so and proceed with general best practices.

## Modes of Use
- **Secure-by-default coding**: apply best practices while implementing features.
- **Passive detection**: flag high-impact issues noticed during work.
- **Full report**: if the user requests a formal security review.

## Report Mode
- Write a Markdown report to `security_best_practices_report.md` unless the user specifies a path.
- Include line numbers for code references.
- Prioritize by severity and impact.

## Severity Buckets
- Critical: cross-tenant access, RCE, credential exposure.
- High: auth bypass with constraints, stored XSS, SSRF to internal.
- Medium: limited XSS, missing rate limits on sensitive flows.
- Low: hygiene issues without a clear exploit path.

### Report Template
```markdown
## Executive Summary

## Findings (Prioritized)
1. [ID] Title (Severity)
   - Evidence:
   - Impact:
   - Recommendation:

## Verification Steps

## Assumptions / Constraints
```

## Fixes
- Ask before starting fixes unless the user already requested them.
- Fix one finding at a time.
- Keep changes scoped; avoid collateral refactors.
- Run existing tests if available.

## Guardrails
- Do not over-report low-impact issues.
- Do not treat missing TLS in local dev as a finding.
- Be explicit about assumptions and evidence gaps.

## Common High-Impact Areas
- Authn/authz checks at every access path.
- Input validation and output encoding.
- Secrets handling and logging hygiene.
- Dependency and supply-chain risk.
- Rate limiting and abuse prevention.

## Severity Guidance
- **Critical**: unauthenticated RCE, cross-tenant data access, key exfiltration.
- **High**: auth bypass with user interaction, stored XSS, SSRF to metadata.
- **Medium**: reflected XSS with constraints, missing rate limits on sensitive flows.
- **Low**: best-practice gaps without clear exploit paths.

## Verification Checklist
- Reproduce the issue safely in non-prod if possible.
- Add regression tests for the exploit path.
- Confirm logs/alerts capture the blocked attempt.

## Evidence Rules
- Prefer concrete code and config references over general claims.
- When uncertain, label it as a hypothesis and explain what would confirm it.

## Output Expectations
- Clear summary of top risks.
- Specific, actionable recommendations.
- Next steps and verification guidance.

## Definition of Done
- The requested security guidance or report is delivered.
- The user knows what to fix and how to validate it.
