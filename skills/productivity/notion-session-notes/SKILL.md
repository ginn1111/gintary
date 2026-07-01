---
name: notion-session-notes
description: "Session notes and API reference for Notion workspace — search pagination, page extraction, property formats, common queries."
version: 1.0.0
author: gintary-session
license: MIT
platforms: [linux, macos, windows]
prerequisites:
  env_vars: [NOTION_API_KEY]
metadata:
  hermes:
    tags: [Notion, Productivity, API, Reference]
    homepage: https://developers.notion.com
---

# Notion Session Notes & API Reference

Local notes from gintary profile session (2026-07-01). References the bundled `notion` skill.

## Search Pagination

The Notion Search API uses cursor-based pagination.

**Request:**
```json
{
  "page_size": 50,
  "start_cursor": "optional-cursor-string"
}
```

**Response fields:**
- `next_cursor`: string or null — pass as `start_cursor` for next page
- `has_more`: boolean — true if more pages exist
- `results`: array of page or data_source objects

**CLI (ntn):** Does not support `start_cursor` directly. Use curl for pagination.

**curl pattern:**
```bash
curl -s -X POST "https://api.notion.com/v1/search" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{"page_size": 50, "start_cursor": "cursor-here"}'
```

## Extracting Page Titles from Search Results

Pages have a `title` property (type: `title`). Extract with jq:

```bash
jq -r '.results[] | select(.object=="page") | "\(.id) | \(.properties | to_entries[] | select(.value.type=="title") | .value.title[0].plain_text // "(untitled)") | \(.last_edited_time[:10])"'
```

Output format: `page_id | title | last_edited_date`

## Databases vs Data Sources (API 2025-09-03)

- **database_id**: used when creating pages (`parent: { database_id: "..." }`)
- **data_source_id**: used when querying (`POST /v1/data_sources/{id}/query`)

Search returns objects with `object: "data_source"` containing `data_source_id` field.

## Page Properties — Common Types

| Type | Read format | Write format |
|------|-------------|--------------|
| title | `{title: [{plain_text: "..."}]}` | `{title: [{text: {content: "..."}}]}` |
| rich_text | `{rich_text: [{plain_text: "..."}]}` | `{rich_text: [{text: {content: "..."}}]}` |
| select | `{select: {name: "Option", color: "blue"}}` | `{select: {name: "Option"}}` |
| multi_select | `{multi_select: [{name: "A"}, {name: "B"}]}` | `{multi_select: [{name: "A"}, {name: "B"}]}` |
| date | `{date: {start: "2026-01-01", end: null}}` | `{date: {start: "2026-01-01"}}` |
| checkbox | `{checkbox: true}` | `{checkbox: true}` |
| number | `{number: 42}` | `{number: 42}` |
| url | `{url: "https://..."}` | `{url: "https://..."}` |
| relation | `{relation: [{id: "page-id"}]}` | `{relation: [{id: "page-id"}]}` |
| rollup | `{rollup: {array: [...], function: "show_original"}}` | (read-only) |
| formula | `{formula: {string: "..."}}` | (read-only) |
| created_time | `{created_time: "2026-01-01T00:00:00Z"}` | (read-only) |
| last_edited_time | `{last_edited_time: "..."}` | (read-only) |

## Reading Page Content

- **Markdown (agent-friendly)**: `GET /v1/pages/{page_id}/markdown`
- **Blocks (structured)**: `GET /v1/blocks/{page_id}/children`

## Creating Pages

```bash
# Via ntn (supports markdown body)
ntn api v1/pages \
  parent[page_id]=xxx \
  properties[title][0][text][content]="Title" \
  markdown="# Content"

# Via curl
curl -s -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2025-09-03" \
  -H "Content-Type: application/json" \
  -d '{"parent": {"page_id": "xxx"}, "properties": {"title": [{"text": {"content": "Title"}}]}, "markdown": "# Content"}'
```

## File Uploads

**ntn (one-liner):**
```bash
ntn files create < file.png
ntn files create --external-url https://example.com/file.png
```

**curl (3-step):**
1. POST `/v1/file_uploads` with `{filename, content_type}` → returns `upload_url`
2. PUT bytes to `upload_url`
3. Reference `file_upload_id` in page/block payload

## Rate Limits

~3 requests/second average. CLI does not bypass this.

## Common Pitfalls

- **ntn reads `NOTION_API_TOKEN`**, not `NOTION_API_KEY` — set both in .env
- **Single-quote heredoc** prevents env expansion: use `cat >> .env << EOF` (unquoted)
- **Search returns `data_source` objects for databases** — use `data_source_id` for queries
- **Cannot set database view filters via API** — UI only
- **Use `is_inline: true`** when creating data sources to embed in a page

## Session 2026-07-01: Full Workspace Scan

Scanned entire Notion workspace via paginated search (9 pages, ~400 pages total). Key databases found:
- Main task database: `data_source_id: 16abc2f1-2780-4b57-8477-d45462052055` (database_id: `0585ef26-bd3a-4e3a-8b2f-cfcef7b65eec`)
- Project area relations: `33f54e82-2b34-80de-a121-e86a019f798a` (Foreign languages), `37b54e82-2b34-8088-9545-f5eb60723c9d` (Hermes agent), etc.
- Smart List formula field categorizes tasks: "Calendar", "Do Next", "Backlog"
- Kanban Status select: "To Do", "Backlog", "In Progress", "Done"
- Priority select: "🧊 Low", "🔥 High", "⚡ Urgent"

Top-level pages found: "Mission dashboard", "Process", "All Tasks" (data source)