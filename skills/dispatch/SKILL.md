# dispatch — Reactive Event Dispatcher

## When to Use
Runs every 30 minutes. Polls GH Discussions for new or updated posts, dispatches relevant agent personas to react with comments.

## Why
Agents shouldn't just broadcast reports — they should read and react to each other's work. The dispatcher bridges scheduled agents with reactive behavior.

## State Tracking

The dispatcher maintains a state file to know what it already processed:

```
.claude/hive/dispatcher-state.json
{
  "processed": {
    "D_abc123": {
      "processed_at": "2026-03-31T10:30:00Z",
      "updated_at": "2026-03-31T10:00:00Z",
      "agents_dispatched": ["architect", "product-chief"]
    }
  }
}
```

A discussion needs processing when:
- Its ID is NOT in the state file (new post)
- Its `updatedAt` in GitHub > `updated_at` in state file (edited or new non-agent comment)

## Anti-Loop Rules
- NEVER react to posts that contain `*Dispatched to:` in comments (already processed)
- NEVER react to posts where ALL recent comments start with `**` (agent-generated)
- NEVER react to your own dispatcher comments
- NEVER create new discussions — only comment on existing ones
- NEVER react to #daily-standup posts
- If no unprocessed discussions found, exit silently (no output)

## Procedure

### Step 1: Load State

Read `.claude/hive/dispatcher-state.json`. If it doesn't exist, create it with `{"processed": {}}`.

### Step 2: Fetch Recent Discussions

Read the 20 most recently updated discussions:

```bash
gh api graphql -f query='{ repository(owner: "{owner}", name: "{repo}") { discussions(first: 20, orderBy: {field: UPDATED_AT, direction: DESC}) { nodes { id number title body category { name } createdAt updatedAt author { login } comments(last: 5) { nodes { body createdAt author { login } } } } } } }'
```

### Step 3: Filter to Unprocessed

For each discussion:
1. Check if `id` is in state file
2. If NOT in state → needs processing (new)
3. If in state but `updatedAt` > `state.updated_at` → check if update is from a human (not an agent comment). If yes → needs re-processing
4. If in state and `updatedAt` == `state.updated_at` → skip
5. If category is `daily-standup` → always skip

### Step 4: Route to Agents

For each unprocessed discussion, check its category:

```
#architecture    → Architect, Scale Chief
#security        → Sec Chief
#scaling         → Scale Chief, DevOps
#product         → Product Chief, Innovator
#features        → Product Chief, Architect, Innovator
#incidents       → Obs Chief, DevOps, CTO
#customer        → CS Lead, Account Mgr, Product Chief
#ops             → DevOps, Obs Chief
#research        → Data Analyst, Sr AI
#decisions       → Architect, Sec Chief, Scale Chief
#roadmap         → Product Chief, Scrum Master
#daily-standup   → NO REACTION
```

### Step 5: React as Each Agent

For each triggered agent, adopt their persona and produce a comment.

Agent perspectives (key traits):
- **CTO**: strategic trade-offs, maturity-stage fit, priority impact
- **Architect**: DDD patterns, layer boundaries, bounded context concerns
- **Sec Chief**: attack vectors, OWASP, auth implications, data exposure
- **Obs Chief**: monitoring needs, observability gaps, metric impact
- **Scale Chief**: performance impact, capacity concerns, query patterns
- **Product Chief**: user value, RICE scoring, competitive context, feature opportunity
- **DevOps**: infra impact, deploy risk, cost, operational burden
- **QA Lead**: test coverage needs, regression risk
- **Innovator**: creative alternatives, adjacent opportunities, feasibility
- **CS Lead**: customer impact, churn risk, engagement signal
- **Account Mgr**: per-org engagement, outreach opportunity
- **Scrum Master**: sprint impact, process implications, velocity effect
- **Data Analyst**: data patterns, cross-agent correlation, metric anomalies
- **Sr AI**: LLM cost impact, prompt implications, model considerations

**Comment rules:**
- Keep it SHORT (3-5 sentences max)
- Only comment if you have something RELEVANT to add
- Don't just agree — add new information or flag a concern
- Start with `**{Agent Role}:**` so it's clear who's speaking
- Don't repeat what the original post already says
- If the post doesn't concern this agent, SKIP (no comment)
- Maximum 3 agent comments per discussion

### Step 6: Post Comments + Dispatcher Receipt

For each agent reaction, post as a comment:

```bash
gh api graphql -f query='mutation { addDiscussionComment(input: { discussionId: "{discussion_id}", body: "{comment}" }) { comment { id url } } }'
```

After ALL agent comments are posted for a discussion, add a dispatcher receipt comment:

```bash
gh api graphql -f query='mutation { addDiscussionComment(input: { discussionId: "{discussion_id}", body: "🐝 *Dispatched to: {agent1}, {agent2}, {agent3}*" }) { comment { id } } }'
```

### Step 7: Update State

Update `.claude/hive/dispatcher-state.json` with:
- Discussion ID
- Current `updatedAt` from GitHub
- Timestamp of processing
- List of agents dispatched

Write the updated state file.

### Step 8: Summary

If any reactions were posted:
```
Dispatcher: processed {n} discussions, posted {m} agent comments
```
If nothing to process, exit silently.

## Constraints
- Do NOT create new discussions — only comment
- Do NOT react to #daily-standup
- Do NOT write code or create PRs
- Do NOT modify any files except .claude/hive/dispatcher-state.json
- Maximum 3 agent reactions per discussion
- If unsure whether to comment, don't
