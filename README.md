# The Hive

Autonomous AI corporation that manages software projects end-to-end.

18 specialized agents. 76 skills. 12 GH Discussion categories. Scheduled ceremonies. Human-in-the-loop for key decisions.

## Status: Design Complete

See `design.md` for the full architecture blueprint.

## Structure

```
hive/
├── design.md              # Full architecture blueprint
├── metadata.json          # Epic metadata
├── agents/                # 18 agent personas (AGENT.md per role)
├── skills/                # 76 hive-generic skills (15 categories)
├── ceremonies/            # Scheduled ceremony configs
├── bridges/               # Notification adapters (Telegram, Email, WhatsApp)
├── protocols/             # Communication rules, decision authority
├── data-brain/            # Analytics pipeline config
└── clients/               # Per-project adapter configs
    └── gotchi/            # First client
```

## Agents

| Role | Codename | Mission |
|------|----------|---------|
| CTO | `cto` | Strategy, dispatch, roadmap |
| Architect | `architect` | DDD, ADRs, design review |
| Sec Chief | `sec-chief` | Security audits, compliance |
| Obs Chief | `obs-chief` | Prod monitoring, incidents |
| DevOps | `devops` | Infra, deploys, backups |
| Product Chief | `product-chief` | Roadmap, market, features |
| Scale Chief | `scale-chief` | Performance, capacity |
| Sr Backend | `sr-backend` | TDD implementation |
| Sr AI Engineer | `sr-ai` | AI pipeline quality |
| QA Lead | `qa-lead` | Test coverage, acceptance |
| Scout | `scout` | Competitive intelligence |
| Innovator | `innovator` | Feature invention |
| Scrum Master | `scrum-master` | Ceremonies, process |
| CS Lead | `cs-lead` | Customer health, churn |
| Account Mgr | `account-mgr` | Per-org care |
| Support | `support` | Triage, auto-resolve |
| DevRel | `devrel` | Docs, changelog, FAQ |
| Data Analyst | `data-analyst` | Cross-agent intelligence |
