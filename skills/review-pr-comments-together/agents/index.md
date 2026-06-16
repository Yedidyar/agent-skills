# Index subagent prompt

Spawn this subagent once per cache miss. It fetches the PR and writes the cache file.

```
Load the PR comment index. You have no other context.

Repo: {absolute path}
PR number: {number, or omit for current branch}

Run from repo root:

  owner_repo=$(gh repo view --json nameWithOwner -q .nameWithOwner)
  owner="${owner_repo%%/*}"
  repo="${owner_repo##*/}"
  pr=$(gh pr view {number_or_blank} --json number -q .number)
  head=$(gh pr view {number_or_blank} --json headRefOid -q .headRefOid)

  gh api graphql -f query='
    query($owner: String!, $repo: String!, $pr: Int!) {
      repository(owner: $owner, name: $repo) {
        pullRequest(number: $pr) {
          number title url state
          reviewThreads(first: 100) {
            nodes {
              isResolved
              comments(first: 50) {
                nodes {
                  databaseId body path
                  line: originalLine
                  author { login }
                  replyTo { databaseId body }
                }
              }
            }
          }
        }
      }
    }' -f owner="$owner" -f repo="$repo" -F pr="$pr"

Filter to isResolved == false threads. Use the first comment of each thread as the index entry.

Write `.cursor/local/review-pr-cache.jsonl` (create dir if needed). Overwrite on refresh.

Line 1 — header:
{"v":1,"pr":{number},"title":"{title}","url":"{pr_url}","state":"{state}","head":"{head}","fetched":"{ISO8601}","m":{M}}

Lines 2…M+1 — one object per unresolved comment (short keys, verbatim body):
{"i":1,"id":{databaseId},"p":"{path}","l":{line},"u":"{author.login}","b":"{body}","pid":{replyTo.databaseId|null},"pb":{replyTo.body|null}}

Return only:
- PR number, title, url, state
- M (total unresolved)
- Slim list for progress init: [{i, p, l, u}] — no bodies
```
