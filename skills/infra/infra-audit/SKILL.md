# infra-audit — Infrastructure Health Audit

## When to Use
DevOps uses this when running the every-4h health check or during weekly deep audit.

## Inputs
- Railway service status
- Supabase project status
- DNS and SSL configuration
- Resource utilization metrics (CPU, memory, disk)

## Procedure

1. Check Railway service status (all services healthy)
2. Check Supabase project status (database accessible, auth functional)
3. Verify DNS resolution for all configured domains
4. Check SSL certificate expiry dates — warn if < 30 days
5. Check resource utilization (CPU, memory, disk) for each service
6. Check for unused services or resources (cost waste)
7. Post report to #ops:

```markdown
---
agent: devops
type: report
severity: {info | warning | critical}
tags: [infra-audit]
requires: {ack | action}
---

## Infrastructure Audit

### Railway: {healthy | degraded | down}
### Supabase: {healthy | degraded | down}
### DNS: {all resolving | issues found}
### SSL Expiry: {nearest expiry date and domain}
### Resource Utilization:
- CPU: {%}
- Memory: {%}
- Disk: {%}
### Unused Resources: {list or "none"}
### Status: {healthy | attention needed | critical}
```

## Output Format
Infrastructure audit report posted to #ops (see template above).

## Rules
- SSL cert expiry < 14 days is critical — escalate to CTO
- SSL cert expiry < 30 days is a warning
- Resource utilization > 80% is a warning
- Resource utilization > 90% is critical — recommend scaling action
- Weekly deep audit includes all checks. 4h check focuses on service status and resource utilization
