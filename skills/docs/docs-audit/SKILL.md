# docs-audit — Audit Documentation for Staleness and Gaps

## When to Use
DevRel uses this weekly to scan all documentation for stale content, missing docs, broken links, and inconsistencies with the current codebase.

## Inputs
- All documentation files (README, API docs, guides, ADRs)
- Current codebase (for cross-referencing endpoints, features, configs)
- Previous audit report (for tracking fixes)

## Procedure

1. Scan all documentation files in the repo
2. Check for staleness:
   - References to removed or renamed features/endpoints
   - Version numbers that no longer match
   - Screenshots or examples using outdated UI/API
3. Check for gaps:
   - New features (merged in last 2 weeks) without corresponding docs
   - API endpoints present in code but missing from API docs
   - Configuration options not documented
4. Check for broken links:
   - Internal links pointing to moved/deleted pages
   - External links returning 404
5. Check for inconsistencies:
   - Docs contradicting current code behavior
   - Conflicting information across different doc files
6. Post audit report to `#daily-standup`:

```markdown
---
agent: devrel
type: report
severity: info
tags: [docs-audit, weekly]
channel: #daily-standup
---

## Docs Audit — {week}

### Stale Content ({count})
| File | Issue | Line/Section |
|------|-------|--------------|
| ...  | ...   | ...          |

### Missing Docs ({count})
| Feature/Endpoint | Added | Docs Status |
|------------------|-------|-------------|
| ...              | ...   | Missing     |

### Broken Links ({count})
| File | Link | Status |
|------|------|--------|
| ...  | ...  | 404    |

### Inconsistencies ({count})
| File A | File B | Conflict |
|--------|--------|----------|
| ...    | ...    | ...      |
```

## Output Format
Markdown audit report posted to `#daily-standup` with categorized issues and file/line references.

## Rules
- Always cross-reference docs against the actual codebase — docs say X, code does Y means docs are wrong
- Track fix rate week over week — are flagged issues getting resolved?
- Prioritize: broken functionality docs > missing docs > stale content > cosmetic
- Never auto-fix docs — flag issues for humans to review and correct
