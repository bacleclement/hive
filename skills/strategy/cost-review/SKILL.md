# cost-review — Review All Costs

## When to Use
CTO runs this weekly (during sprint planning) and monthly (roadmap review).

## Inputs
- `adapter:observe.metrics` — LLM token usage, infra costs
- Previous cost-review output (for trend comparison)

## Procedure

1. Collect cost data:
   - Claude Code scheduled tasks (API usage)
   - OpenAI API (Gotchi's enrichment + conversation)
   - Railway hosting
   - Supabase (DB + Storage + Auth)
   - External services (Tavily, Deepgram, etc.)

2. Compare to last period

3. Flag overruns (> $10/day on any service without prior approval)

4. Post to `#daily-standup` (weekly) or `#roadmap` (monthly)

## Output Format

```markdown
---
agent: cto
type: report
severity: info
tags: [costs]
requires: info
---

## Cost Review — {period}

| Service | Daily Avg | Previous | Delta | Status |
|---------|-----------|----------|-------|--------|
| OpenAI (Gotchi) | ${n} | ${n} | {%} | {emoji} |
| Claude Code (Hive) | ${n} | ${n} | {%} | {emoji} |
| Railway | ${n} | ${n} | {%} | {emoji} |
| Supabase | ${n} | ${n} | {%} | {emoji} |
| Tavily | ${n} | ${n} | {%} | {emoji} |
| Deepgram | ${n} | ${n} | {%} | {emoji} |
| **Total** | **${n}** | **${n}** | **{%}** | |

### Flags
- {any overrun or concerning trend}

### Recommendations
- {cost optimization suggestions}
```
