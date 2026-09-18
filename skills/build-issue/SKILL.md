---
name: build-issue
description: >-
  Autonomously build an already-specced Linear ticket end-to-end (features, not
  bugs): read the ticket, fork a worktree off staging (npm install included),
  implement the spec, push, and open a PR against staging. Use for features,
  refactors,
  copy passes, and multi-surface work when the user says /build-issue LEA-XXX or
  pastes a Linear URL for work that is already written up. Use fix-issue instead
  for narrow bug fixes; use spec-run instead when the ticket does not exist yet
  or has no real spec.
---

# build-issue

Run the full loop on a **ticket that is already specced** without stopping for
permission: read the ticket, spin up an isolated worktree off `staging`, build
it, push, and open a PR.

This is the sibling of **fix-issue**, for everything fix-issue turns away —
features, refactors, copy passes, migrations, multi-surface work. The trade is
that the ticket must carry a real spec, because there is no human gate here to
catch a misread.

Pick the right skill:

| Situation | Skill |
| --- | --- |
| Narrow bug, regression, one-area patch | **fix-issue** |
| Ticket exists and is specced, work is bigger than a bug | **build-issue** (this) |
| No ticket yet, or the ticket is a one-line stub | **spec-run** |

Input: a Linear issue identifier (`LEA-XXX`) or URL, given after `/build-issue`
or in the user's message. If no issue was given, ask for one — don't guess.

**Autonomous:** Do not pause for approval between steps. Do not open the IDE with
`code`. Always run `npm install` in the worktree before verifying. Commit,
push, and create the PR yourself when the work is verified.

---

## Fixed facts for this workspace

- **Linear team**: `Learnpf` (key `LEA`).
- **Main repo**: `/Users/ryanpey/Projects/learnpf`.
- **Base branch**: `staging` (branch off `origin/staging`, PR targets `staging`).
- **Worktree convention**: sibling dir `/Users/ryanpey/Projects/learnpf-lea-<NUM>`.
- **Copy-gitignored script**: `/Users/ryanpey/Projects/learnpf/ryan/copy-gitignored.sh <worktree-dir>`.

Linear tools come from the Linear MCP and are **deferred** — load them with
`ToolSearch` before calling (e.g. `get_issue`, `save_issue`, `list_comments`,
`list_issues`). If Linear returns an **authentication error**, stop and say:
*Linear isn't connected — run `/mcp`, authenticate the linear server, then try
again.*

---

## 1. Read the ticket

Fetch the issue with `get_issue` (pass `includeRelations: true` — blocked-by and
related tickets change what you build). Capture:

- **identifier** (e.g. `LEA-207`) and **URL**
- **title** and **description** — this is the contract you build to
- **gitBranchName** (e.g. `lea-207-rename-course-to-module`) — the single name
  used for branch and worktree
- current **state**, **estimate**, and **labels**

Also pull `list_comments` — comments routinely amend or narrow the description,
and the newest comment usually wins over the description when they conflict.
If the ticket has sub-issues, `list_issues` with `parentId` and treat the parent
as the umbrella: build only what this ticket's own description covers unless the
sub-issues are clearly meant to ship together.

**Spec gate.** This skill builds a written spec; it does not invent one.
Proceed only if the description defines what "done" looks like — acceptance
criteria, a scope list, or a description concrete enough that two people would
build the same thing.

**Stop** (do not worktree or PR) when:

- The description is empty, a one-liner, or a title restated. Say what's missing
  and offer **spec-run** to write the spec first.
- The ticket is genuinely a bug. Point at **fix-issue** — it's a tighter loop.
- Real product forks are unresolved (which surface, which behavior, which
  default). Ask 1–3 sharp questions with `AskUserQuestion` and wait. Only ask
  about forks that change what gets built — decide the rest yourself.

Let `NUM` = the issue number and `BRANCH` = the gitBranchName.

**Name the chat** as soon as you have the issue title (especially in a new
Claude Code / Agent chat). Rename the conversation in the sidebar to:

