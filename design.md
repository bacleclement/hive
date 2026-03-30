# The Hive — Design

## Problem

Solo founders running complex software projects hit three ceilings:

1. **Attention ceiling** — can't watch prod health, scan competitors, audit security, AND build features simultaneously. Things fall through cracks.
2. **Continuity ceiling** — AI sessions are stateless between conversations. Operational rhythm dies unless manually re-loaded.
3. **Speed ceiling** — sequential work. While building feature X, nobody researches feature Y, monitors prod, analyzes patterns, or checks deps for CVEs.

The Hive solves this by creating a persistent, multi-agent "virtual company" that runs continuously, maintains institutional knowledge, parallelizes work across specialized roles, keeps a human-in-the-loop for decisions that matter, and learns from its own data over time.

**This is not a SaaS product.** It's personal infrastructure — a force multiplier for solo founders. Gotchi is the first client, but the system is project-agnostic.

## Approach

### Chosen: Claude Code Native + GitHub Discussions

Build entirely on **Claude Code primitives** — scheduled tasks, skills, subagents, worktrees — with **GitHub Discussions** as the communication bus. No external orchestration framework.

### Rejected Alternatives

| Alternative | Why rejected |
|---|---|
| CrewAI / LangGraph | External framework dependency. Can't use kitt skills natively. Agents are ephemeral (run once, die) — no persistent rhythm |
| n8n / Temporal | Good for automation, bad for reasoning. Agents need to think, not just execute steps. Another infra to host |
| Custom Python multi-agent | Reinventing what Claude Code already does. Scheduled tasks = cron. Skills = agent definitions. Subagents = parallel execution |
| Single mega-agent | One agent with 50 responsibilities = context overload, hallucination, no parallelism |

### Two Repos, One Brain

- **hive** (this repo) — project-agnostic infrastructure: agent definitions, ceremonies, protocols, notification bridges, data pipeline
- **project repos** (e.g. gotchi) — the actual product code. GH Discussions live here because they're about the product

The hive connects to project repos as "clients" via an adapter layer — same agents, same ceremonies, different tools underneath.

## Architecture

### 5 Subsystems

```
          ┌─────────────┐
          │  (E) CLIENT  │──── gotchi repo (kitt skills, code, GH Discussions)
          │  CONNECTOR   │──── future-project repo
          └──────┬──────┘
                 │
    ┌────────────┼────────────┐
    ▼            ▼            ▼
┌────────┐ ┌─────────┐ ┌──────────┐
│(A) HIVE│ │(B) COMMS │ │(C) HUMAN │
│  CORE  │ │   BUS    │ │   LOOP   │
│        │ │          │ │          │
│agents  │ │GH Discuss│ │email     │
│ceremony│ │protocols │ │telegram  │
│engine  │ │threading │ │whatsapp  │
│dispatch│ │tagging   │ │escalation│
└───┬────┘ └────┬─────┘ └────┬─────┘
    │           │             │
    └───────────┼─────────────┘
                ▼
         ┌────────────┐
         │ (D) DATA   │
         │   BRAIN    │
         │            │
         │ ingest all │
         │ analyze    │
         │ patterns   │
         │ insights   │
         └────────────┘
```

### Three-Layer Architecture (Ports & Adapters)

```
Layer 1: AGENTS + CEREMONIES + PROTOCOLS + HIVE SKILLS
         (pure logic — who does what, when, how they talk)

Layer 2: CAPABILITY INTERFACES
         observe.logs | infra.deploy | security.audit | notify.email
         (abstract — no tool mentioned)

Layer 3: CLIENT ADAPTERS
         clients/gotchi/adapters.json → Railway, Supabase, Sentry
         clients/gotchi/skills-map.json → kitt skills
         (concrete — resolved per project)
```

Agents never call project tools directly. They call abstract capabilities. The adapter resolves to the concrete tool.

---

## (A) Hive Core — Agents & Ceremonies

### Agent Registry — 18 Roles

