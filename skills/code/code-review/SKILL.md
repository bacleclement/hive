# code-review — Review Code Before Merge

## When to Use
Sr Backend uses this to self-review before requesting merge, or to review another agent's code.

## Inputs
- Branch/worktree with changes
- plan.md / spec.md for acceptance criteria
- Project's code-standards.md

## Procedure

1. **Diff review** — `git diff main...HEAD`
   - Every changed file, every line

2. **Checklist**
   | Check | Pass? |
   |-------|-------|
   | Tests exist for all new behavior | |
   | Tests actually test the right thing (not just coverage) | |
   | No business logic in infrastructure layer | |
   | No infrastructure leaking into domain | |
   | Repository queries scoped by organizationId | |
   | Error handling — no swallowed errors | |
   | No hardcoded values that should be config | |
   | Naming follows project conventions | |
   | No TODO/FIXME left without ticket reference | |
   | No console.log / debug artifacts | |
   | Imports are direct (no barrel files) | |

3. **Architecture check**
   - Does this change respect bounded context boundaries?
   - Are domain invariants enforced in aggregates (not handlers)?
   - Does the data flow follow ports & adapters?

4. **Output**
   If issues found → fix them (self-review) or post findings (peer review)
   If clean → post to `#daily-standup`: "Code review passed. Ready for merge."

## Rules
- Never skip this skill before requesting merge
- Self-review catches 80% of issues — take it seriously
- If unsure about an architecture question → post to `#architecture` for Architect
