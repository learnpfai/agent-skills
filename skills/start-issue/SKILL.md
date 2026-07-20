---
name: start-issue
description: >-
  Start work on an existing Linear issue: read the ticket, fork a worktree off
  staging named after the issue, open it in the IDE with `code .`, then begin
  implementing based on the issue description. Use when the user says
  /start-issue LEA-XXX, "start on LEA-XXX", "pick up this ticket", or pastes a
  Linear issue URL and asks to start working on it.
---

# start-issue

Go from a Linear ticket to actively working on it: read the issue, spin up an
isolated worktree off `staging`, open the IDE, and start implementing what the
issue describes.

Input: a Linear issue identifier (`LEA-XXX`) or URL, given after `/start-issue`
or in the user's message. If no issue was given, ask for one — don't guess.

---

## Fixed facts for this workspace

- **Linear team**: `Learnpf` (key `LEA`).
- **Main repo**: `/Users/ryanpey/Projects/learnpf`.
- **Base branch**: `staging` (branch off `origin/staging`).
- **Worktree convention**: sibling dir `/Users/ryanpey/Projects/learnpf-lea-<NUM>`.
- **Copy-gitignored script**: `/Users/ryanpey/Projects/learnpf/ryan/copy-gitignored.sh <worktree-dir>`.

Linear tools come from the Linear MCP and are **deferred** — load them with
`ToolSearch` before calling (e.g. `get_issue`, `save_issue`,
`list_comments`). If Linear returns an **authentication error**, stop and say:
*Linear isn't connected — run `/mcp`, authenticate the linear server, then try
again.*

---

## 1. Read the ticket

Fetch the issue with `get_issue`. Capture:

- **identifier** (e.g. `LEA-742`) and **URL**
- **title** and **description** — this is the spec you'll build from
- **gitBranchName** (e.g. `lea-742-add-streak-badge`) — the single name used
  for branch and worktree
- current **state**

Also pull `list_comments` for the issue — comments often carry decisions and
constraints that amend the description.

If the description is empty or too vague to act on, stop and tell the user
what's missing instead of inventing a scope.

Let `NUM` = the issue number and `BRANCH` = the gitBranchName.

## 2. Fork a worktree off staging

From the **main repo** (`/Users/ryanpey/Projects/learnpf`):

```bash
git fetch origin
git worktree add -b <BRANCH> /Users/ryanpey/Projects/learnpf-lea-<NUM> origin/staging
```

**Idempotent — never clobber:**
- Branch already exists → `git worktree add <worktree-dir> <BRANCH>` (no `-b`).
- Worktree dir already exists → reuse it as-is.

Then make it runnable:

```bash
/Users/ryanpey/Projects/learnpf/ryan/copy-gitignored.sh /Users/ryanpey/Projects/learnpf-lea-<NUM>
(cd /Users/ryanpey/Projects/learnpf-lea-<NUM> && npm install)
```

## 3. Open the IDE

```bash
code /Users/ryanpey/Projects/learnpf-lea-<NUM>
```

## 4. Start on the task

Mark the issue in progress with `save_issue` (`id: LEA-XXX`,
`state: In Progress`) if it isn't already.

Then begin implementing **inside the worktree** — every edit, command, and
check runs against `/Users/ryanpey/Projects/learnpf-lea-<NUM>`, never the main
repo:

1. Restate the task in a couple of sentences from the issue description +
   comments, so the user can catch a misread early.
2. Explore the relevant code and form a short plan (key files, approach).
3. Implement. Treat the issue's acceptance criteria (if present) as the
   definition of done; otherwise derive done-ness from the description.
4. Verify as you go — build / lint / relevant tests / preview.

This skill **starts** the work; it does not commit, push, or open a PR on its
own. When the implementation is done and verified, report status and let the
user decide the next step (commit/push/PR).

---

## Rules

- One name throughout: the Linear `gitBranchName` is the branch and the
  worktree suffix. Don't invent variants.
- Never clobber an existing branch or worktree — reuse it.
- Don't touch the main repo's working tree.
- No commits, pushes, or PRs without the user asking.
