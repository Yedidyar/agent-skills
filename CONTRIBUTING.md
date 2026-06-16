# Contributing

Thank you for improving this agent skills library. This document covers the rules, lifecycle, and quality bar for skills in this repo.

## Portability rules

Skills in this repo must work in Claude Code, Codex CLI, and Cursor without modification.

- **Core format only.** Use standard YAML frontmatter (`name`, `description`) plus markdown. No tool-specific extensions — no Claude Code context forking, no Codex `openai.yaml`, no Cursor-only fields.
- **Test across tools.** Before merging, test in at least Claude Code and Cursor. If the skill bundles or references executable scripts, also test in Codex (which runs sandboxed by default).
- **Naming convention.** Use kebab-case folder names under `skills/`. Names should describe the workflow, not the tool (e.g. `deploy-checklist`, not `claude-deploy`).
- **One skill = one folder.** Each skill lives in its own directory with a `SKILL.md` at the root.
- **No repo-specific knowledge.** Skills that only apply to one codebase belong in that repo's `.claude/skills/` or `.cursor/skills/` directory and `AGENTS.md`, not here.
- **Deprecation over deletion.** Mark a skill as deprecated in its `description` first. Remove it only after a grace period so you have time to update installed copies.

## Skill lifecycle

1. **Propose** — Copy `skills/_template/` to `skills/<your-skill-name>/`, fill in `SKILL.md`, and open a PR.
2. **Review** — A code owner reviews the PR against the checklist in the pull request template.
3. **Merge** — Once approved, the skill is available on `main`.
4. **Use** — Install or update the skill on your machines with `npx skills update`.

## Writing good trigger descriptions

The `description` field in frontmatter is how agents decide whether to use a skill. It must state **both** what the skill does **and** the specific contexts or phrases that should trigger it. Write descriptions slightly "pushy" — agents tend to under-trigger skills.

### Good example

> Use this skill whenever the user mentions deploy, release, or production rollout, even if they don't explicitly ask for a checklist. Runs the org deploy verification workflow before merge.

This works because it names concrete trigger phrases and explains the outcome.

### Bad example

> Helps with deployments.

This is too vague. No trigger phrases, no outcome — the agent will rarely invoke it.

## Testing before you open a PR

Before opening a PR, run the skill against 2–3 realistic prompts in at least Claude Code. Confirm that:

1. The skill triggers when expected.
2. The agent follows the step-by-step instructions correctly.
3. The output matches what you intended.

If the skill includes or references scripts, repeat the same tests in Codex to confirm behavior in a sandboxed environment.

## File size and progressive disclosure

Keep `SKILL.md` under ~500 lines. If you need more detail — long reference docs, API schemas, example outputs — put them in a `references/` subdirectory and link to them from `SKILL.md`. The agent reads linked files only when needed.

## Checklist

Every skill PR must satisfy the items in [.github/pull_request_template.md](.github/pull_request_template.md), including updating the skills table in [README.md](README.md).