| # | Role | Codename | Mission | Runs |
|---|------|----------|---------|------|
| 1 | CTO | `cto` | Strategic decisions, conflict resolution, roadmap, dispatch | Every 2h |
| 2 | Architect | `architect` | DDD alignment, bounded context integrity, ADR governance | On demand |
| 3 | Sec Chief | `sec-chief` | Deps, auth, data exposure, compliance, secret scanning | Daily 6am + weekly |
| 4 | Obs Chief | `obs-chief` | Prod health — errors, usage, anomalies, incident triage | Hourly |
| 5 | DevOps / Infra | `devops` | Deploys, scaling, backups, CI/CD, uptime, infra health | Every 4h + on deploy |
| 6 | Product Chief | `product-chief` | What to build next — user needs, market gaps, prioritization | Daily 9am |
| 7 | Scale Chief | `scale-chief` | Query perf, N+1 detection, caching, capacity planning | Every 4h |
| 8 | Sr Backend | `sr-backend` | Implements features with TDD. The builder. | On dispatch |
| 9 | Sr AI Engineer | `sr-ai` | AI pipeline quality, prompts, costs, accuracy, model selection | Every 4h + on dispatch |
| 10 | QA Lead | `qa-lead` | Quality gate — coverage, acceptance, regression | On push + daily |
| 11 | Scout | `scout` | Competitive intel, market signals, partnership opportunities | Every 12h |
| 12 | Innovator | `innovator` | Dream up killer features — wild but grounded | Weekly Mon |
| 13 | Scrum Master | `scrum-master` | Run ceremonies, unblock agents, enforce process | Daily ceremonies |
| 14 | CS Lead | `cs-lead` | Customer health scoring, churn prevention, expansion detection | Weekly Wed |
| 15 | Account Mgr | `account-mgr` | Per-org care — onboarding, engagement, retention | On signup + weekly |
| 16 | Support | `support` | First responder — triage, auto-resolve, escalate | Continuous |
| 17 | DevRel | `devrel` | Docs freshness, changelog, FAQ extraction, onboarding clarity | Weekly Fri |
| 18 | Data Analyst | `data-analyst` | Cross-agent pattern analysis, KPIs, insights nobody asked for | Every 6h |

### Agent Definition Format

Each agent lives in `agents/{codename}/`:

```
agents/{codename}/
├── AGENT.md          # Persona, mission, responsibilities, authority matrix
├── schedule.json     # Cron expressions
├── tools.md          # Layer 1 skills + Layer 2 adapters
└── context.md        # Persistent memory (updated by agent)
```

### Full Agent Skills Matrix

#### CTO

| Layer | Skill | Purpose |
|-------|-------|---------|
| L1 — Hive | `prioritize`, `dispatch`, `decision`, `roadmap`, `retrospective`, `cost-review` | Strategy + coordination |
| L2 — Client | `brainstorm`, `orchestrate`, `build-plan`, `refine` | Product development workflow |
| L3 — Tools | `gh discussion` (all), `gh issue`, `git log/diff`, `web search` | Full visibility + action |
| L4 — GH Disc | Read: ALL / Write: `#decisions`, `#daily-standup`, `#roadmap` | |

#### Architect

| Layer | Skill | Purpose |
|-------|-------|---------|
| L1 — Hive | `adr`, `design-review`, `dependency-map`, `bounded-context-audit` | Architecture governance |
| L2 — Client | `align`, `refine` (review), `build-plan` (review) | Validate features |
| L3 — Tools | `codebase search`, `nx graph`, `docs/adr/*`, `gh discussion` | Code + ADR access |
| L4 — GH Disc | Read: `#architecture`, `#decisions`, `#features` / Write: `#architecture`, `#decisions` | |

#### Sec Chief

