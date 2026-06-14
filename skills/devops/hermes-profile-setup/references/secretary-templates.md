# Secretary Output Templates

Reusable templates for executive assistant / secretary agent profiles.

## Morning Brief

```
## Morning Brief — {date}

### 📅 Calendar Today
- {event list or "No calendar connected — cannot read"}

### 🎯 Top Priorities
1. {priority 1}
2. {priority 2}
3. {priority 3}

### ⚠️ Urgent Messages
- {urgent items or "No email connected — cannot scan"}

### 📬 Follow-Ups Due
- {follow-ups or "None tracked"}

### 📋 Meeting Prep Needed
- {meetings needing prep or "None"}

### 🔥 Risks / Conflicts
- {risks or "None identified"}

### ✅ Recommended First Action
- {action}
```

## Meeting Brief

```
## Meeting Brief — {Meeting Name}

- **Time:** {start} — {end}
- **Attendees:** {list}
- **Objective:** {objective}
- **Context:** {background}
- **Recent related messages:** {messages or "none"}
- **Open tasks:** {tasks or "none"}
- **Suggested talking points:**
  1. {point 1}
  2. {point 2}
- **Decisions needed:** {decisions or "none"}
```

## End-of-Day Wrap

```
## End-of-Day Wrap — {date}

### ✅ Completed
- {completed items}

### 🔄 Still Open
- {open items}

### ⏳ Waiting on Others
- {waiting-on items}

### ⚠️ Risks
- {risks or "none"}

### 🚀 Tomorrow's First Actions
- {action 1}
- {action 2}
```

## Follow-Up Draft

```
**Subject:** {Context}
**To:** {Name}

Hi {Name},

{1-line context}. {Clear ask}. {Deadline if relevant}.

Best,
{User Signature}
```

## Priority Classification

| Level | Meaning | Examples |
|-------|---------|---------|
| P0 | Urgent, blocks outcome | Same-day deadline, blocked exec decision, major client issue, meeting soon w/o prep |
| P1 | Important, handle soon | Client follow-up, deadline this week, decision needed |
| P2 | Useful, not urgent | Admin cleanup, docs, non-urgent scheduling |
| P3 | Optional/backlog | Nice-to-have, low-value FYI |

## Approval Gates

| Category | Actions |
|----------|---------|
| ✅ Allowed w/o approval | Read email metadata, read calendar availability, read user-owned docs, summarize threads, draft replies, draft agendas, draft follow-ups, create private notes, suggest tasks |
| 🔒 Approval required | Send email/chat, create/accept/decline calendar invites, edit shared docs, update CRM, submit forms, book travel, make purchases, delete anything |
| 🚫 Forbidden | git push to remote, reveal credentials, legal/financial/medical/HR final decisions, change security settings, commit user to contracts |
