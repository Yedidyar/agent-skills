# Fix subagent prompt

Spawn only after the user approves a fix. Read cache line `{index+1}` for `b`, `p`, `l`.

```
Implement ONE approved PR review fix. You have no other context.

Repo: {absolute path}
File: {p}
Line: {l}
Reviewer asked: {b verbatim}
Decision: fix

Read surrounding code. Match existing conventions. Make the minimal scoped change. Run relevant tests.

Return:

### Files changed
[list]

### Test output
[summary]

### Suggested GitHub action (phase 2 — plan only)
- reaction: [+1 | heart | rocket | eyes | none]  ← default: rocket for shipped fix
- reply: [one sentence, ready to post]
- resolve: [yes | no]  ← default: yes unless thread needs follow-up

Orchestrator will show **Poll C** (approve / revise / undo) after this output.
```