| Layer | Skill | Purpose |
|-------|-------|---------|
| L1 — Hive | `vuln-scan`, `auth-audit`, `secret-scan`, `compliance-check`, `pentest-light`, `incident-security` | Full security coverage |
| L2 — Client | `debug` (vuln investigation) | Trace vulnerabilities |
| L3 — Tools | `pnpm audit`/`snyk`, `adapter:security.auth`, `adapter:security.secrets`, `codebase search`, `web search` | Scanning + investigation |
| L4 — GH Disc | Read: `#security`, `#architecture`, `#incidents` / Write: `#security`, `#incidents` | |

#### Obs Chief

| Layer | Skill | Purpose |
|-------|-------|---------|
| L1 — Hive | `health-check`, `anomaly-detect`, `incident-triage`, `runbook-execute`, `metrics-digest`, `postmortem` | Prod monitoring |
| L2 — Client | `debug`, `prod-check` (via adapter) | Investigation + gotchi health |
| L3 — Tools | `adapter:observe.logs`, `adapter:observe.errors`, `adapter:observe.metrics` | Railway, Sentry, psql |
| L4 — GH Disc | Read: `#incidents`, `#daily-standup`, `#ops` / Write: `#incidents`, `#daily-standup`, `#ops` | |

#### DevOps / Infra

| Layer | Skill | Purpose |
|-------|-------|---------|
| L1 — Hive | `deploy`, `rollback`, `backup-verify`, `infra-audit`, `scale-action`, `ci-monitor`, `smoke-test` | Infra reliability |
| L2 — Client | `prod-check` (via adapter) | Project-specific health |
| L3 — Tools | `adapter:infra.deploy`, `adapter:infra.db`, `adapter:infra.dns`, `adapter:observe.logs` | Railway, Supabase, DNS |
| L4 — GH Disc | Read: `#ops`, `#incidents`, `#scaling` / Write: `#ops`, `#incidents` | |

#### Product Chief

| Layer | Skill | Purpose |
|-------|-------|---------|
| L1 — Hive | `competitive-scan`, `user-insight`, `prioritize` (product), `feature-brief`, `roadmap` (product), `market-size` | Product strategy |
| L2 — Client | `brainstorm`, `refine` (epic), `orchestrate` | Feature development |
| L3 — Tools | `web search`, `adapter:observe.metrics`, `gh discussion` | Market data + user metrics |
| L4 — GH Disc | Read: `#features`, `#research`, `#daily-standup`, `#customer` / Write: `#features`, `#product`, `#decisions` | |

#### Scale Chief

| Layer | Skill | Purpose |
|-------|-------|---------|
| L1 — Hive | `perf-audit`, `n-plus-one-detect`, `capacity-plan`, `cache-strategy`, `connection-pool-audit` | Performance |
| L2 — Client | `debug` (perf mode) | Bottleneck investigation |
| L3 — Tools | `adapter:observe.metrics` (pg_stat, EXPLAIN ANALYZE), `codebase search` | Query + code analysis |
| L4 — GH Disc | Read: `#scaling`, `#architecture`, `#ops` / Write: `#scaling` | |

#### Sr Backend

| Layer | Skill | Purpose |
|-------|-------|---------|
| L1 — Hive | `code-review`, `refactor` | Code quality |
| L2 — Client | `implement`, `tdd`, `debug`, `verify`, `build-plan` (impl view) | Full dev cycle |
| L3 — Tools | `git`, `worktrees`, `pnpm nx run`, `codebase (full access)`, `gh discussion` | Build + validate |
| L4 — GH Disc | Read: `#architecture`, `#daily-standup`, `#features` / Write: `#daily-standup` | |

#### Sr AI Engineer

| Layer | Skill | Purpose |
|-------|-------|---------|
| L1 — Hive | `prompt-audit`, `llm-cost-track`, `model-eval`, `rag-quality`, `prompt-optimize` | AI pipeline quality |
| L2 — Client | `implement`, `tdd`, `debug` | Build AI features |
| L3 — Tools | `adapter:observe.metrics` (token costs), `codebase search`, `web search` | Cost + quality data |
| L4 — GH Disc | Read: `#architecture`, `#research`, `#daily-standup` / Write: `#research`, `#daily-standup` | |

#### QA Lead

