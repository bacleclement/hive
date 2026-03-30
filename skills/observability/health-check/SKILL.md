# health-check — Full System Health Assessment

## When to Use
Obs Chief runs this every hour. DevOps can also invoke during deploy verification.

## Inputs
- `clients/{project}/adapters.json` — adapter config for observe.*
- `agents/obs-chief/context.md` — baselines for comparison

## Procedure

1. **LOGS** — Check recent production logs
   ```
   adapter:observe.logs --last 1h --severity error,warn
   ```
   - Count errors and warnings
   - Look for new error types (not seen in last 7d)
   - Check for repeated patterns (same error > 5 times)

2. **ERRORS** — Check error tracking
   ```
   adapter:observe.errors
   ```
   - New unresolved errors since last check?
   - Frequency spike on existing errors?
   - Any error affecting > 1 org?

3. **METRICS** — Check database and app health
   ```
   adapter:observe.metrics
   ```
   Run queries from adapters.json:
   - Active DB connections (vs baseline)
   - Error rate (vs baseline)
   - Enrichment success rate (vs baseline)
   - Table sizes (growth check)

4. **COMPARE** — Current vs baselines in context.md
   - Flag anything > 20% deviation
   - Flag any new error type
   - Flag connection count > 80% of pool max

5. **CLASSIFY**
   | Condition | Severity | Action |
   |-----------|----------|--------|
   | All within baseline | info | Brief "all clear" to #daily-standup |
   | One metric 20-50% off | warning | Detailed post to #daily-standup, tag relevant agent |
   | Multiple metrics off OR > 50% deviation | critical | Incident thread in #incidents + telegram alert |

6. **OUTPUT** — Post result to appropriate GH Discussion category

7. **UPDATE** — Write latest metrics to `agents/obs-chief/context.md` baselines

## Output Format

### All Clear
```markdown
---
agent: obs-chief
type: report
severity: info
requires: info
---

## Health Check — {timestamp} — All Clear

| Metric | Value | Baseline | Delta |
|--------|-------|----------|-------|
| Error rate | 2.1% | 2.0% | +0.1% |
| DB connections | 12 | 11 | +1 |
| Enrichment success | 97.5% | 97.8% | -0.3% |
```

### Warning/Critical — use incident-triage skill
