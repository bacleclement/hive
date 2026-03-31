# The Hive

Autonomous AI corporation that manages software projects end-to-end.

18 specialized agents. 85 skills. 21 scheduled tasks. Human-in-the-loop for key decisions.

**This is not a SaaS product.** It's personal infrastructure — a force multiplier for solo founders.

## How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                         HIVE REPO                                │
│                   (this repo — GitHub)                            │
│                                                                  │
│  The blueprint. Never runs. Never has secrets.                   │
│                                                                  │
│  agents/           18 agents (AGENT.md + SKILL.md + schedule)    │
│  skills/           85 skills across 18 categories                │
│  protocols/        Communication, escalation, adapters           │
│  skills/setup/     Bootstrap skill for new client projects       │
│                                                                  │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               │  setup skill reads hive,
                               │  creates .claude/hive/ in client
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                      CLIENT PROJECT                              │
│                  (your product repo)                              │
│                                                                  │
│  .claude/hive/                                                   │
│    config.json        maturity stage, repo info, discussion IDs  │
│    skills-map.json    maps client skills to hive agents           │
│    adapters/          HOW agents access this project's tools      │
│      observe-logs     → your hosting logs (Railway, Vercel, ...) │
│      observe-metrics  → your DB metrics (Supabase, RDS, ...)     │
│      observe-errors   → your error tracker (Sentry, ...)         │
│      security-deps    → your package manager audit               │
│      build            → your test/lint/build commands             │
│      infra-deploy     → your deploy tool                         │
│      customer-*       → your usage/feedback data                 │
│                                                                  │
│  codebase            the actual product code                     │
│                                                                  │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               │  scheduled tasks read SKILL.md
                               │  prompts + adapters, run against
                               │  the codebase
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                     YOUR MACHINE (runtime)                       │
│              ~/.claude/scheduled-tasks/                           │
│                                                                  │
│  {agent}-{schedule}/SKILL.md     created from hive SKILL.md      │
│  ...                             persists across sessions        │
│                                                                  │
│  Requires: Claude Code open from client project directory        │
│  Tasks fire when REPL is idle. You can code in between.          │
│                                                                  │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               │  agents READ from project,
                               │  WRITE to GitHub
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                        GITHUB                                    │
│                                                                  │
│  GH Discussions (agent reports)    GH Issues (task tracker)      │
│  ┌─────────────────────────┐      ┌──────────────────────┐      │
│  │ #daily-standup          │      │ Prioritized backlog  │      │
│  │ #security               │      │ P0 / P1 / P2        │      │
│  │ #incidents              │      │ XS / S / M / L / XL │      │
│  │ #architecture           │      │                      │      │
│  │ #ops                    │      │ Project board:       │      │
│  │ #research               │      │ Backlog → Ready →    │      │
│  │ #features               │      │ In Progress → Done   │      │
│  │ #product                │      │                      │      │
│  │ #customer               │      └──────────────────────┘      │
│  │ #scaling                │                                     │
│  │ #decisions              │                                     │
│  │ #roadmap                │                                     │
│  └─────────────────────────┘                                     │
└─────────────────────────────────────────────────────────────────┘
```

## Agent Lifecycle

```
DEFINITION (this repo, written once)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  agents/{name}/
    AGENT.md         WHO — persona, authority, knowledge domains
    SKILL-daily.md   WHAT — prompt for daily cycle
    SKILL-weekly.md  WHAT — prompt for weekly cycle
    schedule.json    WHEN — cron expressions
    context.md       STATE — rolling snapshot, updated each run


CONFIGURATION (client project, per-project)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  .claude/hive/
    config.json      project info, maturity stage
    adapters/        HOW to access this project's tools
    skills-map.json  which client skills agents can use


ACTIVATION (your machine, one-time setup)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  create_scheduled_task(
    taskId: "{agent}-{schedule}",
    cron:   from schedule.json,
    prompt: from SKILL-{schedule}.md
  )

  Persists to ~/.claude/scheduled-tasks/


RUNTIME (daily, automatic)
━━━━━━━━━━━━━━━━━━━━━━━━━━

  Cron fires → load SKILL prompt → read project → think → post report
