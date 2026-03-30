# bounded-context-audit — Audit Bounded Context Integrity

## When to Use
Architect uses this during the weekly audit ceremony to verify bounded context boundaries are respected across the codebase.

## Inputs
- `agents/architect/context.md` — bounded context map, aggregate inventory
- Codebase source files across all BCs
- Previous audit report for trend comparison

## Procedure

1. **Review BC map** — Read the bounded context map in `agents/architect/context.md`. Confirm it reflects current modules.

2. **Scan for BC leaks** — Check each bounded context for boundary violations:
   - **Import leaks** — Direct imports crossing BC boundaries (not through ports)
   - **Shared mutable state** — Any state shared between BCs without event-based communication
   - **Direct DB queries** — Queries outside the owning context (e.g., BC-A querying BC-B's tables directly)

3. **Verify aggregate invariants** — For each aggregate:
   - Invariants are enforced in the aggregate itself, not in application handlers
   - No business rules scattered across services or controllers
   - Aggregate root is the only entry point for mutations

4. **Check organizationId scoping** — Verify all repository queries are scoped by `organizationId`:
   - Scan repository implementations for queries missing org scope
   - Check query handlers for unscoped data access
   - Verify multi-tenant isolation at the data layer

5. **Classify findings** by severity:
   | Severity | Criteria | Example |
   |----------|----------|---------|
   | critical | Runtime bug risk | Unscoped query leaking data across orgs |
   | warning | Maintainability risk | Invariant enforced in handler instead of aggregate |
   | info | Cleanup opportunity | Unused port interface |

6. **Post report** to `#architecture`:

```markdown
---
agent: architect
type: report
severity: {info | warning | critical}
tags: [bc-audit]
requires: {info | action}
---

## BC Audit — {date}

### Summary
- Bounded contexts audited: {count}
- Critical findings: {count}
- Warnings: {count}
- Info: {count}

### Findings

#### Critical
{numbered findings with file paths, description, and fix recommendation}

#### Warning
{numbered findings with file paths, description, and fix recommendation}

#### Info
{numbered findings — cleanup opportunities}

### Trend
{comparison to last audit — improving, stable, or degrading}
```

## Output Format
- BC audit report posted to `#architecture`

## Rules
- BC violations are architectural debt — always tracked, never ignored
- Severity classification: critical = runtime bug risk, warning = maintainability risk, info = cleanup opportunity
- Every finding includes a file path and a concrete fix recommendation
- organizationId scoping violations are always critical — they risk data leaks between tenants
