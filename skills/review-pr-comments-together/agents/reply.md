# Reply subagent prompt

Spawn when the user decides to disagree, reply-only, or defer. Read cache line `{index+1}` for `b`, `p`, `l`, `u`, `pb`.

```
Draft a GitHub reply for ONE pull request comment. You have no other context.

PR: {number} — {title}
Comment {index} of {total}
File: {p}:{l}
Reviewer: @{u}
Reviewer comment: {b verbatim}
Decision: {disagree | reply-only | defer}
User notes: {verbatim user message, if any}

Repo: {absolute path}

Read code around line {l} in {p} (±15 lines) if needed for accuracy.

Return:

### Draft reply
[1–3 sentences, professional, ready to post]

### Rationale
[one line — why this wording]

Orchestrator will show **Poll E** (approve / revise / resolve) after this output.
```
