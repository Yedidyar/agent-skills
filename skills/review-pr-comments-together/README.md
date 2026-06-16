# Review PR Comments Together

Interactive walkthrough for pull request review comments — built to **learn from feedback**, not to auto-fix a PR and move on.

## Origin story

This skill came out of working through a real PR review: a Terraform Provider where a reviewer with a couple of years of Go experience left dozens of line-level comments.

At first the plan was to let an agent handle the whole review. That would work, but I would not learn much about *why* the feedback was given or what to watch for on the next PR.

The other option was to go through every comment manually in chat. That teaches you, but it is slow, repetitive, and easy to lose track of what you already decided.

So I built something in the middle: a skill that goes to GitHub, reads the comments, and walks through them with me **one at a time** — explaining what the reviewer likely means (a lot of comments are terse or assume context I do not have), suggesting concrete improvements, and asking what I want to do before anything gets changed or posted.

## How it works

1. **Fetch once, work locally** — an index subagent pulls comments from GitHub into a local cache. No re-fetching on every comment.
2. **Explain, then ask** — for each comment, a dedicated subagent reads the nearby code, explains the feedback, and proposes options. The orchestrator runs a poll so I choose — fix, agree, reply, defer — instead of typing free-form answers every time.
3. **Learn while shipping** — I stay in the loop on every decision; the agent does the heavy lifting (analysis, patches, reply drafts, `gh` in phase 2).

## Overview flow

The skill has two distinct phases separated by a checkpoint poll. Phase 1 is entirely local — no GitHub calls — so you can think, revise, and change your mind freely. Phase 2 is when decisions actually get posted.

```mermaid
flowchart TD
    Start([User starts skill]) --> CacheCheck{Cache exists\nfor this PR?}
    CacheCheck -->|No| Index[Index subagent\ngh fetch → cache.jsonl]
    CacheCheck -->|Yes| PollA{Poll A\nResume / Refresh / Restart}
    PollA -->|Refresh| Index
    PollA -->|Restart| InitProgress
    PollA -->|Resume| InitProgress
    Index --> InitProgress[Init / merge progress.md]
    InitProgress --> Phase1

    subgraph Phase1["Phase 1 — Triage (local only, no gh calls)"]
        direction TB
        CommentLoop[Comment subagent\nanalyze comment + code] --> PollB{Poll B\nFix / Agree / Reply / …}
        PollB -->|fix| Fix[Fix subagent\npatch + tests]
        PollB -->|agree / done| Record[Patch progress.md]
        PollB -->|reply / disagree| Reply[Reply subagent\ndraft text]
        Fix --> PollC{Poll C\nApprove / Revise / Undo}
        PollC -->|revise| Fix
        PollC -->|undo| PollB
        PollC -->|approve| Record
        Reply --> PollE{Poll E\nApprove / Revise}
        PollE -->|revise| Reply
        PollE -->|approve| Record
        Record --> MoreComments{More\ncomments?}
        MoreComments -->|Yes| CommentLoop
    end

    MoreComments -->|No| PollF{Poll F\nTriage complete}
    PollF -->|stop| EndStop([Session saved\npost later])
    PollF -->|github| Phase2

    subgraph Phase2["Phase 2 — GitHub (gh api calls)"]
        direction TB
        GHStart[Per-comment GitHub subagent] --> PollG{Poll G\nPost / Skip / Edit}
        PollG -->|post| GH[react · reply · resolve]
        PollG -->|skip| GHNext{More?}
        GH --> GHNext
        GHNext -->|Yes| GHStart
    end

    GHNext -->|No| PollH{Poll H\nPush / Done}
    PollH --> EndDone([Session complete])
```

The key design choice is the **poll at every decision point**. Rather than having the agent decide on your behalf, each poll hands control back to you — and because no GitHub calls happen until Phase 2, every decision in Phase 1 is reversible.

## Triage decision loop

Each comment goes through the same branching loop. The three main paths — fix, agree, reply — each have their own approve/revise cycle before anything is recorded.

```mermaid
flowchart TD
    Entry([Comment subagent\nanalyzes comment + code]) --> PollB{Poll B\nWhat do you want to do?}

    PollB -->|agree · praise · done| RecordAgree[Record: plan reaction only]
    PollB -->|fix| FixSub[Fix subagent\npatch + tests]
    PollB -->|reply · disagree · defer| ReplySub[Reply subagent\ndraft response text]

    FixSub --> PollC{Poll C\nApprove / Revise / Undo}
    PollC -->|revise| FixSub
    PollC -->|undo| PollB
    PollC -->|approve| PollD{Poll D\nResolve thread?}
    PollD --> RecordFix[Record: plan reaction + reply + resolve flag]

    ReplySub --> PollE{Poll E\nApprove / Revise}
    PollE -->|revise| ReplySub
    PollE -->|approve| RecordReply[Record: plan reply + draft text]

    RecordAgree --> Next{Next comment?}
    RecordFix --> Next
    RecordReply --> Next
    Next -->|Yes| Entry
    Next -->|No| PollF([Poll F — triage complete])
```

A few things worth noting about this loop:

- **Undo is always available** — Poll C lets you go back to Poll B if the fix subagent's patch isn't what you wanted. Nothing is committed until you approve.
- **Fix and reply are independent** — the skill doesn't assume a fix means you don't also want to reply, or vice versa. The planned GitHub actions are recorded separately and composed in Phase 2.
- **Each subagent is isolated** — the fix subagent only sees the one comment and its surrounding code. It doesn't know what decisions you made on previous comments. This keeps each analysis clean and avoids context bleed.

## Architecture

The key idea is **subagent-first**: anything that is not polling or bookkeeping runs outside the orchestrator.

| Piece | Role |
|-------|------|
| **Orchestrator** | Polls, decisions, progress file, spawns subagents |
| **Index** | Fetch PR comments → cache |
| **Comment** | Analyze one comment + nearby code |
| **Fix** | Patch + tests for one comment |
| **Reply** | Draft reply for one comment |
| **GitHub** | Post reactions, replies, resolves (phase 2) |

Each subagent gets a **fresh context**. The orchestrator never loads the full cache, never reads code, never calls `gh`. That keeps the main session's context window very small — almost no token waste on work that does not need to be there.

## Local cache & resume

Session state lives under `.cursor/local/` (gitignored, never committed):

| File | Purpose |
|------|---------|
| `review-pr-cache.jsonl` | Immutable comment data from GitHub |
| `review-pr-progress.md` | Decisions, planned GitHub actions, resume pointer |

Because comments and progress are cached:

- No need to hit GitHub again on every comment
- Jump to a new chat window and say **resume** — pick up at the first unfinished comment
- Start over without losing the fetched comments

Even when the original orchestrator session's context gets too large, the cache + progress file let you continue in a clean window.

## Two phases

| Phase | What happens |
|-------|----------------|
| **1 — Triage** | Analyze every comment, decide, implement fixes locally, draft replies. GitHub is planned only (`~` codes in progress), not called. |
| **2 — GitHub** | Post planned reactions, replies, and resolves — one GitHub subagent per comment. |

The phase boundary is intentional: you can finish all of Phase 1 in one sitting, review the planned actions in `progress.md`, and only then run Phase 2 — or hand it off to a different session entirely.

## Usage

Install from this repo (see root [README](../../README.md)), then in any project with an open PR:

- "Walk me through the PR review"
- "Go over comments one by one"
- "Let's resolve comments together"

Agent instructions: [`SKILL.md`](SKILL.md)