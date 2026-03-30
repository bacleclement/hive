# onboard-test — Simulate New Developer Onboarding

## When to Use
DevRel uses this monthly to test the developer onboarding experience end-to-end by following setup instructions from scratch.

## Inputs
- README and setup documentation
- Clean environment (or as close to fresh as possible)

## Procedure

1. Start from the README — read the "Getting Started" or equivalent section
2. Follow every instruction step by step:
   - Clone the repo (or simulate from a fresh directory)
   - Install dependencies as documented
   - Configure environment variables as documented
   - Run database setup/migrations as documented
   - Start the application as documented
   - Run tests as documented
3. At each step, note:
   - **Pass**: Instruction worked as written
   - **Unclear**: Instruction was ambiguous or assumed knowledge
   - **Outdated**: Instruction references something that changed
   - **Wrong**: Instruction produces an error
   - **Missing**: A necessary step is not documented
4. Attempt to make a simple code change and verify it works
5. Document all gaps found
6. Post onboarding test report to `#daily-standup`:

```markdown
---
agent: devrel
type: report
severity: info
tags: [onboard-test, monthly]
channel: #daily-standup
---

## Onboarding Test — {month}

### Overall: {pass | partial | fail}
### Time to first run: {duration}

| Step | Instruction | Result | Issue |
|------|-------------|--------|-------|
| 1    | Clone repo  | Pass   | —     |
| 2    | npm install | Fail   | Missing node version note |
| ...  | ...         | ...    | ...   |

### Gaps Found ({count})
- {gap 1 — file, line, issue}
- {gap 2}

### Recommendations
- {fix 1}
- {fix 2}
```

## Output Format
Markdown test report posted to `#daily-standup` with step-by-step results, gaps, and fix recommendations.

## Rules
- Actually run the commands — do not just read the instructions and guess
- Use the documented prerequisites only — do not bring in undocumented tribal knowledge
- If a step fails, note the error and attempt to work around it, then continue
- Test on the documented platform/OS — if docs say "macOS", test on macOS
- Compare with previous month's report to check if past gaps were fixed