```

## Daily Schedule

### Weekday Morning Flow

```
08:00  🔒 Sec Chief      → #security       Dependency scan, secret detection
08:15  👁 Obs Chief       → #daily-standup  Production health, overnight review
08:30  📋 Scrum Master    → #daily-standup  Standup compilation, blockers
09:00  🎯 CTO            → #daily-standup  Dispatch report, priority stack
09:15  📦 Product Chief   → #product        Product pulse, user signal
09:30  🔭 Scout           → #research       Competitive scan, market trends
14:00  🧪 QA Lead         → #daily-standup  Test suite, coverage, regressions
17:00  📋 Scrum Master    → #daily-standup  EOD wrap, tomorrow's priorities
```

### Weekly Schedule

```
MONDAY
  09:30  🎯 CTO           → #roadmap        Sprint planning, priority ranking
  10:00  💡 Innovator      → #features       Weekly ideation
  10:30  📊 Data Analyst   → #research       KPI trends, cross-agent analysis

WEDNESDAY
  10:00  🏗 Architect      → #architecture   Design review, BC audit
  10:00  ⚙️ DevOps         → #ops            Infra audit, cost review
  10:30  👥 CS Lead        → #customer       Org health scores, churn risk
  11:00  💼 Account Mgr    → #customer       Engagement review

THURSDAY
  10:00  📈 Scale Chief    → #scaling        Slow queries, capacity planning
  10:30  🤖 Sr AI          → #research       Token costs, prompt quality

FRIDAY
  16:00  📋 Scrum Master   → #daily-standup  Sprint review + retro
  16:00  📝 DevRel         → #daily-standup  Docs audit, changelog

MONTHLY (1st Tuesday)
  09:00  🔒 Sec Chief      → #security       Full security audit
```

## Information Flow

```
REAL WORLD                    AGENTS                         YOU
━━━━━━━━━━                    ━━━━━━                         ━━━

User interview ──► post to #customer ──► Product Chief ──┐
                                         CS Lead ────────┤
                                                         │
Competitor launch ─────────────────────► Scout ──────────┤
                                                         ├──► GH Discussions
Prod error ────────────────────────────► Obs Chief ─────┤     (you read over
                                                         │      coffee)
CVE published ─────────────────────────► Sec Chief ─────┤         │
                                                         │         ▼
Code pushed ───────────────────────────► QA Lead ───────┘   Pick what to
                                                             work on
                                              │
                                              ▼
                                         Your dev workflow
                                         (orchestrate → refine
                                          → build-plan → implement)
                                              │
                                              ▼
                                         GH Issue → Done
                                         Scrum Master tracks it
```

## Adapter Architecture

Agents are project-agnostic (ports). Client projects provide adapters (implementations).

```
HIVE defines PORTS                    CLIENT provides ADAPTERS
━━━━━━━━━━━━━━━━━                    ━━━━━━━━━━━━━━━━━━━━━━━

