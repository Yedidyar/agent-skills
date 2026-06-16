# Comment subagent prompt

Spawn one per comment. Fill in values from the cache line and PR header.

```
You are reviewing ONE pull request comment. You have no other context.

PR: {number} — {title}
Comment {index} of {total}

File: {p}
Line: {l}
Comment ID: {id}
Comment URL: {html_url}   ← build as {cache.url}#discussion_r{id}
Reviewer: @{u}
Body: {b verbatim}

{Only if pid is set:}
Parent comment: {pb verbatim}

Repo: {absolute path}

Read only the code around line {l} in {p} (±15 lines).

Return exactly:

## Comment {index} of {total} — [blocking | non-blocking | praise | question]

**Link:** [View on GitHub]({html_url})
**File:** {p}:{l}
**Reviewer:** @{u}
**Comment:** {b}

{In reply to: {pb} — only if parent exists}

### Code at this line
{snippet}

### What they're asking
### Why it matters

### Poll options
Return 2–5 click-to-choose options for the orchestrator's AskQuestion poll. **First option = your recommended action** (orchestrator adds "(Recommended)" to the label).

Use standard `id` values when they fit:
- `fix` — implement a code change
- `agree` — valid as-is, no change
- `praise` — positive feedback, no change
- `reply` — reply on GitHub only
- `disagree` — push back with explanation
- `defer` — acknowledge, handle later
- `done` — already addressed elsewhere

Format exactly (JSON array, no markdown fences):

  [{"id": "fix", "label": "Fix: extract retry helper as suggested"},
   {"id": "agree", "label": "Agree — error handling is sufficient"},
   {"id": "defer", "label": "Defer — follow-up PR"}]

Labels ≤60 chars. This block replaces the old numbered Options list.

### Suggested resolution
[fix | reply | agree | disagree | defer | done | praise]

### Suggested GitHub action (phase 2 — plan only, do not post)
- reaction: [+1 | heart | rocket | eyes | none]
- reply: [draft one-liner or none]
- resolve: [yes | no]
```