`<issue title> [LEA-XXX] <build summary>`

- Use the Linear **title** verbatim (or trimmed only if absurdly long).
- Put **`[LEA-XXX]`** immediately after the title.
- **Build summary**: fewer than 3 words — the thrust of the work.

Example: `Rename Course to Module [LEA-207] copy pass`

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

## 3. Build it

Implement **inside the worktree** — every edit, command, and check runs against
`/Users/ryanpey/Projects/learnpf-lea-<NUM>`, never the main repo.

1. **Read the relevant docs first.** The repo's `CLAUDE.md` maps areas to
   `docs/README_*.md` — read the ones your ticket touches before writing code.
   Feature work that ignores the house architecture gets rejected in review.
2. **Plan against the real code.** Explore the surfaces the ticket names,
   confirm they still exist, and form a short plan: files, order of work, and
   anything the ticket got wrong. If reality contradicts the spec, say so in a
   Linear comment (`save_comment`) and build the version that makes sense —
   don't silently redefine the scope.
3. **Implement to the acceptance criteria.** They are the definition of done.
   Build everything in scope; build nothing that's listed out of scope, however
   tempting. If part of the ticket turns out to be blocked, finish the rest and
   report exactly what you left out and why.
4. **Verify before push.** Build, lint, run the tests that cover what you
   touched, and drive the app in the preview when the change is visible — as
   far as the worktree allows. Do not push a red tree. For UI work, capture
   screenshots for the PR. The worktree has its own `node_modules` from step
   2, so tsc, jest, biome (changed paths only), and `next build` all run
   locally — run them; do not lean on CI for checks you can run here.

If you cannot build it without more product input, stop and report what you did,
what's blocking, and the specific decision you need — do not open a speculative
PR.

## 4. Push and open the PR

From the worktree, commit with a clear imperative message, push, and open the
PR with `gh` (base `staging`). **Always open it as a draft** (`--draft`) —
Ryan flips it to ready after his own pass; never `gh pr ready` it yourself:

```bash
cd /Users/ryanpey/Projects/learnpf-lea-<NUM>
git add -A
git commit -m "<imperative summary>

<short body>"
git push -u origin HEAD
gh pr create --draft --base staging --head <BRANCH> \
  --title "<issue title>" \
  --body "$(cat <<'EOF'
## Summary
<what changed and why, in two or three sentences>

<LEA-XXX>: <issue URL>

## Core decisions
- <each decision a reviewer needs to know, one line each>
EOF
)"
```

Keep the PR body to exactly those two sections. No acceptance-criteria
checklist, test plan, out-of-scope list, design notes, or setup steps: Ryan
reads the ticket for criteria and the diff for detail. A decision earns a bullet
only if a reviewer would otherwise ask "why this way?". Attach screenshots under
the summary for anything user-visible.

Move the issue to `In Review` with `save_issue` (`id: LEA-XXX`,
`state: In Review`) once the PR is open.

## 5. Report

Give the user, in a few lines: worktree path, branch, what you built, anything
you deliberately left out, and the PR URL.

---

## Rules

- **Specced tickets only** — if there's no spec, use spec-run; if it's a small
  bug, use fix-issue.
- Build the ticket that exists. Don't widen scope because the code nearby looks
  wrong — file a follow-up ticket instead and mention it in the report.
- Name every session chat: `<issue title> [LEA-XXX] <≤3-word build summary>`.
- One name throughout: the Linear `gitBranchName` is the branch and the
  worktree suffix. Don't invent variants.
- Never clobber an existing branch or worktree — reuse it.
- Don't touch the main repo's working tree.
- Always run `npm install` in the worktree after creating it. Never symlink or
  APFS-clone `node_modules` from the main repo.
- Do not open the IDE with `code` unless the user asks separately.
- Only create commits when implementing this workflow; follow the user's git
  safety rules (no force push to main/staging, no skipping hooks unless asked).
