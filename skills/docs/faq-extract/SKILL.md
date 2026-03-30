# faq-extract — Extract FAQs from Discussions and Support Tickets

## When to Use
DevRel uses this weekly to identify repeated questions across GH Discussions and support tickets, then add them to the FAQ docs.

## Inputs
- GH Discussion threads (all categories)
- Support ticket history (from the past week)
- Existing FAQ docs (to avoid duplicates)

## Procedure

1. Scan all GH Discussion threads from the past week
2. Scan support tickets resolved in the past week
3. Identify repeated questions — same topic asked 2+ times across any source
4. For each repeated question:
   - Write a clear, canonical version of the question
   - Write a concise answer (sourced from the best resolution)
   - Link to the original discussion(s) for context
5. Check existing FAQ docs — skip if already covered
6. Add new FAQ entries to the FAQ document
7. Post extraction report to `#daily-standup`:

```markdown
---
agent: devrel
type: report
severity: info
tags: [faq-extract, weekly]
channel: #daily-standup
---

## FAQ Extraction — {week}

### New FAQs Added ({count})
| Question | Times Asked | Sources |
|----------|-------------|---------|
| ...      | ...         | ...     |

### Already Covered ({count})
| Question | Existing FAQ Link |
|----------|-------------------|
| ...      | ...               |

### Observation
{any pattern — e.g., "3 questions this week about enrichment limits suggest docs gap"}
```

## Output Format
New entries added to FAQ docs + extraction report posted to `#daily-standup`.

## Rules
- Minimum 2 occurrences to qualify as FAQ — single questions are not patterns
- Always check existing FAQ before adding — no duplicates
- Write questions as users ask them, not as developers phrase them
- Link back to source discussions so users can find deeper context
- Flag clusters of related questions to docs-audit as potential documentation gaps
