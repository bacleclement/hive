# vuln-scan — Scan Dependencies for Vulnerabilities

## When to Use
Sec Chief runs this daily at 6am, weekly for deep dive, or on demand when a new CVE is reported.

## Inputs
- `agents/sec-chief/context.md` — existing CVE inventory, maturity stage, response SLAs
- `clients/{project}/adapters.json` — adapter config for `security.audit`
- `package.json` / `pnpm-lock.yaml` — dependency tree

## Procedure

1. **Run audit** — Execute dependency vulnerability scan:
   ```
   adapter:security.audit
   ```
   Parse results by severity: critical, high, medium, low.

2. **Cross-reference inventory** — Compare findings against existing CVE inventory in `agents/sec-chief/context.md`:
   - Identify NEW vulnerabilities (not seen before)
   - Identify RESOLVED vulnerabilities (previously tracked, now gone)
   - Identify UNCHANGED vulnerabilities (still present)

3. **Check for patches** — For each new vulnerability:
   - Is a patched version available?
   - Is the patch a minor/patch bump (low risk) or major (breaking)?
   - Is the vulnerable dependency direct or transitive?

4. **Classify response urgency** based on maturity stage:
   | Severity | Stage 2 SLA | Stage 3 SLA | Stage 4 SLA |
   |----------|-------------|-------------|-------------|
   | Critical | Immediate | Immediate | Immediate |
   | High | 48h | 48h | 24h |
   | Medium | 7 days | 48h | 48h |
   | Low | Next sprint | 7 days | 7 days |

5. **Post report** to `#security`:

```markdown
---
agent: sec-chief
type: report
severity: {info | warning | critical}
tags: [vuln-scan]
mentions: [{sr-backend if critical}]
requires: {info | action}
---

## Vulnerability Scan — {date}

### Summary
- Critical: {count} ({new count} new)
- High: {count} ({new count} new)
- Medium: {count} ({new count} new)
- Low: {count} ({new count} new)

### New Vulnerabilities
| CVE | Severity | Package | Patched Version | SLA |
|-----|----------|---------|-----------------|-----|
| {id} | {sev} | {pkg} | {version or "none"} | {deadline} |

### Resolved Since Last Scan
{list of CVEs no longer present}

### Action Required
{prioritized list of fixes with commands, e.g., pnpm update {pkg}@{version}}
```

6. **Alert if critical** — Critical CVEs trigger Level 2 notification (notify human).
7. **Update inventory** — Write current CVE state to `agents/sec-chief/context.md`.

## Output Format
- Vulnerability report to `#security`
- Level 2 human notification if critical CVEs found
- Updated CVE inventory in `agents/sec-chief/context.md`

## Rules
- Critical CVEs trigger immediate alert (Level 2 notify human)
- High CVEs must be fixed within 48h
- Medium CVEs follow maturity-stage SLA (7 days at Stage 2, 48h at Stage 3+)
- Never suppress or downplay a finding — report everything, classify accurately
