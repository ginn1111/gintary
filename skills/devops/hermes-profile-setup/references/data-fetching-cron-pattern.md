# Data-Fetching Cron Job Pattern

Run standalone Python scripts on a schedule to fetch/process external data and deliver results to the user.

## Pattern Steps

1. **Write a standalone Python script** to `~/.hermes/profiles/<profile>/scripts/`

   - No external deps beyond stdlib (urllib, json, datetime)
   - Output formatted markdown to stdout (delivered verbatim)
   - Handle errors gracefully: print error message, exit 1

2. **Test the script** before wiring it to cron

3. **Create the cron job**:

   ```python
   cronjob(
       action='create',
       name='<descriptive-name>',
       schedule='<cron-expr>',   # e.g. "0 2 * * *"
       prompt='Run <script_path> with python3. Deliver script stdout as final response.',
       enabled_toolsets=['terminal'],  # script handles logic; agent just runs it
       # Omit repeat param for recurring jobs!
   )
   ```

## Key Rules

- **Omit `repeat` for recurring schedules** — passing `repeat=false` creates a one-shot job. For forever-running cron schedules, just don't set `repeat`.
- **`enabled_toolsets=['terminal']`** when the script does the work — saves tokens.
- **Prompt = simple instruction** to run the script; don't overdescribe what the script does.
- **Script lives in `/scripts/`** under the profile directory.

## Example: GitHub Trending

The `github-trending.py` script at `~/.hermes/profiles/gintary/scripts/github-trending.py`:

- Fetches top 10 repos by stars (created in last 7 days) via GitHub Search API
- Outputs markdown table with rank, repo name+link, stars, language, description
- Runs daily at 09:00 +07 via cron job `github-trending-daily`

Cron config:

```yaml
name: github-trending-daily
schedule: "0 2 * * *"
prompt: "Run /home/aioz/.hermes/profiles/gintary/scripts/github-trending.py with python3. Deliver the script stdout as your final response."
enabled_toolsets: [terminal]
repeat: <omitted>  # forever by default for cron schedules
```
