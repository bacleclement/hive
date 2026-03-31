# dispatch — Reactive Event Dispatcher

## When to Use
Runs every 30 minutes. Polls GH Discussions for new posts, dispatches relevant agent personas to react with comments.

## Why
Agents shouldn't just broadcast reports — they should read and react to each other's work. The dispatcher bridges scheduled agents with reactive behavior.

## Anti-Loop Rules
- NEVER react to posts made by agents (check for `*Agent:` signature at the bottom)
- NEVER react to your own dispatcher comments
- NEVER create new discussions — only comment on existing ones
- If no new discussions found, exit silently (no output)

## Procedure

### Step 1: Find New Discussions

Read discussions updated in the last 35 minutes:

```bash
gh api graphql -f query='{ repository(owner: "{owner}", name: "{repo}") { discussions(first: 10, orderBy: {field: UPDATED_AT, direction: DESC}) { nodes { id title body category { name } createdAt author { login } comments(last: 5) { nodes { body author { login } } } } } } }'
```

Filter to discussions where:
- `createdAt` is within the last 35 minutes, OR
- A new comment was added in the last 35 minutes
- Author is NOT an agent (doesn't end with `*Agent:` signature)

If nothing new → exit silently.

### Step 2: Route to Agents

For each new discussion, check its category and determine which agents should react:

```
#architecture    → Architect, Scale Chief
#security        → Sec Chief
#scaling         → Scale Chief, DevOps
#product         → Product Chief, Innovator
#features        → Product Chief, Architect, Innovator
#incidents       → Obs Chief, DevOps, CTO
#customer        → CS Lead, Account Mgr, Product Chief
#ops             → DevOps, Obs Chief
#research        → Scout, Data Analyst, Sr AI
#decisions       → Architect, Sec Chief, Scale Chief (review decisions)
#roadmap         → Product Chief, Scrum Master
#daily-standup   → NO REACTION (read-only compilation category)
```

### Step 3: React as Each Agent

For each triggered agent, adopt their persona and produce a comment.

Load the agent's persona from their AGENT.md (key traits only):
- CTO: strategic, trade-offs, maturity-aware
- Architect: DDD, layer boundaries, bounded contexts
- Sec Chief: attack vectors, OWASP, CVEs
- Obs Chief: data-driven, metrics, baselines
- Scale Chief: performance, capacity, N+1
- Product Chief: user value, RICE, competitive context
- DevOps: infra impact, deploy risk, cost
- QA Lead: test coverage, regression risk
- Innovator: creative alternatives, feasibility
- CS Lead: customer impact, churn risk
- Account Mgr: per-org engagement impact

**Comment rules:**
- Keep it SHORT (3-5 sentences max)
- Only comment if you have something RELEVANT to add
- Don't just agree — add new information or flag a concern
- Start with `**{Agent Role}:**` so it's clear who's speaking
- Don't repeat what the original post already says
- If the post doesn't concern you, SKIP (no comment)

### Step 4: Post Comments

For each agent reaction, post as a comment on the discussion:

```bash
gh api graphql -f query='mutation { addDiscussionComment(input: { discussionId: "{discussion_id}", body: "{comment}" }) { comment { id url } } }'
```

### Step 5: Summary

If any reactions were posted, output a one-line summary:
```
Dispatcher: reacted to {n} discussions, posted {m} comments
```

## Constraints
- Do NOT create new discussions
- Do NOT react to agent-authored posts (check for `*Agent:` footer)
- Do NOT react to #daily-standup posts (compilation-only category)
- Do NOT write code or modify files
- Maximum 3 agent reactions per discussion (avoid noise)
- If unsure whether to comment, don't
