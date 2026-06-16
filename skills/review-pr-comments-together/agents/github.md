# GitHub subagent prompt

Spawn one per comment in phase 2. Use `subagent_type: shell` since the job is only `gh` commands.
Read planned actions from the progress row (`~` prefix = execute now; no `~` = already posted, skip).
Reply text comes from the `n` column or fix/reply subagent output.

```
Post GitHub feedback for ONE pull request comment. You have no other context.

Repo: {absolute path}
PR: {number}
Comment {index} of {total}
Comment ID: {id}

Planned actions (run only what is listed):
- react: {+1 | heart | rocket | eyes | none}
- reply: {exact approved text, or none}
- resolve: {yes | no}

Run from repo root:

  owner_repo=$(gh repo view --json nameWithOwner -q .nameWithOwner)
  owner="${owner_repo%%/*}"
  repo="${owner_repo##*/}"

React (skip if none):
  gh api "repos/${owner_repo}/pulls/comments/{id}/reactions" \
    -X POST -f "content={reaction}" \
    --jq '{id, content, user: .user.login}'

Reply (skip if none):
  gh api "repos/${owner_repo}/pulls/{pr}/comments" \
    -X POST --input - <<EOF
  {"body": "{text}", "in_reply_to": {id}}
  EOF

Resolve (skip if no):
  thread_id=$(gh api graphql -f query='
    query($owner: String!, $repo: String!, $pr: Int!) {
      repository(owner: $owner, name: $repo) {
        pullRequest(number: $pr) {
          reviewThreads(first: 100) {
            nodes {
              id
              comments(first: 50) { nodes { databaseId } }
            }
          }
        }
      }
    }' -f owner="$owner" -f repo="$repo" -F pr={pr} \
    --jq ".data.repository.pullRequest.reviewThreads.nodes[]
      | select(.comments.nodes | map(.databaseId) | index({id}))
      | .id" | head -n 1)

  gh api graphql -f query='
    mutation($threadId: ID!) {
      resolveReviewThread(input: {threadId: $threadId}) {
        thread { id isResolved }
      }
    }' -f threadId="$thread_id" \
    --jq '.data.resolveReviewThread.thread'

Reaction reference:
  +1 = agree/LGTM · heart = praise/thanks · rocket = fix shipped · eyes = will look later
  Valid values: +1, -1, laugh, confused, heart, hooray, rocket, eyes
  HTTP 201 = created; 200 = already reacted.

Return:

### Actions taken
- react: {emoji or skipped}
- reply: {html_url or skipped}
- resolve: {resolved | skipped}

### Errors
[any failures, or "none"]
```
