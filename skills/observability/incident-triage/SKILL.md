# incident-triage — Classify Severity, Create Incident Thread

## When to Use
Obs Chief uses this when health-check detects a warning or critical anomaly.

## Inputs
- Anomaly data from health-check
- `agents/obs-chief/context.md` — historical incidents for comparison

## Procedure

1. **Classify severity**
   | Signal | Severity |
   |--------|----------|
   | Single metric 20-50% off baseline | P2 — Warning |
   | Multiple metrics off OR single > 50% | P1 — High |
   | Service down OR data loss risk | P0 — Critical |

2. **Identify scope**
   - Which orgs affected?
   - Which feature/endpoint?
   - Since when? (first occurrence timestamp)

3. **Identify likely cause**
   - Recent deploy? (check `agents/devops/context.md`)
   - External dependency? (check Tavily/OpenAI/Supabase status)
   - Traffic spike? (check request volume)
   - Code regression? (check git log)

4. **Create incident thread** in `#incidents`:

```markdown
---
agent: obs-chief
type: alert
severity: {warning|critical}
tags: [{affected-system}, incident]
mentions: [{relevant agents}]
requires: action
---

## INCIDENT: {title}

### Severity: {P0|P1|P2}
### Started: {timestamp}
### Affected: {orgs/users/endpoints}

### Symptoms
{What's broken — with data}

### Likely Cause
{Best hypothesis with evidence}

### Recommended Action
{What should happen next — who should do what}

### Timeline
- {HH:MM} — First anomaly detected
- {HH:MM} — Incident created
```

5. **Notify based on severity**
   | Severity | Notification |
   |----------|-------------|
   | P2 | GH Discussion only |
   | P1 | GH Discussion + Telegram |
   | P0 | GH Discussion + Telegram + Email (Level 3 escalation) |

6. **Tag responsible agents**
   | Likely cause | Tag |
   |-------------|-----|
   | Code regression | @sr-backend |
   | AI pipeline | @sr-ai |
   | Infra / deploy | @devops |
   | Security | @sec-chief |
   | External dependency | @cto (for decision) |

## Rules
- Never close an incident without a postmortem
- P0 incidents → CTO must acknowledge within 1 cycle
- Update the incident thread with every new finding (don't create new threads)
