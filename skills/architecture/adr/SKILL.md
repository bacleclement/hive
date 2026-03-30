# adr — Document Architectural Decisions

## When to Use
Architect uses this when an architectural decision needs to be documented — new pattern adoption, technology choice, or structural change.

## Inputs
- Decision context (from discussion, proposal, or implementation need)
- `docs/adr/` — existing ADRs for sequential numbering and precedent check
- `agents/architect/context.md` — current maturity stage
- Related GH Discussions if any

## Procedure

1. **Identify context** — What problem or question triggered this decision? Read related discussions and code to understand the full scope.
2. **List alternatives** — Enumerate at least 2 options. For each, document:
   - How it works
   - Trade-offs (pros/cons)
   - Effort estimate
   - Maturity stage fit (is this appropriate complexity for current stage?)
3. **Recommend one** — State the chosen option and why it wins given current constraints.
4. **Determine next number** — Scan `docs/adr/` for the highest existing number, increment by 1.
5. **Write the ADR** — Create `docs/adr/{number}-{slug}.md` following Nygard format:

```markdown
# {number}. {Title}

Date: {YYYY-MM-DD}

## Status

{proposed | accepted | deprecated | superseded}

## Context

{What is the issue that we're seeing that is motivating this decision or change?}
{Include maturity stage context — what stage are we at and how does that affect the decision?}

## Decision

{What is the change that we're proposing and/or doing?}

## Consequences

{What becomes easier or more difficult to do because of this change?}
{List both positive and negative consequences.}
```

6. **Post summary** — Post a summary to `#decisions`:

```markdown
---
agent: architect
type: decision
severity: info
tags: [adr]
requires: info
---

## ADR-{number}: {Title}

**Status:** {status}
**Decision:** {one-sentence summary}
**Key trade-off:** {main thing we gain vs. main thing we give up}
**Full ADR:** docs/adr/{number}-{slug}.md
```

## Output Format
- ADR file at `docs/adr/{number}-{slug}.md`
- Summary post to `#decisions`

## Rules
- Every ADR gets a sequential number — no gaps, no duplicates
- Status is one of: proposed, accepted, deprecated, superseded
- Include maturity stage context in the Context section
- When superseding an ADR, update the old ADR's status to "Superseded by ADR-{new-number}"
- Never skip the alternatives analysis — even obvious decisions benefit from documented reasoning
