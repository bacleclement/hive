# incident-security — Handle Security Incidents

## When to Use
Sec Chief uses this when a security incident is detected — critical CVE discovered, breach suspected, secret exposed, or unauthorized access identified.

## Inputs
- Incident trigger (CVE alert, secret scan finding, anomaly detection, human report)
- `agents/sec-chief/context.md` — credential inventory, system architecture
- Relevant scan reports (vuln-scan, secret-scan, auth-audit)

## Procedure

1. **Assess scope** — Determine what's exposed and who's affected:
   - What data/systems are compromised?
   - Which users/organizations are impacted?
   - How long has the exposure existed?
   - Is the vulnerability actively being exploited?

2. **Contain immediately** — Stop the bleeding before investigating:
   - Disable compromised credentials / rotate keys
   - Block suspicious IPs or sessions if applicable
   - Revoke affected tokens
   - Rotate ALL potentially compromised credentials, not just confirmed ones

3. **Create incident thread** in `#incidents`:

```markdown
---
agent: sec-chief
type: incident
severity: critical
tags: [security-incident, P0-security]
mentions: [@cto, @{affected-agents}]
requires: action
---

## Security Incident: {brief description}

### Severity: P0-security
### Status: {investigating | contained | mitigating | resolved}
### Detected: {timestamp}

### Scope
- **What:** {what is compromised}
- **Who:** {affected users/orgs}
- **Since:** {estimated exposure window}
- **Exploited:** {yes/no/unknown}

### Containment Actions Taken
1. {action taken with timestamp}
2. {action taken with timestamp}

### Timeline
| Time | Event |
|------|-------|
| {time} | {event} |
```

4. **Notify human immediately** — Level 3 notification. Security incidents always require human awareness.

5. **Coordinate fix** — Work with relevant agents:
   - Sr-backend for code fixes
   - DevOps for infrastructure changes
   - Architect if architectural changes needed

6. **Verify fix** — Confirm the vulnerability is resolved:
   - Re-run the relevant scan (vuln-scan, secret-scan, auth-audit)
   - Verify containment actions are effective
   - Confirm no residual exposure

7. **Write postmortem** — Update the incident thread:

```markdown
### Postmortem

**Root cause:** {what caused the incident}
**Impact:** {what was affected and for how long}
**Resolution:** {what was done to fix it}
**Prevention:** {what changes prevent recurrence}

### Action Items
1. {action — owner — deadline}
2. {action — owner — deadline}
```

8. **Update context** — Record the incident in `agents/sec-chief/context.md` for future reference.

## Output Format
- Incident thread in `#incidents` with P0-security severity
- Level 3 human notification
- Postmortem appended to incident thread after resolution

## Rules
- Security incidents are always Level 3 (human notification) — no exceptions
- Containment before investigation — stop the damage first, understand it second
- Rotate ALL potentially compromised credentials, not just confirmed ones
- Never downgrade a security incident severity without human approval
- Document every action with timestamps in the incident timeline
- Postmortem is mandatory — every incident gets one, no matter how small