| Layer | Skill | Purpose |
|-------|-------|---------|
| L1 — Hive | `coverage-audit`, `acceptance-check`, `regression-scan`, `test-strategy` | Quality gate |
| L2 — Client | `verify`, `tdd` (review) | Evidence-based checking |
| L3 — Tools | `pnpm nx run test`, `vitest --coverage`, `codebase search` | Test execution |
| L4 — GH Disc | Read: `#daily-standup`, `#features` / Write: `#daily-standup` | |

#### Scout

| Layer | Skill | Purpose |
|-------|-------|---------|
| L1 — Hive | `market-scan`, `competitor-track`, `trend-detect`, `partnership-scout`, `source-evaluate` | External intelligence |
| L2 — Client | — (research only) | |
| L3 — Tools | `web search`, `web fetch`, `gh discussion create` | Internet scanning |
| L4 — GH Disc | Read: `#research`, `#features`, `#product` / Write: `#research` | |

#### Innovator

| Layer | Skill | Purpose |
|-------|-------|---------|
| L1 — Hive | `ideate`, `feasibility`, `impact-estimate`, `prototype-brief` | Feature invention |
| L2 — Client | `brainstorm` | Structure ideas |
| L3 — Tools | `web search`, scout's reports, `adapter:observe.metrics`, `gh discussion` | Data-grounded creativity |
| L4 — GH Disc | Read: `#research`, `#features`, `#product`, `#customer` / Write: `#features` | |

#### Scrum Master

| Layer | Skill | Purpose |
|-------|-------|---------|
| L1 — Hive | `standup-run`, `sprint-plan`, `sprint-review`, `retrospective`, `blocker-detect`, `velocity-track` | Process enforcement |
| L2 — Client | `orchestrate` | Route unblocked work |
| L3 — Tools | `gh discussion` (all), `agents/*/context.md`, `adapter:notify.*` | Full read + notify |
| L4 — GH Disc | Read: ALL / Write: `#daily-standup`, `#decisions` | |

#### CS Lead

| Layer | Skill | Purpose |
|-------|-------|---------|
| L1 — Hive | `health-score`, `churn-detect`, `cohort-analysis`, `expansion-detect`, `nps-track` | Customer strategy |
| L2 — Client | — | |
| L3 — Tools | `adapter:observe.metrics`, `web search`, `gh discussion` | Engagement data |
| L4 — GH Disc | Read: `#customer`, `#product`, `#daily-standup` / Write: `#customer` | |

#### Account Manager

| Layer | Skill | Purpose |
|-------|-------|---------|
| L1 — Hive | `onboard-check`, `engagement-report`, `outreach-draft`, `churn-response` | Per-org care |
| L2 — Client | `org-onboard` (via adapter) | Project onboarding flow |
| L3 — Tools | `adapter:observe.metrics`, `adapter:notify.email`, `adapter:notify.telegram` | Data + outreach |
| L4 — GH Disc | Read: `#customer`, `#daily-standup` / Write: `#customer` | |

#### Support

| Layer | Skill | Purpose |
|-------|-------|---------|
| L1 — Hive | `ticket-triage`, `auto-resolve`, `escalate`, `kb-update` | First response |
| L2 — Client | `debug` (triage mode) | Bug investigation |
| L3 — Tools | `adapter:observe.errors`, `adapter:observe.metrics`, `adapter:notify.*`, knowledge base, `gh issue create` | Triage + respond |
| L4 — GH Disc | Read: `#incidents`, `#customer`, `#daily-standup` / Write: `#customer`, `#incidents` | |

#### DevRel

| Layer | Skill | Purpose |
|-------|-------|---------|
| L1 — Hive | `docs-audit`, `changelog-gen`, `faq-extract`, `onboard-test` | Docs quality |
| L2 — Client | `onboard` (review mode) | Test onboarding UX |
| L3 — Tools | `codebase (read/write)`, `gh discussion (scan)`, `git log` | Docs + history |
| L4 — GH Disc | Read: ALL / Write: `#daily-standup` | |

