# compliance-check — OWASP Top 10 Compliance Scorecard

## When to Use
Sec Chief runs this monthly as a full audit to assess the project's security posture against OWASP Top 10.

## Inputs
- `agents/sec-chief/context.md` — maturity stage, previous compliance scores
- Codebase source files
- Previous month's scorecard for trend comparison

## Procedure

1. **Determine scope** based on maturity stage:
   - **Stage 2** — Focus on top 5: Injection, Broken Auth, Sensitive Data Exposure, Broken Access Control, Vulnerable Components
   - **Stage 3+** — Full OWASP Top 10 (add: Security Misconfiguration, XSS, Insecure Deserialization, Insufficient Logging, XXE)

2. **Audit each category**:

   **A1 — Injection**
   - Parameterized queries used everywhere (no string concatenation in SQL)
   - ORM/query builder used consistently
   - Input validation on all external inputs

   **A2 — Broken Authentication**
   - Strong password policies enforced
   - Session management is secure (JWT expiry, refresh rotation)
   - Multi-factor authentication available (Stage 3+)

   **A3 — Sensitive Data Exposure**
   - Data encrypted at rest and in transit
   - No sensitive data in logs
   - PII handling follows data classification policy

   **A4 — Broken Access Control**
   - RLS enabled on all tables
   - organizationId scoping on all queries
   - Role-based access enforced at API level

   **A5 — Vulnerable Components**
   - Cross-reference with latest vuln-scan results
   - No known critical/high CVEs unpatched

   **A6 — Security Misconfiguration** (Stage 3+)
   - Default credentials removed
   - Error handling doesn't expose stack traces
   - Unnecessary features/ports disabled

   **A7 — XSS** (Stage 3+)
   - Output encoding on all rendered user input
   - CSP headers configured
   - DOM-based XSS mitigations in place

   **A8 — Insecure Deserialization** (Stage 3+)
   - Input validation on all deserialized data
   - Type checking enforced on API payloads

   **A9 — Insufficient Logging** (Stage 3+)
   - Auth events logged (login, logout, failed attempts)
   - Access control failures logged
   - Logs are tamper-evident and retained

   **A10 — XXE** (Stage 3+)
   - XML external entity processing disabled
   - XML parsers configured securely

3. **Score each category**: pass / partial / fail

4. **Compare to last month** — Note improvements and regressions.

5. **Post scorecard** to `#security`:

```markdown
---
agent: sec-chief
type: report
severity: {info | warning | critical}
tags: [compliance]
requires: {info | action}
---

## OWASP Compliance — {month year}

### Scorecard
| # | Category | Score | Last Month | Trend |
|---|----------|-------|------------|-------|
| A1 | Injection | {pass/partial/fail} | {prev} | {up/down/same} |
| A2 | Broken Auth | {pass/partial/fail} | {prev} | {up/down/same} |
| ... | ... | ... | ... | ... |

### Overall: {X}/{Y} passing ({percentage}%)

### Failures Requiring Action
{numbered list with category, specific finding, and fix recommendation}

### Improvements Since Last Month
{list of categories that improved and what was done}
```

## Output Format
- Compliance scorecard to `#security`

## Rules
- Scope the checklist to maturity stage — Stage 2 covers A1-A5, Stage 3+ covers all 10
- Score honestly — partial means some controls exist but gaps remain
- Every "fail" must include a concrete fix recommendation
- Track trends month over month — regression is always flagged
