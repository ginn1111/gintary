# Notion in Hermes Profile — .env Setup & Verification

## Profile vs Global .env

When running inside a Hermes profile session, `~` resolves to `~/.hermes/profiles/<name>/home/`. The .env file that Hermes loads is at:

```
~/.hermes/profiles/<name>/.env   # profile-level — loaded by Hermes
~/.hermes/.env                    # global fallback
```

Always set tool env vars in the **profile-level** `.env`.

## ntn CLI env vars

Add to `~/.hermes/profiles/<name>/.env`:

```bash
NOTION_API_TOKEN=ntn_yo...here      # ntn reads NOTION_API_TOKEN
NOTION_KEYRING=0                         # skip OS keychain
```

**Why both NOTION_API_KEY and NOTION_API_TOKEN?** Some tooling reads `NOTION_API_KEY` (Hermes skills, curl scripts) and `ntn` reads `NOTION_API_TOKEN`. Set both to the same value so both paths work.

## Heredoc pitfall

When appending to `.env` with a heredoc, single-quoted delimiters prevent variable expansion:

```bash
# BAD — single quotes: $NOTION_API_KEY stays literal text
cat >> .env << 'EOF'
NOTION_API_TOKEN=$NOT...not expanded!
EOF

# GOOD — no quotes: $NOTION_API_KEY expands to the value
cat >> .env << EOF
NOTION_API_TOKEN=$NOT...
EOF
```

If you already wrote the literal string, fix by reading the key with Python and rewriting the line:

```python
with open("/Users/gin/.hermes/profiles/<name>/.env") as f:
    lines = f.readlines()

api_key = next(line.strip().split("=", 1)[1] for line in lines if line.startswith("NOTION_API_KEY=***        # find key

for i, line in enumerate(lines):
    if i == 5 and 'NOTION_API_TOKEN' in line:  # adjust index
        lines[i] = f"NOTION_API_TOKEN={ap.... # write actual value
        break

with open("/Users/gin/.hermes/profiles/<name>/.env", "w") as f:
    f.writelines(lines)
```

## Verification pattern (workaround terminal masking)

The terminal tool **masks** output containing API key patterns (`***`), making `echo $NOTION_API_KEY` confusing. Verify from `execute_code`:

```python
import subprocess, os, json

with open("/Users/gin/.hermes/profiles/gintary/.env") as f:
    for line in f:
        if line.startswith("NOTION_API_TOKEN=***            token = line.strip().split("=", 1)[1]

env = os.environ.copy()
env["NOTION_API_TOKEN"] = token
env["NOTION_KEYRING"] = "0"

result = subprocess.run(
    ["ntn", "api", "v1/search", "--data", "{}"],
    capture_output=True, text=True, timeout=15, env=env
)
d = json.loads(result.stdout)
print(f"OK: {d['object']} ({len(d['results'])} results)")
```

## ntn CLI flags (quick ref)

| Action | Command |
|--------|---------|
| Search | `ntn api v1/search --data '{"query": ""}'` |
| POST body | `--data <json>` or stdin |
| PATCH | `-X PATCH` |
| Typed values | `key:=true` (bool), `key:=42` (number) |

## Token type guard

- `ntn_*` prefix = Integration token (headless, works via `NOTION_API_TOKEN`)
- `secret_*` prefix = Legacy internal integration token
- If `ntn` says "API token is invalid" but curl works: `ntn` reads `NOTION_API_TOKEN`, not `NOTION_API_KEY`

## Integration access

Integration must be explicitly shared with each page/database:
1. Open page in Notion
2. `···` → `Connect to` → Select your integration name
3. Without this, API returns 404 even though page exists