#### Data Analyst

| Layer | Skill | Purpose |
|-------|-------|---------|
| L1 — Hive | `cross-agent-analysis`, `trend-detect`, `decision-audit`, `sentiment-scan`, `kpi-dashboard`, `conversation-mine`, `weekly-insights` | Intelligence |
| L2 — Client | — | |
| L3 — Tools | `adapter:observe.*` (ALL), `agents/*/context.md`, `gh discussion (all)`, `mem0` (Phase 3), `psql` | Full data access |
| L4 — GH Disc | Read: ALL / Write: `#research`, `#daily-standup` | |

### Hive Skills Catalog — 76 Skills, 15 Categories

| Category | Skills | Agents |
|----------|--------|--------|
| Strategy | `prioritize`, `dispatch`, `decision`, `roadmap`, `cost-review` | CTO, Product Chief |
| Architecture | `adr`, `design-review`, `dependency-map`, `bounded-context-audit` | Architect |
| Security | `vuln-scan`, `auth-audit`, `secret-scan`, `compliance-check`, `pentest-light`, `incident-security` | Sec Chief |
| Observability | `health-check`, `anomaly-detect`, `incident-triage`, `runbook-execute`, `metrics-digest`, `postmortem` | Obs Chief, DevOps |
| Infra | `deploy`, `rollback`, `backup-verify`, `infra-audit`, `scale-action`, `ci-monitor`, `smoke-test` | DevOps |
| Performance | `perf-audit`, `n-plus-one-detect`, `capacity-plan`, `cache-strategy`, `connection-pool-audit` | Scale Chief |
| Code | `code-review`, `refactor` | Sr Backend |
| AI | `prompt-audit`, `llm-cost-track`, `model-eval`, `rag-quality`, `prompt-optimize` | Sr AI |
| QA | `coverage-audit`, `acceptance-check`, `regression-scan`, `test-strategy` | QA Lead |
| Product | `competitive-scan`, `user-insight`, `feature-brief`, `market-size` | Product Chief, Scout |
| Research | `market-scan`, `competitor-track`, `trend-detect`, `partnership-scout`, `source-evaluate` | Scout, Data Analyst |
| Innovation | `ideate`, `feasibility`, `impact-estimate`, `prototype-brief` | Innovator |
| Ceremonies | `standup-run`, `sprint-plan`, `sprint-review`, `retrospective`, `blocker-detect`, `velocity-track` | Scrum Master, CTO |
| Customer | `health-score`, `churn-detect`, `cohort-analysis`, `expansion-detect`, `nps-track` | CS Lead |
| Account | `onboard-check`, `engagement-report`, `outreach-draft`, `churn-response` | Account Mgr |
| Support | `ticket-triage`, `auto-resolve`, `escalate`, `kb-update` | Support |
| Docs | `docs-audit`, `changelog-gen`, `faq-extract`, `onboard-test` | DevRel |
| Data | `cross-agent-analysis`, `decision-audit`, `sentiment-scan`, `kpi-dashboard`, `conversation-mine`, `weekly-insights` | Data Analyst |

### Ceremony Engine

#### Daily

| Time | Ceremony | Owner | Participants |
|------|----------|-------|-------------|
| 06:00 | Security Morning Audit | sec-chief | Solo |
| 07:00 | Dawn Observability Report | obs-chief | Solo |
| 08:00 | Infra Health Check | devops | Solo |
| 08:30 | Daily Standup | scrum-master | ALL |
| 09:00 | Product Pulse | product-chief | + scout |
| 12:00 | Midday Sync | cto | + architect, sr-backend |
| 14:00 | QA Checkpoint | qa-lead | Solo |
| 16:00 | AI Cost Check | sr-ai | Solo |
| 17:00 | End-of-Day Wrap | scrum-master | ALL |
| 22:00 | Night Watch | obs-chief + devops | Duo |

#### Weekly

