# design-review — Review Feature Proposals Against Architecture

## When to Use
Architect uses this when a feature proposal appears in `#features` or `#architecture` needing review.

## Inputs
- Feature proposal (GH Discussion or spec document)
- `docs/adr/` — established architectural decisions
- `agents/architect/context.md` — bounded context map, maturity stage
- Codebase structure for pattern verification

## Procedure

1. **Read the proposal** — Understand what's being proposed, which bounded contexts it touches, and what new concepts it introduces.
2. **Check bounded context alignment** — Does the proposal respect existing BC boundaries? Does it introduce cross-BC coupling?
3. **Verify layer boundaries** — Confirm domain logic stays in domain layer:
   - No business rules in controllers or handlers
   - No infrastructure concerns (DB, HTTP) in domain
   - Ports defined in domain, adapters in infrastructure
4. **Check coupling** — Look for:
   - Direct imports between modules that should communicate through ports
   - Shared mutable state across contexts
   - God objects or services that span multiple aggregates
5. **Verify pattern compliance** — Cross-reference against existing ADRs:
   - Does it follow established patterns?
   - If it deviates, is there a good reason (potential new ADR)?
6. **Assess maturity fit** — Is the proposed complexity appropriate for the current maturity stage? Flag over-engineering at early stages.
7. **Post review comment** on the GH Discussion:

```markdown
---
agent: architect
type: review
severity: {info | warning | critical}
tags: [design-review]
mentions: [{proposal-author}]
requires: {info | action}
---

## Design Review: {proposal title}

**Verdict:** {approved | needs-changes | blocked}

### Findings
{numbered list of findings, each with:}
1. **{finding title}** — {description}
   - Impact: {what breaks or degrades if not addressed}
   - Suggestion: {how to fix}
   - Reference: {ADR or pattern link if applicable}

### Summary
{one paragraph — overall assessment and next steps}
```

## Output Format
- Review comment on the GH Discussion with verdict: approved, needs-changes, or blocked

## Rules
- Always reference specific ADRs when blocking — never just say "this is wrong"
- Explain what pattern it violates and suggest the correct approach
- "Approved" can still include minor suggestions (info severity)
- "Needs-changes" means the design is sound but has fixable issues
- "Blocked" means fundamental architectural problems — requires redesign
