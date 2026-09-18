---
name: fix-issue
description: >-
  Autonomously fix a small Linear bug or issue end-to-end (not features): read
  the ticket, fork a worktree off staging (npm install included), patch the bug, push,
  and open a PR against staging. Use for narrow regressions, broken behavior,
  and small fixes when the user says /fix-issue LEA-XXX or pastes a Linear URL.
  Do not use for new features, large refactors, or multi-surface work — use
  spec-run instead.
---

# fix-issue

Run the full loop on a **small bug or issue** without stopping for permission:
read the ticket, spin up an isolated worktree off `staging`, fix it, push, and
open a PR. This skill is not for features or large work — stop and point the
user at **spec-run** if the ticket is net-new capability, spans many areas, or
needs a written spec first.

Input: a Linear issue identifier (`LEA-XXX`) or URL, given after `/fix-issue`
or in the user's message. If no issue was given, ask for one — don't guess.

**Autonomous:** Do not pause for approval between steps. Do not open the IDE with
`code`. Always run `npm install` in the worktree before verifying. Commit,
push, and create the PR yourself when the fix is verified.

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
- **title** and **description** — this is the spec you'll fix from
- **gitBranchName** (e.g. `lea-742-add-streak-badge`) — the single name used
  for branch and worktree
- current **state**

Also pull `list_comments` for the issue — comments often carry decisions and
constraints that amend the description.

If the description is empty or too vague to act on, stop and tell the user
what's missing instead of inventing a scope.

**Scope gate — small bugs only.** Proceed only if the ticket is a focused fix
(regression, incorrect behavior, small UI/logic bug, config typo, obvious
one-area patch). **Stop** (do not worktree or PR) and tell the user to use
**spec-run** instead when the ticket is a feature, epic, redesign, migration,
or likely touches many files / needs product spec beyond the description.

Let `NUM` = the issue number and `BRANCH` = the gitBranchName.

**Name the chat** as soon as you have the issue title (especially in a new
Claude Code / Agent chat). Rename the conversation in the sidebar to:

`<issue title> [LEA-XXX] <fix summary>`

- Use the Linear **title** verbatim (or trimmed only if absurdly long).
- Put **`[LEA-XXX]`** immediately after the title.
- **Fix summary**: fewer than 3 words — what you're fixing, not the full spec.

Example: `Streak badge missing on profile [LEA-742] toast not showing`

If you cannot rename programmatically, tell the user the exact string to paste
when renaming, and use that same string as the first line of your reply so
auto-titles pick it up when the product supports that.

Mark the issue in progress with `save_issue` (`id: LEA-XXX`,
`state: In Progress`) if it isn't already.

## 2. Fork a worktree off staging

From the **main repo** (`/Users/ryanpey/Projects/learnpf`):

```bash
git fetch origin
git worktree add --no-track -b <BRANCH> /Users/ryanpey/Projects/learnpf-lea-<NUM> origin/staging
```

`--no-track` matters: without it the new branch tracks `origin/staging`, so a
plain `git push` from the worktree refuses to push (upstream name mismatch).
The branch gets its real upstream on first push (`git push -u origin HEAD`).

**Idempotent — never clobber:**
- Branch already exists → `git worktree add <worktree-dir> <BRANCH>` (no `-b`).
- Worktree dir already exists → reuse it as-is.

Copy gitignored local files, then install dependencies (always a real
`npm install` — never symlink or clone `node_modules` from the main repo):

```bash
/Users/ryanpey/Projects/learnpf/ryan/copy-gitignored.sh /Users/ryanpey/Projects/learnpf-lea-<NUM>
cd /Users/ryanpey/Projects/learnpf-lea-<NUM> && npm install
```

## 3. Fix the bug

Implement **inside the worktree** — every edit, command, and check runs against
`/Users/ryanpey/Projects/learnpf-lea-<NUM>`, never the main repo:

1. Explore the relevant code and form a short plan (key files, root cause,
   approach).
2. Implement the fix. Treat the issue's acceptance criteria (if present) as the
   definition of done; otherwise derive done-ness from the description.
3. Verify before push — build / lint / relevant tests / preview as appropriate.
   Do not push a red tree.

If you cannot reproduce or fix without more product input, stop and report what
you tried and what's blocking — do not open an empty or speculative PR.

## 4. Push and open the PR

From the worktree, commit with a clear imperative message, push, and open the
PR with `gh` (base `staging`):

```bash
cd /Users/ryanpey/Projects/learnpf-lea-<NUM>
git add -A
git commit -m "<imperative summary>

<short body>"
git push -u origin HEAD
gh pr create --base staging --head <BRANCH> \
  --title "<issue title>" \
  --body "$(cat <<'EOF'
## Summary
<what changed and why, in two or three sentences>

<LEA-XXX>: <issue URL>

## Core decisions
- <only if the fix involved a real choice; omit the section otherwise>
EOF
)"
```

Keep the PR body to exactly those two sections. No acceptance-criteria
checklist, test plan, out-of-scope list, design notes, or setup steps: Ryan
reads the ticket for criteria and the diff for detail. A decision earns a bullet
only if a reviewer would otherwise ask "why this way?". Attach screenshots under
the summary for anything user-visible.

Optionally move the issue to `In Review` with `save_issue` (`id: LEA-XXX`,
`state: In Review`) once the PR is open.

## 5. Report

Give the user, in a few lines: worktree path, branch, commit summary, and PR
URL.

---

## Rules

- **Small bugs and issues only** — not features; defer big work to spec-run.
- Name every session chat: `<issue title> [LEA-XXX] <≤3-word fix summary>`.
- One name throughout: the Linear `gitBranchName` is the branch and the
  worktree suffix. Don't invent variants.
- Never clobber an existing branch or worktree — reuse it.
- Don't touch the main repo's working tree.
- Always run `npm install` in the worktree after creating it. Never symlink or
  APFS-clone `node_modules` from the main repo.
- Do not open the IDE with `code` unless the user asks separately.
- Only create commits when implementing this workflow; follow the user's git
  safety rules (no force push to main/staging, no skipping hooks unless asked).
