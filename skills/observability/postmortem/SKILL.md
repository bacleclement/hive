# postmortem — Structured Post-Incident Analysis

## When to Use
Obs Chief runs this after every incident is resolved. No exceptions.

## Inputs
- Incident thread from `#incidents`
- Agent comments and actions during incident
- Metrics data before/during/after

## Procedure

1. Compile timeline from incident thread
2. Identify root cause (not symptoms)
3. Assess impact
4. Document what worked and what didn't
5. Propose prevention actions

## Output Format

Post as final comment on the incident thread, then link from `#daily-standup`:

```markdown
## Postmortem — {incident title}

### Timeline
| Time | Event |
|------|-------|
| {HH:MM} | First anomaly detected by obs-chief |
| {HH:MM} | Incident thread created |
| {HH:MM} | Root cause identified |
| {HH:MM} | Fix deployed |
| {HH:MM} | Metrics returned to baseline |

### Duration: {total time}
### Affected: {orgs/users}

### Root Cause
{What actually caused the issue — technical detail}

### Impact
- Users affected: {n}
- Duration: {time}
- Data loss: {yes/no — detail if yes}

### What Worked
- {thing that helped}

### What Didn't Work
- {thing that should have been better}

### Prevention
| Action | Owner | Priority |
|--------|-------|----------|
| {what to do to prevent recurrence} | {agent} | {high/medium/low} |
```

## Rules
- Blameless — focus on systems, not agents
- Every postmortem must have at least one prevention action
- Prevention actions get posted as proposals in `#decisions` for CTO review
