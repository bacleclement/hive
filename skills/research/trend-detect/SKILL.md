# trend-detect — Detect Emerging Trends

## When to Use
Scout and Data Analyst use this skill. Scout scans external sources for market trends. Data Analyst scans internal data for usage trends. Can be run independently or together.

## Inputs
- **Scout**: Web sources (industry blogs, competitor updates, tech news, community forums)
- **Data Analyst**: Internal usage data (feature adoption, engagement patterns, growth metrics)
- Previous trend reports for continuity

## Procedure

1. **Scan for signals**:
   - Scout: search web for emerging trends in sourcing, CRM, AI-assisted workflows, and adjacent spaces
   - Data Analyst: query internal data for shifts in usage patterns, feature adoption curves, anomalies
2. For each signal detected, document:
   - **What's changing** — describe the trend in one sentence
   - **Evidence** — source link, data point, or metric
   - **Direction** — growing / declining / emerging / stabilizing
3. **Assess relevance** — does this trend affect gotchi's product, market, or users?
   - **Direct**: affects core product functionality or target users
   - **Adjacent**: affects a related space, could become relevant
   - **Peripheral**: interesting but no clear connection
4. **Classify urgency**:
   - **Act now**: trend is active and competitors are moving
   - **Watch**: trend is building, worth monitoring
   - **Note**: interesting signal, log for future reference
5. Post to `#research`:

```markdown
---
agent: {scout | data-analyst}
type: research
severity: info
tags: [trend, {external | internal}]
mentions: []
requires: read
---

## Trend Report — {date}

### Signal: {trend name}
- **What's changing**: {description}
- **Evidence**: {source or data point}
- **Direction**: {growing / declining / emerging / stabilizing}
- **Relevance**: {direct / adjacent / peripheral}
- **Urgency**: {act now / watch / note}
- **Implication for Gotchi**: {what this means for us}

### Signal: {trend name}
...
```

## Output Format
Single post to `#research` in the template above. Multiple signals can be in one report.

## Rules
- Every signal must have evidence — a source link or a data point. No speculation without basis
- Relevance and urgency are judgments — explain the reasoning, don't just label
- "Act now" signals must be flagged to Innovator and Product Chief within 24h
- Revisit previous "watch" signals monthly — upgrade or archive them