| Day | Time | Ceremony | Owner |
|-----|------|----------|-------|
| Mon | 09:00 | Sprint Planning | scrum-master + cto |
| Mon | 10:00 | Innovation Brainstorm | innovator |
| Tue | 09:00 | Security Deep Dive | sec-chief |
| Wed | 09:00 | Customer Health Review | cs-lead + account-mgr |
| Wed | 14:00 | AI Pipeline Review | sr-ai |
| Thu | 09:00 | Scaling Review | scale-chief |
| Fri | 14:00 | Sprint Review | scrum-master |
| Fri | 15:00 | Retrospective | scrum-master |
| Fri | 16:00 | Docs Refresh | devrel |
| Sat | 09:00 | Deep Internet Scan | scout |

#### Monthly

| When | Ceremony | Owner |
|------|----------|-------|
| 1st Mon | Roadmap Review | cto + product-chief |
| 1st Tue | Full Security Audit | sec-chief |
| 2nd Wed | Innovation Sprint (48h) | innovator + scout |
| Last Fri | Monthly Company Report | cto |

#### Event-Driven

| Trigger | Ceremony | Owner |
|---------|----------|-------|
| Error rate > 5% | Incident Response | obs-chief → cto |
| CVE critical | Security Incident | sec-chief → cto |
| New org signup | Onboarding Kickoff | account-mgr |
| Support ticket | Triage | support |
| Feature shipped | Post-Ship Review | qa-lead + obs-chief |
| User inactive 7d | Churn Alert | cs-lead |

### Dispatch Model

```
Work source (approved discussion / ceremony output / event / human request)
    → CTO reads queue, assigns agent, sets priority
        → Coding agents get worktrees (isolated)
        → Observation agents work in-place (read-only)
```

---

## (B) Communication Bus — GitHub Discussions

### Categories (on project repo)

| Category | Type | Who creates | Who comments |
|----------|------|-------------|-------------|
| `#decisions` | Announcement | CTO, Architect | ALL (vote/review) |
| `#daily-standup` | General | Scrum Master | ALL (progress) |
| `#incidents` | General | Obs Chief, Sec Chief, Support | DevOps, CTO, Sr Backend |
| `#features` | Ideas | Product Chief, Innovator | ALL (debate/vote) |
| `#architecture` | General | Architect | CTO, Sr Backend, Scale Chief |
| `#security` | General | Sec Chief | CTO, DevOps |
| `#scaling` | General | Scale Chief | DevOps, Architect |
| `#research` | General | Scout, Sr AI, Data Analyst | Product Chief, Innovator |
| `#customer` | General | CS Lead, Account Mgr, Support | Product Chief, CTO |
| `#ops` | General | DevOps, Obs Chief | CTO |
| `#product` | General | Product Chief | Scout, Innovator, CS Lead |
| `#roadmap` | Announcement | CTO | Product Chief, Scrum Master |

### Message Protocol

Every post uses structured frontmatter:

```markdown
---
agent: {codename}
type: report | proposal | alert | decision | question | progress
severity: info | warning | critical
tags: [tag1, tag2]
mentions: [@agent1, @agent2]
requires: info | review | approval | action
---

## [Title]

### Summary
### Evidence
### Recommendation
### Decision Needed
```

### Decision Authority Levels

| Level | Name | Process |
|-------|------|---------|
| 0 | AUTONOMOUS | Agent decides alone, posts as INFO |
| 1 | AGENT CONSENSUS | Agent proposes → 2+ agents agree → proceed. CTO can override |
| 2 | CTO DECIDES | Agent proposes → CTO reviews → approves/rejects. Human notified |
| 3 | HUMAN APPROVES | Agent proposes → CTO recommends → human decides. Email + Telegram + WhatsApp. Blocks until YES/NO |

### Rate Limiting

| Rule | Limit |
|------|-------|
| Max new discussions per agent/day | 5 |
| Max comments per agent/day | 20 |
| Min time between posts (same agent) | 30 min |
| Critical alerts | Unlimited (bypass) |
| Duplicate detection | Must search before posting — if similar < 24h, comment instead |
| Stale auto-label | No activity 7d → `stale` |

