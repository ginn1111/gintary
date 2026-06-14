# Cron Job Troubleshooting

Runtime issues encountered while running scheduled agents on Hermes.

## Run Submitted But Never Completes

You call `cronjob(action='run', job_id='xxx')` and get `success: true`, but `last_run_at` stays `null` for minutes.

### Checkpoints

1. **Is the job still running?** Read `jobs.json` — if `last_run_at` is null but the job exists, it's either running or queued.
2. **Check the scheduler tick.** The `.tick.lock` file at `cron/.tick.lock` means a scheduler tick is in progress. If it's stale (check mtime), the ticker may be stuck.
3. **Trace the agent session.** Search `errors.log` and `agent.log` for `cron_<job_id>_<timestamp>` — that's the session ID. Example:
   ```
   cron_ebebf0056ca2_20260615_010408
   ```
4. **Check API progress.** In `agent.log`, each `API call #N` log entry shows tokens in/out. Large gaps between calls mean a slow tool (web_search, web_extract waiting on network).
5. **Check output dir.** Output files land at `cron/output/<job_id>/<timestamp>.md` only after the run completes.

### "Already Running — Skipping"

The scheduler refuses to start a new run while one is in progress:

```
INFO cron.scheduler: Job 'Tech Daily Update' already running — skipping
```

**Pattern**: When you call `cronjob(action='run')` and the job was previously triggered (even by you), the new run gets silently skipped. There is no direct way to cancel a running cron session.

**Workarounds**:
- **Wait** — Cron jobs have `max_turns: 150` and `gateway_timeout: 1800` (30 min). It will eventually time out.
- **Do it manually** — If the cron job's goal is something you can do with your current tools (gather news, draft a message), use `web_search` + `send_message` directly instead of waiting for the stuck run.
- **Update prompt** — The prompt update only takes effect on the *next* run. The current run uses the prompt it started with.

### Why a Run Keeps Running

Cron jobs have no `enabled_toolsets` blacklist by default. A job with no toolset restrictions loads all default tools, which means it can call `web_search`, `web_extract`, `read_file`, etc. Slow network calls can make a 10-minute job take 20+ minutes.

Fix: Set `enabled_toolsets` to `['session_search']` or `['web']` (whichever matches the task) to limit token overhead and prevent unplanned tool use.

## Root Cause: Unresolved Placeholder in Prompt

**Symptom**: Agent searches for `*channel*` or `*CHANNEL*` patterns in files, gets regex errors.

**Cause**: A token like `#[CHANNEL_NAME]` was left in the prompt. Cron jobs run headlessly — no user can fill in the blank.

**Fix**: Remove all unresolved `<placeholder>`, `[variable]`, or `#channel` tokens from the prompt. Let the `deliver` field handle routing. The prompt should only describe *what content to produce*, not *where to send it*.

## Prompt Update That Didn't Take Effect

**Symptom**: You updated the prompt via `cronjob(action='update')`, ran the job again, and it still uses the old prompt.

**Cause**: The updated prompt is saved to `jobs.json`, but the cron session that was *already running* started with the old prompt. Agent sessions are isolated — they don't re-read `jobs.json` mid-run.

**Fix**: Wait for the current run to complete. The next run picks up the updated prompt.

## Debugging Log Locations

| File | Contents |
|------|----------|
| `logs/agent.log` | Full agent trace: tool calls, API calls, latency |
| `logs/errors.log` | Warnings and errors only (less noise) |
| `cron/jobs.json` | Job state: `last_run_at`, `last_status`, `last_error`, `last_delivery_error` |
| `cron/output/<job_id>/` | Completed output files (`<timestamp>.md`) |
| `cron/.tick.lock` | Scheduler tick lock — if stale, ticker is hung |

## Manual Fallback

When the cron job is stuck or broken and you need the output *now*:

1. Run the search/gathering task yourself with your current tools
2. Compose the message
3. Send directly via `send_message(target='<platform>')`
