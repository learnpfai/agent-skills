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
| [build-issue](skills/build-issue/) | Autonomously build an already-specced Linear ticket — worktree off staging, implement, push, open the PR |
| [fix-issue](skills/fix-issue/) | Autonomously fix a small Linear bug end-to-end — worktree off staging, patch, push, open the PR |
| [pr-test-checklist](skills/pr-test-checklist/) | Turn a PR into a prioritized manual QA checklist |
| [prune](skills/prune/) | Clean up a repo's worktrees and stale local branches |
| [spec-run](skills/spec-run/) | Spec a prompt into a Linear issue under Planning, then — after approval — build it in a worktree off staging and open the PR |

Which issue skill: **spec-run** when there's no ticket yet, **build-issue** when
the ticket is specced, **fix-issue** when it's a small bug.

## Add a skill

1. Create `skills/<skill-name>/SKILL.md` with YAML frontmatter (`name`, `description`) and instructions.
2. Open a PR.
3. Add a row to the table above.
