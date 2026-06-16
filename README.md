# Yedidyar/agent-skills

Personal library of AI agent skills for Claude Code, Codex CLI, and Cursor. Skills follow the open [Agent Skills standard](https://agentskills.io) (`SKILL.md` format) so the same skill works across all three tools without modification.

## What belongs here

Cross-project workflows you reuse regardless of which repo you are working in — for example, a deploy checklist, a security review process, or a standard PR description format.

## What does not belong here

Repo-specific knowledge belongs in that repo's own skill directory and `AGENTS.md`:

- `.claude/skills/` — Claude Code project skills
- `.cursor/skills/` — Cursor project skills
- `AGENTS.md` — repo-specific agent instructions

This central repo is only for skills that apply across projects, not tied to a single codebase.

## Install skills

Skills are installed with the [skills CLI](https://skills.sh) (`npx skills`).

### Install a single skill

```bash
npx skills add Yedidyar/agent-skills/<skill-name>
```

Equivalent forms:

```bash
npx skills add Yedidyar/agent-skills --skill <skill-name>
npx skills add Yedidyar/agent-skills@<skill-name>
```

### Discover available skills

List skills in this repo without installing:

```bash
npx skills add Yedidyar/agent-skills --list
```

### Install globally

Install a skill for all projects on your machine:

```bash
npx skills add Yedidyar/agent-skills --skill <skill-name> -g
```

### Manage installed skills

```bash
# List skills you have installed
npx skills list

# Update installed skills to latest versions
npx skills update
```

## How skills get discovered

The skills CLI installs skills into the directories each tool reads automatically. If you prefer manual installation, copy the skill folder to the appropriate path:

| Tool | Path |
|------|------|
| Claude Code | `~/.claude/skills/` |
| Codex CLI | `~/.codex/skills/` |
| Cursor | `~/.cursor/skills/` |

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to propose a new skill or change an existing one.

## Available skills

Keep this table updated in every skill PR.

| Skill | Description |
|-------|-------------|
| `_template` | Starter template for authoring new skills (not for production use) |
