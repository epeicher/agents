# agents

Personal skills and agent configurations for [Claude Code](https://claude.com/claude-code).

## Layout

Each top-level directory is a self-contained agent or skill:

- `gh-notifications/` — project-scoped agent with a `CLAUDE.md` that triages GitHub notifications.
- `pr-reviewer/` — reusable skill (`SKILL.md`) for reviewing WordPress Studio PRs.

## Usage

### Project-scoped agents (e.g. `gh-notifications`)

Run Claude Code from inside the directory. Its `CLAUDE.md` is loaded automatically as project instructions:

```sh
cd gh-notifications
claude
```

### Skills (e.g. `pr-reviewer`)

Skills are invoked by name. To make a skill available globally, symlink it into your user skills directory:

```sh
ln -s "$(pwd)/pr-reviewer" ~/.claude/skills/pr-reviewer
```

Then invoke it from any Claude Code session:

```
/pr-reviewer
```

Claude will also trigger it automatically when the request matches the skill's `description`.

## Adding a new agent or skill

1. Create a new directory at the repo root.
2. Add a `CLAUDE.md` (for a project agent) or a `SKILL.md` with frontmatter (for a skill).
