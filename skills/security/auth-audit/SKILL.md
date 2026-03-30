# auth-audit — Audit Authentication and Authorization

## When to Use
Sec Chief runs this during the weekly deep dive or after any auth-related code changes (middleware, RLS policies, JWT handling).

## Inputs
- `agents/sec-chief/context.md` — previous auth audit results, known exceptions
- `clients/{project}/adapters.json` — adapter config for `security.auth`
- Supabase RLS policies
- API route definitions and middleware configuration

## Procedure

1. **Audit RLS policies** — Review Supabase Row Level Security:
   ```
   adapter:security.auth --check rls
   ```
   - Verify every table has RLS enabled
   - Verify policies scope queries by `organizationId`
   - Check for overly permissive policies (e.g., `true` as policy expression)
   - Verify no table uses `security definer` functions that bypass RLS

2. **Audit JWT validation** — Review API middleware:
   - JWT is validated on every protected route
   - Token expiry is checked
   - Token signature is verified against the correct secret/key
   - Refresh token rotation is implemented correctly

3. **Scan for exposed endpoints** — Check all API routes:
   - List endpoints without auth middleware
   - Verify each unprotected endpoint is intentionally public (health checks, webhooks with their own auth)
   - Flag any endpoint that accesses user data without auth

4. **Review API key handling** — Check for:
   - API keys stored in environment variables (not hardcoded)
   - API keys not logged or exposed in error responses
   - Key rotation mechanism exists

5. **Post findings** to `#security`:

```markdown
---
agent: sec-chief
type: report
severity: {info | warning | critical}
tags: [auth-audit]
requires: {info | action}
---

## Auth Audit — {date}

### RLS Coverage
- Tables with RLS: {count}/{total}
- Tables missing RLS: {list or "none"}
- Policies scoped by organizationId: {count}/{total policies}

### Endpoint Protection
- Protected endpoints: {count}
- Intentionally public: {count} ({list})
- Unprotected (FINDING): {count} ({list with file paths})

### JWT Validation
- Status: {pass | partial | fail}
- Findings: {list if any}

### API Key Handling
- Status: {pass | partial | fail}
- Findings: {list if any}

### Critical Findings
{numbered list with file paths, description, and fix recommendation}
```

## Output Format
- Auth audit report to `#security`

## Rules
- Any table without RLS is a critical finding
- Any endpoint accessing user data without auth is critical
- Flag both findings with specific fix recommendations
- Known intentional exceptions (public health check, etc.) must be documented in `context.md` to avoid repeat flags
