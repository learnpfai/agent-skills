# agent-skills

Shared [Agent Skills](https://agentskills.io) for the LearnPF team — reusable
workflows for Cursor, Claude Code, and other compatible tools.

Each skill is a folder under `skills/` with a `SKILL.md` file.

## Install

Cloning only downloads the repo — it does **not** register skills with your
agent. You still need the symlink step below (or a Cursor remote rule).

### Get the repo

Only if you don't already have a local copy:

```bash
git clone git@github.com:learnpfai/agent-skills.git
# then symlink below
```

### Symlink (Cursor / Claude Code)

Replace `/path/to/agent-skills` with wherever you cloned this repo. Repeat for
each skill you want. You can run these from any directory. Installs globally so
skills work in any project.

```bash
# Cursor (global)
ln -s /path/to/agent-skills/skills/pr-test-checklist ~/.cursor/skills/pr-test-checklist

# Claude Code (global)
ln -s /path/to/agent-skills/skills/pr-test-checklist ~/.claude/skills/pr-test-checklist
```

Symlinks pick up changes after `git pull` in the agent-skills repo.

### Cursor remote rule

**Customize → Rules → Add Rule → Remote Rule (GitHub)** → `learnpfai/agent-skills`

No symlink needed; Cursor pulls skills from the repo directly.

## Skills

| Skill | Description |
| --- | --- |
| [pr-test-checklist](skills/pr-test-checklist/) | Turn a PR into a prioritized manual QA checklist |

## Add a skill

1. Create `skills/<skill-name>/SKILL.md` with YAML frontmatter (`name`, `description`) and instructions.
2. Open a PR.
3. Add a row to the table above.