observe.logs                          → railway logs
observe.errors                        → sentry API
observe.metrics                       → supabase MCP / pg_stat
infra.deploy                          → railway status
infra.db                              → supabase CLI
security.deps                         → pnpm audit
security.secrets                      → gitleaks
customer.activity                     → SQL on usage_events
customer.feedback                     → SQL on usage_events
notify.telegram                       → telegram bot API
build.*                               → pnpm nx run {project}:*
```

See `protocols/adapters.md` for the full port registry.

## Maturity-Aware Decisions

Agents read `config.json.maturity.stage` and adjust behavior:

| Stage | Label | Agent behavior |
|-------|-------|----------------|
| 1 | POC | Ship fast. Only block on security basics. Monolith mandatory. |
| 2 | Early Product | Balance speed with stability. Start measuring. No distributed systems. |
| 3 | Growth | Invest in foundations. Auto-scaling. Cache strategy. Pay tech debt. |
| 4 | Scale | Every decision has a business case. Multi-region. Chaos engineering. |

## Repo Structure

```
hive/
├── README.md              This file
├── design.md              Full architecture blueprint
├── metadata.json          Epic metadata
│
├── agents/ (18 agents)
│   └── {name}/
│       ├── AGENT.md       Persona, authority, knowledge domains
│       ├── SKILL-*.md     Executable prompts (1 per scheduled cycle)
│       ├── schedule.json  Cron expressions
│       └── context.md     Rolling state template
│
├── skills/ (85 skills across 18 categories)
│   ├── strategy/          dispatch, prioritize, decision, roadmap, cost-review
│   ├── architecture/      adr, design-review, dependency-map, bounded-context-audit
│   ├── security/          vuln-scan, auth-audit, secret-scan, compliance, pentest, incident
│   ├── observability/     health-check, anomaly-detect, incident-triage, metrics, postmortem
│   ├── infra/             deploy, rollback, backup, audit, scale, ci-monitor, smoke-test
│   ├── performance/       perf-audit, n-plus-one, capacity, cache, connection-pool
│   ├── code/              code-review, refactor
│   ├── ai/                prompt-audit, llm-cost, model-eval, rag-quality, prompt-optimize
│   ├── qa/                coverage, acceptance, regression, test-strategy
│   ├── product/           competitive-scan, user-insight, feature-brief, market-size
│   ├── research/          market-scan, competitor, trend, partnership, source-evaluate
│   ├── innovation/        ideate, feasibility, impact-estimate, prototype-brief
│   ├── ceremonies/        standup, sprint-plan, sprint-review, retro, blocker, velocity
│   ├── customer/          health-score, churn, cohort, expansion, nps
│   ├── account/           onboard, engagement, outreach, churn-response
│   ├── support/           ticket-triage, auto-resolve, escalate, kb-update
│   ├── docs/              docs-audit, changelog, faq, onboard-test
│   ├── data/              cross-agent, decision-audit, sentiment, kpi, conversation, insights
│   └── setup/             Bootstrap skill for new client projects
│
└── protocols/
    ├── communication.md   How agents talk to each other
    ├── escalation.md      When to involve the human
    ├── knowledge-dispatch.md  Who owns what knowledge
    ├── project-maturity.md    Stage-based decision rules
    └── adapters.md        Port registry for all adapters
```

## Agents

| Role | Codename | Schedule | Writes to |
|------|----------|----------|-----------|
| CTO | `cto` | daily + weekly | #daily-standup, #roadmap, #decisions |
| Architect | `architect` | weekly | #architecture, #decisions |
| Sec Chief | `sec-chief` | daily + monthly | #security, #incidents |
| Obs Chief | `obs-chief` | daily | #daily-standup, #incidents, #ops |
| DevOps | `devops` | weekly | #ops, #incidents |
| Product Chief | `product-chief` | daily | #product |
| Scale Chief | `scale-chief` | weekly | #scaling |
| Sr Backend | `sr-backend` | on-demand | #daily-standup |
| Sr AI | `sr-ai` | weekly | #research |
| QA Lead | `qa-lead` | daily | #daily-standup |
| Scout | `scout` | daily | #research |
| Innovator | `innovator` | weekly | #features |
| Scrum Master | `scrum-master` | daily (x2) + weekly | #daily-standup, #decisions |
| CS Lead | `cs-lead` | weekly | #customer |
| Account Mgr | `account-mgr` | weekly | #customer |
| DevRel | `devrel` | weekly | #daily-standup |
| Data Analyst | `data-analyst` | weekly | #research |
| Support | `support` | — | — |

## Getting Started

1. **Bootstrap a client project:**
   ```bash
   cd ~/Code/your-project
   claude --prompt "$(cat ~/Code/hive/skills/setup/SKILL.md)"
   ```

2. **Set up GH Discussions** on the client repo (12 categories)

3. **Create scheduled tasks** from agent SKILL.md files:
   ```bash
   cd ~/Code/your-project
   claude
   # then: "Read all agents from ~/Code/hive/agents/ and create scheduled tasks"
   ```

4. **Open Claude Code daily** from the client project directory. Agents fire automatically.

## Design Principles

1. **Agents don't code** — they read, analyze, and report. The human decides.
2. **Hive is project-agnostic** — zero client-specific content. Adapters live in the client.
3. **Reports are discussions, not files** — GH Discussions is the comms bus. No report files to manage.
4. **Maturity-aware** — agents adjust behavior based on project stage. A POC doesn't need chaos engineering.
5. **Human-in-the-loop** — agents recommend, you decide. Authority matrix defines what's autonomous vs needs approval.
