---
name: hermes-profile-setup
description: "Build specialized Hermes agent profiles: secretary, executive assistant, or custom role profiles with SOUL.md, config, cron workflows, permissions, and templates."
version: 1.0.0
author: agent-created
tags: [hermes, profile, setup, secretary, assistant, config, cron]
---

# Hermes Profile Setup

Build specialized agent profiles on Hermes. Each profile gets its own identity (SOUL.md), config personality, cron workflows, memory schema, and output templates.

## Prerequisites

- Hermes Agent running
- Target profile exists (directory at `~/.hermes/profiles/<name>/`)
- Hermes CLI available (`hermes config set`)

## Step 1 — Survey Profile State

Check what exists before writing:

```bash
# Profile directory structure
ls ~/.hermes/profiles/<name>/

# Config
cat ~/.hermes/profiles/<name>/config.yaml

# Identity document
cat ~/.hermes/profiles/<name>/SOUL.md

# Existing cron jobs
cronjob action='list'

# Existing memory
read_file ~/.hermes/profiles/<name>/memories/USER.md
read_file ~/.hermes/profiles/<name>/memories/MEMORY.md
```

Check connected tools:

```bash
# Common CLIs
which gws himalaya imsg remindctl memo ntn obsidian airtable xurl gh
```

## Step 2 — Write SOUL.md

The SOUL.md is the agent's identity document. Structure:

```
# SOUL.md — [Profile Name] Agent

## Identity
Who you are, operating style, core traits.

## Mission
What success means.

## Operating Style
Conciseness rules, output format preference.

## Communication Rules
Tone, fluff policy, fact-checking.

## Output Templates
### Morning Brief
- Calendar today:
- Top priorities:
...

### Meeting Brief
- Time:
- Attendees:
...

### Follow-Up Draft
Subject: ...

Hi [Name],
...
```

## Step 3 — Set Profile Personality

The file tool **cannot** modify `config.yaml` directly (security restriction). Use `hermes config set`:

```bash
# Set timezone
hermes config set profile.timezone "Asia/Bangkok"

# Set display personality
hermes config set display.personality "secretary"

# Set compact display
hermes config set display.compact true

# Register a custom personality (use in SOUL.md context)
hermes config set agent.personalities.secretary "You are Hermes Secretary..."
```

DO NOT attempt `patch` or `write_file` on `config.yaml` — they are refused with "Agent cannot modify security-sensitive configuration."

## Step 4 — Configure Tool .env Vars

Each tool CLI expects specific env var names. The profile's `.env` at `~/.hermes/profiles/<name>/.env` is loaded by Hermes every session.

### Common tool env vars

```bash
# Notion
NOTION_API_KEY=***                        # Hermes skills / curl
NOTION_API_TOKEN=***        # same value — ntn reads this
NOTION_KEYRING=0

# GitHub (uses gh CLI — no env var needed, uses ~/.config/gh/)
```

### Rules

- **Set both aliases** when a tool uses a different env var name (e.g. `NOTION_API_KEY` vs `NOTION_API_TOKEN`)
- **Heredoc pitfall**: single-quoted delimiters prevent expansion: `cat >> .env << 'EOF'` writes literal `$VAR` not its value. Use unquoted `cat >> .env << EOF`
- **Terminal masks secrets** — verify tool setup from `execute_code` with Python `subprocess` and explicit env dict
- **PATH**: binary at `~/.hermes/profiles/<name>/home/bin/` is in the sandboxed PATH; symlink to `~/.local/bin/` for system-wide access

## Step 5 — Create Memory Schema

Save durable facts using the memory tool:

- User profile: name, timezone, hours, tone, VIPs
- Permission model: allowed w/o approval, approval required, forbidden
- Priority rules: P0-P3 definitions
- Connected tools inventory

## Step 6 — Set Up Cron Workflows

**Pitfall**: `repeat:false` disables recurrence; omit `repeat` entirely for forever schedule.

Pattern for each cron job:

```python
cronjob(
    action='create',
    name='[Workflow Name]',
    schedule='[cron or human-readable]',  # e.g. "0 2 * * 1-5", "every 15m"
    prompt='[self-contained prompt with {{time}} {{date}}]',
    enabled_toolsets=['session_search'],  # limit to reduce token cost
    workdir='/Users/<user>',  # for consistent project context
)
```

### Secretary cron jobs (reference set):

| Job | Schedule | Purpose |
|-----|----------|---------|
| Morning Brief | Weekdays 02:00 UTC (09:00 +7) | Calendar, priorities, urgent messages |
| Inbox Triage | Weekdays hourly 02-11 UTC | Classify P0-P3, extract asks |
| Urgent Monitor | Every 15m | Scan for P0/P1 items |
| Pre-Meeting Prep | Every 10m | Brief for meetings in 60min |
| Post-Meeting Follow-Up | Every 15m | Draft summaries, action items |
| Daily Follow-Up Scan | Weekdays 09:00 UTC (16:00 +7) | Find threads waiting on response |
| End-of-Day Wrap | Weekdays 10:30 UTC (17:30 +7) | Completed/open/risks |
| Weekly Review | Friday 09:00 UTC (16:00 +7) | Week summary, next priorities |
| Tech Daily Update | Weekdays 09:00 local | Gather 24h news, post to Slack |