---

## (C) Human Loop — Notification & Approval

### Escalation Levels

| Level | Trigger | Channels | Behavior |
|-------|---------|----------|----------|
| 0 — INFO | `requires: info` | GH Discussion only | Read when convenient |
| 1 — NOTIFY | `requires: review` | GH + Telegram | FYI, no action needed |
| 2 — APPROVAL | `requires: approval` | GH + Telegram + Email | Blocks until YES/NO |
| 3 — URGENT | `severity: critical` | GH + Telegram + Email + WhatsApp | Escalates to phone if no reply 30min |

### Escalation Timeline

```
T+0     Approval request → GH + Telegram + Email
T+2h    No response → WhatsApp + Telegram reminder
T+3h    No response → SMS via Twilio
T+6h    Non-critical: auto-deferred to next standup
T+30min Critical: phone call. If still no response → pre-authorized safe action (rollback)
```

### Approval Queue

Managed by Scrum Master + `/loop` heartbeat:

```json
{
  "id": "approval-{date}-{seq}",
  "from": "{agent}",
  "type": "{deploy|email|architecture|spend|external}",
  "subject": "{description}",
  "discussion": "gh:discussions/{category}/{id}",
  "channels_sent": ["github", "telegram", "email"],
  "sent_at": "{ISO}",
  "status": "pending | approved | rejected | deferred",
  "response": null,
  "timeout": "2h",
  "escalate_to": "whatsapp",
  "blocking": ["{agent-codename}"]
}
```

### Response Handling

Human replies YES/NO on any channel → bridge router:
1. Match reply to pending approval
2. Update queue status
3. Post confirmation to GH Discussion
4. Label discussion (approved/rejected)
5. Notify blocking agents to proceed

Multi-channel sync: reply on email → Telegram updated too. Single truth = `approval-queue.json`.

### What Requires Human Approval (Level 3)

- Deploy to production
- Send email/message to customer
- Merge breaking architectural change
- Any spend > $10/day
- Delete data / destructive operations
- External communications
- New dependency adoption
- Discounting / billing changes

---

## (D) Data Brain — Intelligence Layer

### Mission

Every agent output, discussion, metric, log line = data. The Data Brain turns noise into intelligence.

### Data Sources

- GH Discussions (all posts, comments, labels, reactions)
- Agent `context.md` + `last-report.md` files
- Production logs (via observe.logs adapter)
- Error tracking (via observe.errors adapter)
- DB metrics (via observe.metrics adapter)
- LLM cost data
- User conversations (bot interactions)
- Git history (commits, velocity)
- Approval queue history

### Memory Strategy

| Phase | Layer | Purpose |
|-------|-------|---------|
| Phase 1-2 | `context.md` per agent | Current state, simple |
| Phase 3 | mem0 (graph memory) | Semantic search, relational memory, cross-agent temporal queries |

### Outputs

| Output | Cadence | Consumers |
|--------|---------|-----------|
| KPI Dashboard | Daily | CTO, Product Chief, Scrum Master |
| Weekly Insights | Monday AM | ALL |
| Anomaly Alerts | Real-time | CTO, Obs Chief |
| Decision Audit | Monthly | CTO, Architect |
| Agent Performance | Weekly | Scrum Master, CTO |
| Customer Patterns | Weekly | CS Lead, Product Chief |
| Cost Trends | Weekly | CTO, Sr AI |

### Pattern Detection

The Data Brain catches correlations no individual agent would:
- Temporal patterns (errors that spike on a schedule)
- Cross-domain signals (churn correlating with feature gaps)
- Agent dynamics (friction between roles, velocity impacts from process changes)
- Demand signals (multiple orgs requesting the same feature)

---

## (E) Client Connector — Multi-Project Support

### Client Configuration

```
hive/
└── clients/
    └── {project}/
        ├── config.json        # Repo, human, timezone, notify channels
        ├── adapters.json      # observe.* / infra.* / security.* / notify.* mappings
        ├── skills-map.json    # Client skill bindings per agent
        ├── discussions.json   # GH Discussion category setup
        ├── ceremonies.json    # Ceremony time overrides
        └── data-sources.json  # Data Brain ingestion config
```

