---
name: _template
description: >
  Template skill for authoring new agent skills. Do not install or
  trigger this skill in production — copy this folder when proposing a new skill.
---

# Skill name

<!--
  Authoring notes (remove this block before opening a PR):

  - The `description` in frontmatter is the TRIGGER MECHANISM. It must state BOTH
    what the skill does AND the specific contexts/phrases that should trigger it.
  - Write descriptions slightly "pushy" because agents tend to under-trigger skills.
    Example pattern:
    "Use this skill whenever the user mentions X, Y, or Z, even if they don't
    explicitly ask for ..."
  - Keep SKILL.md under ~500 lines. Move longer reference material to files in
    references/ and link to them from this file (progressive disclosure).
  - Use only the standard Agent Skills format (frontmatter + markdown). No
    tool-specific extensions.
-->

## Purpose

<!-- What does this skill teach the agent to do? One or two sentences. -->

[Describe the workflow or capability this skill provides.]

## When to use

<!-- List the trigger scenarios. Mirror key phrases from the frontmatter description. -->

- [Trigger scenario 1]
- [Trigger scenario 2]
- [Trigger scenario 3]

## Step-by-step instructions

<!-- Numbered workflow the agent should follow. -->

1. [First step]
2. [Second step]
3. [Third step]

## Examples

### Example 1: [Short title]

**User prompt:**

> [Realistic example prompt]

**Expected behavior:**

[What the agent should do when this skill triggers.]

### Example 2: [Short title]

**User prompt:**

> [Another realistic example prompt]

**Expected behavior:**

[What the agent should do.]

## Common mistakes

<!-- Anti-patterns to avoid. -->

- [Mistake 1 and how to avoid it]
- [Mistake 2 and how to avoid it]

## References

<!-- Link to files in references/ for detailed material that would make SKILL.md too long. -->

- [reference-name.md](references/reference-name.md) — [one-line description]