### Cron prompt structure:
- Start with "You are [profile identity]..."
- Include `{{time}}` and `{{date}}` templates
- Handle missing integrations gracefully ("No email CLI — note limitation")
- Only use `enabled_toolsets` that match the task (ideally just `session_search` for secretary tasks)
- Set `workdir` for consistent context injection
- **`deliver` field vs. prompt instructions**: The `deliver` field handles WHERE the final output goes (e.g. `"slack"` posts to the home channel DM). Do NOT add redundant instructions like "then post to Slack channel" in the prompt — the prompt should focus on WHAT to produce (the content), and `deliver` handles routing. Redundant instructions create confusion when the channel isn't specified.
- **No placeholder text**: Never use placeholder tokens like `#[CHANNEL_NAME]`, `[YOUR_CHANNEL]`, or similar unresolved variables in cron prompts. These will never be filled in — cron jobs have no user interaction. If a channel is needed, use the `deliver` field or hardcode it in the prompt.

## Step 7 — Permission Model

Define three tiers:

| Category | Typical Actions |
|----------|----------------|
| Allowed w/o approval | Read metadata, read calendar, read docs, summarize, draft replies, suggest tasks, create private notes |
| Approval required | Send email/chat, create/accept/decline invites, edit shared docs, submit forms, book travel, purchase, delete anything |
| Forbidden | git push remote, reveal credentials, legal/financial/medical/HR decisions, change security, commit user to contracts |

## Step 8 — Priority Classification

Standard secretary priority schema:

| Level | Meaning | Examples |
|-------|---------|---------|
| P0 | Urgent, blocks outcome | Same-day deadline, blocked exec decision, major client issue, meeting soon w/o prep |
| P1 | Important, handle soon | Client follow-up, deadline this week, decision needed |
| P2 | Useful, not urgent | Admin cleanup, docs, non-urgent scheduling |
| P3 | Optional/backlog | Nice-to-have, low-value FYI |

## Step 9 — Command Interface

Define commands in SOUL.md for user-facing invocation:

- `/hermes brief today`
- `/hermes triage inbox`
- `/hermes draft reply to [person]`
- `/hermes schedule [meeting]`
- `/hermes prep meeting [name]`
- `/hermes follow up on [topic]`
- `/hermes summarize thread [link]`
- `/hermes create tasks from this`
- `/hermes eod`
- `/hermes weekly review`
- `/hermes settings`

## Pitfalls

- **Config file edits blocked**: NEVER try `patch` or `write_file` on `config.yaml`. Always use `hermes config set`.
- **HOME env var is sandboxed**: Inside a profile session, `~` resolves to `~/.hermes/profiles/<name>/home/`. Use `pwd.getpwuid(os.getuid()).pw_dir` in Python to find real user home.
- **Cron jobs cannot ask questions**: Prompts must be fully self-contained. Include `{{time}}` and `{{date}}` for temporal context.
- **`deliver` field + prompt redundancy**: Setting `deliver: "slack"` (or any platform) handles output routing automatically. The prompt should NOT also say "post to [platform]" — it creates conflicting instructions and leaves undefined variables (e.g. `#[CHANNEL_NAME]`) that the job cannot resolve.
- **No unresolved placeholders**: Never leave tokens like `#[CHANNEL_NAME]`, `[TEAM_NAME]`, `<INSERT_HERE>` in cron prompts. Cron jobs run headlessly — no user sees the prompt to fill in blanks. The job will fail or produce confused output.
- **`repeat` param gotcha**: Passing `repeat=false` (or `repeat: 1`) on a cron schedule creates a one-shot job, not recurring. For recurring schedules (cron expressions), **omit `repeat` entirely** — it defaults to `"forever"`. Only pass `repeat` for one-shot jobs.
- **Debugging cron jobs that don't complete**: When `cronjob(action='run')` returns success but `last_run_at` stays null:
  1. Check `errors.log` for session ID matching `cron_<job_id>_<timestamp>`
  2. Check `cron/jobs.json` for the job's `state`, `last_error`, `last_status`
  3. Check `cron/output/<job_id>/` for generated output
  4. The job may still be running — try a `cronjob(action='list')` re-read to see updated state
- **Missing integrations**: Design cron prompts to gracefully report "missing [tool] — cannot perform [function]" rather than failing silently.
- **Fresh profile has no skills/memories**: Create user profile, memory schema, and SOUL.md before cron jobs so they have context.
- **Memory char limits**: User profile is 1375 chars, generic memory is 2200 chars. Stay within limits when saving schema.
- **Tool env vars**: Each CLI may use a different env var name — set both the canonical name and the CLI-specific name in `.env`. See `skill_view('notion', 'references/hermes-profile-env-setup.md')` for example.
- **Cron job stuck / won't start new run**: When a run is in progress, `cronjob(action='run')` returns `success: true` but the scheduler silently skips it (`already running`). See `references/cron-troubleshooting.md` for debug steps and manual fallback.
