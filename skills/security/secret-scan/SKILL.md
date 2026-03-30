# secret-scan — Detect Exposed Secrets in Codebase

## When to Use
Sec Chief runs this daily or before any deploy to verify no secrets are committed to the repository.

## Inputs
- `agents/sec-chief/context.md` — known false positives, previous scan results
- `clients/{project}/adapters.json` — adapter config for `security.secrets`
- Git repository and history

## Procedure

1. **Run secret detection** — Execute gitleaks or equivalent:
   ```
   adapter:security.secrets --mode detect
   ```
   Scan the current codebase for patterns matching API keys, tokens, passwords, connection strings.

2. **Scan .env files** — Check for committed environment files:
   - `.env` files present in the repository (not in `.gitignore`)
   - `.env.example` files containing real values instead of placeholders

3. **Check for hardcoded secrets** — Scan source code for:
   - Hardcoded API keys or tokens (string literals matching key patterns)
   - Hardcoded database connection strings
   - Hardcoded passwords or passphrases
   - Base64-encoded secrets

4. **Verify environment variable usage** — Confirm all secrets are loaded from environment:
   - Check `process.env` usage for expected secrets
   - Verify no fallback to hardcoded defaults for secrets

5. **Scan git history** — Check for previously committed secrets:
   ```
   adapter:security.secrets --mode history
   ```
   Secrets removed from current code but still in git history are still exposed.

6. **Filter false positives** — Cross-reference against known false positives in `context.md`.

7. **Post findings** to `#security`:

```markdown
---
agent: sec-chief
type: report
severity: {info | critical}
tags: [secret-scan]
requires: {info | action}
---

## Secret Scan — {date}

### Summary
- Secrets found: {count}
- In current code: {count}
- In git history only: {count}
- False positives filtered: {count}

### Findings
| File | Line | Type | Location |
|------|------|------|----------|
| {path} | {line} | {api-key | token | password | connection-string} | {current | history} |

### Required Actions
1. {action — e.g., "Rotate API key for service X, found in path/to/file.ts:42"}
2. {action}

### Git History Note
{if secrets found in history: "Git history contains exposed secrets. Consider using git-filter-repo to remove them."}
```

8. **Alert if found** — Any secret finding is critical. Trigger immediate rotation.

## Output Format
- Secret scan report to `#security` (file paths and line numbers only)
- Level 2 human notification if secrets found

## Rules
- Any found secret is a critical finding
- Rotation of compromised credentials is required immediately
- NEVER post actual secret values in the report — only file paths and line numbers
- Git history secrets are just as critical as current code secrets
- Update `context.md` with any new confirmed false positives