### Gotchi Adapter Example

```json
{
  "observe": {
    "logs": { "tool": "railway", "cmd": "railway logs --last {period}" },
    "errors": { "tool": "sentry", "dsn": "env:SENTRY_DSN" },
    "metrics": { "tool": "psql", "conn": "env:DB_READ_REPLICA" }
  },
  "infra": {
    "deploy": { "tool": "railway", "cmd": "railway up" },
    "db": { "tool": "supabase", "project": "env:SUPABASE_PROJECT_ID" }
  },
  "security": {
    "deps": { "tool": "npm-audit", "cmd": "pnpm audit" },
    "auth": { "tool": "supabase-rls", "cmd": "supabase inspect db policies" },
    "secrets": { "tool": "gitleaks", "cmd": "gitleaks detect" }
  },
  "notify": {
    "email": { "tool": "resend", "key": "env:RESEND_API_KEY" },
    "telegram": { "tool": "telegram-bot", "token": "env:HIVE_TG_BOT_TOKEN" },
    "whatsapp": { "tool": "whatsapp", "token": "env:WA_BOT_TOKEN" }
  }
}
```

### Multi-Project

Same agents, same ceremonies, different adapters. A new project = new `clients/{name}/` folder with its own adapter config. Agents call `observe.logs` — the adapter resolves to Railway for Gotchi, Vercel for project-b.

### Startup Sequence

1. Read `clients/{project}/config.json`
2. Validate repo access
3. Load + verify adapters
4. Load + verify skills-map
5. Setup GH Discussion categories (if missing)
6. Register scheduled tasks (ceremonies)
7. Start `/loop` heartbeat
8. Initial health-check
9. Post: "Hive online. 18 agents active."

---

## Execution Model

### Three Parallel Modes

| Mode | Mechanism | Purpose |
|------|-----------|---------|
| Scheduled Tasks | Claude Code scheduled tasks (cloud) | Ceremonies — runs even if laptop is off |
| Event Watchers | GH webhooks + `/loop` | React to events — issues, deploys, tickets |
| Continuous Runner | `/loop 10m` in active session | Heartbeat — approvals, queue, reflexes |

### `/loop` Heartbeat (every 10 min)

1. Check notification channels for human replies
2. Check GH Discussions for new agent posts needing dispatch
3. Check support inbox for new tickets
4. Check deploy status if in progress
5. Process ceremony outputs needing action
6. Update approval queue dashboard
7. If idle — quick health check

---

## Out of Scope

- **Frontend/dashboard UI** — agents post to GH Discussions and notify via messaging. No custom web UI.
- **Billing/payments automation** — human-only.
- **Multi-tenant (serving other people's projects)** — this is personal infra, not a SaaS.
- **Agent-to-agent real-time chat** — async via GH Discussions is sufficient. No WebSocket bus.
- **Self-modifying agents** — agents don't rewrite their own AGENT.md. Human adjusts personas.

## Resolved Decisions

1. **Notification priority** — Telegram first. WhatsApp added later (Business API friction).
2. **Agent GitHub identity** — Individual accounts per agent (hive-cto, hive-architect, etc.). Full traceability in GH UI.
3. **Ceremony timezone** — Fixed Europe/Paris. No shift when traveling.
4. **GH Discussion moderation** — Scrum Master auto-closes stale discussions (7d no activity) during EOD ceremony.
5. **First 3 agents** — CTO, Obs Chief, Sr Backend. Minimum viable hive loop: strategy + monitoring + building.

## Remaining Open Questions

1. **Cost ceiling** — 18 agents running on Claude Code scheduled tasks. What's the estimated monthly cost? Need to benchmark before Phase 3.
2. **mem0 hosting** — self-host or managed? Self-host = complexity. Managed = cost + data leaving your infra. Deferred to Phase 3.
