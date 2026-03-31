# setup — Bootstrap Hive in a Client Project

## When to Use
First time connecting the hive to a new project. Creates `.claude/hive/` with config, adapters, and scheduled task registration.

## Procedure

### Step 1: Gather Project Info

Ask the user:
1. Project name?
2. GitHub repo (owner/name)?
3. What maturity stage? (1=POC, 2=Early Product, 3=Growth, 4=Scale)
4. Current user/org count?
5. Tech stack? (hosting, DB, channels)

### Step 2: Create `.claude/hive/config.json`

```json
{
  "project": "{name}",
  "repo": {
    "owner": "{owner}",
    "name": "{repo}",
    "base_branch": "main"
  },
  "maturity": {
    "stage": {n},
    "label": "{label}",
    "users": "~{n}",
    "orgs": {n},
    "last_assessed": "{date}"
  },
  "discussions": {
    "repo_id": "{get via gh api}",
    "category_ids": "{get via gh api}"
  },
  "human": {
    "name": "{name}",
    "notify": {
      "telegram": "env:HIVE_TG_CHAT_ID"
    }
  }
}
```

### Step 3: Create Adapters

Based on the tech stack, create adapter files in `.claude/hive/adapters/`:

**For Railway hosting:**
- `observe-logs.md` — `railway logs --json`
- `infra-deploy.md` — `railway status`

**For Supabase DB:**
- `observe-metrics.md` — SQL queries via psql or Supabase CLI
- `infra-db.md` — `supabase inspect`
- `security-auth.md` — `supabase inspect db policies`

**For NestJS API:**
- `security-deps.md` — `pnpm audit --json`
- `security-secrets.md` — gitleaks or manual scan
- `build.md` — `pnpm nx run {project}:test/lint/build`

**For Telegram/WhatsApp channels:**
- `notify-telegram.md` — bot token + chat ID

**Always create:**
- `customer-activity.md` — SQL on usage_events table (after #11 is built)
- `customer-feedback.md` — SQL on usage_events where type = 'feedback_given'

Each adapter file follows this format:

```markdown
# {adapter-name}

## Tool
{tool name}

## Command
{command to run}

## Environment Variables
- {VAR_NAME} — {description}

## Output Format
{what the agent should expect}

## Notes
{setup instructions, prerequisites}

## Status
{active | not-configured | blocked-by-#{issue}}
```

### Step 4: Verify Environment

Check each adapter's env vars are set:
```bash
echo "Checking env vars..."
for var in RAILWAY_TOKEN SENTRY_DSN SUPABASE_ACCESS_TOKEN; do
  if [ -z "${!var}" ]; then
    echo "❌ $var not set"
  else
    echo "✅ $var set"
  fi
done
```

### Step 5: Get GH Discussion Category IDs

```bash
gh api graphql -f query='{ repository(owner: "{owner}", name: "{repo}") {
  id
  discussionCategories(first: 20) {
    nodes { id name }
  }
} }'
```

Store in config.json.

### Step 6: Register Scheduled Tasks

Read all agent schedules from `~/Code/hive/agents/*/schedule.json`.
For each schedule, read the corresponding `SKILL-*.md`.
Create scheduled tasks via `create_scheduled_task`.

### Step 7: Summary

Print what was created:
```
.claude/hive/
  config.json           ✅ Project: {name}, Stage: {stage}
  adapters/
    observe-logs.md     ✅ Railway
    observe-errors.md   ⚠️ Needs SENTRY_DSN
    observe-metrics.md  ✅ Supabase
    ...

Scheduled tasks: {n} created
  cto-daily             ✅ weekdays 9:00
  scrum-master-standup  ✅ weekdays 8:30
  ...

Missing env vars:
  SENTRY_DSN — set up Sentry first (#35)
```

## Constraints
- Do NOT create secrets — only reference env vars
- Do NOT commit env vars to git
- Add `.claude/hive/adapters/` to `.gitignore` if it contains any non-env-var secrets
- Ask before creating scheduled tasks
