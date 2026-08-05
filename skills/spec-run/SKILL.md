---
name: spec-run
description: >-
  Two-phase "spec then run" workflow. Phase 1: turn a prompt into a concrete
  spec and file it as a Linear issue under the Planning status, assigned to you.
  Phase 2 (only after you approve): spin up a worktree off staging named after
  the issue, do the work, push, and open the PR — all under the same name. Use
  when the user says /spec-run, "spec this out", "spec and run", or describes a
  change they want specced into Linear and then built.
---

# spec-run

A guardrailed pipeline from a rough idea to an open PR, in two phases with a
human gate between them.

1. **Spec** — turn the user's prompt into a concrete spec, file it as a Linear
   issue under the **Planning** status, assigned to the user. **Stop and wait
   for approval.**
2. **Run** — only after the user approves, open a worktree off `staging` named
   after the issue, do the work, push, and open the PR — all under the **same
   name** (the issue's git branch name).

Never run Phase 2 without explicit approval. The Linear issue is the contract
the user signs off on before any code is written.

---

## Fixed facts for this workspace

- **Linear team**: `Learnpf` (key `LEA`).
- **Planning status**: the workflow state literally named `Planning`.
- **Assignee**: `me` (the current Linear user).
- **Main repo**: `/Users/ryanpey/Projects/learnpf`.
- **Base branch**: `staging` (branch off `origin/staging`, PR targets `staging`).
- **Worktree convention**: sibling dir `/Users/ryanpey/Projects/learnpf-lea-<NUM>`.
- **Copy-gitignored script**: `/Users/ryanpey/Projects/learnpf/ryan/copy-gitignored.sh <worktree-dir>`.

Linear tools come from the Linear MCP and are **deferred** — load them with
`ToolSearch` before calling (e.g. `save_issue`, `get_issue`, `get_user`,
`list_issue_statuses`). If Linear returns an **authentication error**, stop and
say: *Linear isn't connected — run `/mcp`, authenticate the linear server, then
try again.*

---

## Phase 1 — Spec

The user's prompt (the text after `/spec-run`, or the change they just
described) is the raw idea. Turn it into a spec, then file it.

### 1. Write the spec

Read the relevant code first when it's cheap — spec from facts, not guesses.
Keep it concrete and reviewable. The issue **description** (Markdown) should
have:

- **Problem / goal** — one or two sentences on what we're solving and why.
- **Approach** — the intended implementation in a few bullets (key files,
  functions, or components you expect to touch).
- **Scope** — what's in.
- **Out of scope** — what's explicitly not included, to prevent drift.
- **Acceptance criteria** — a short checklist of observable outcomes that mean
  "done".
- **Open questions** — only genuine forks you couldn't resolve from the code.

Derive a short, imperative **title** (this becomes the PR title too).

If the prompt is too vague to spec responsibly, ask 1–3 sharp questions with
`AskUserQuestion` before filing — only about forks that change what gets built.

### 2. File the Linear issue

Create it with `save_issue`:

- `team`: `Learnpf`
- `title`: the derived title
- `description`: the spec (literal newlines, real Markdown — no escaped `\n`)
- `state`: `Planning`
- `assignee`: `me`
- `priority`: infer from the ask if the user signalled urgency, else leave None.

Do **not** pass `id` (that's for updates). Capture the returned issue
**identifier** (e.g. `LEA-742`), **URL**, and **gitBranchName** — you need all
three for Phase 2.

### 3. Present and stop

Show the user, tightly:

- The issue: `LEA-XXX` (linked to its URL) · title.
- The spec's acceptance criteria and any open questions.
- The **exact** branch / worktree name Phase 2 will use (the gitBranchName).

Then ask for the go-ahead in plain words, e.g. *"Approve this spec and I'll open
the worktree and build it, or tell me what to change first."* **Do not proceed.**

If the user asks for edits, update the same issue with `save_issue` (pass its
`id`) and re-present. Only an explicit approval unlocks Phase 2.

---

## Phase 2 — Run (only after approval)

Everything here keys off the **same name**: the issue's `gitBranchName`. Branch,
worktree, and PR all share it.

Let `NUM` = the issue number (e.g. `742`) and `BRANCH` = the gitBranchName
(e.g. `lea-742-add-streak-badge`).

### 1. Create the branch + worktree off staging

From the **main repo** (`/Users/ryanpey/Projects/learnpf`):

```bash
git fetch origin
git worktree add -b <BRANCH> /Users/ryanpey/Projects/learnpf-lea-<NUM> origin/staging
```

**Idempotent** — never clobber:
- Branch already exists → `git worktree add <worktree-dir> <BRANCH>` (no `-b`).
- Worktree dir already exists → reuse it; just point the user at the path.

### 2. Make it runnable

From the main repo, copy gitignored files (env, local config, private `ryan/`;
skips `node_modules`, `.next`, build artifacts):

```bash
/Users/ryanpey/Projects/learnpf/ryan/copy-gitignored.sh /Users/ryanpey/Projects/learnpf-lea-<NUM>
```

Then install deps in the worktree:

```bash
(cd /Users/ryanpey/Projects/learnpf-lea-<NUM> && npm install)
```

### 3. Do the work

Implement the spec **inside the worktree** — every edit, command, and check runs
against `/Users/ryanpey/Projects/learnpf-lea-<NUM>`, not the main repo. Follow
the acceptance criteria as the definition of done. Verify your changes (build /
lint / relevant tests / preview) before pushing — don't push a red tree.

### 4. Push and open the PR

From the worktree:

```bash
cd /Users/ryanpey/Projects/learnpf-lea-<NUM>
git add -A
git commit -m "<imperative summary>

<short body>

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
git push -u origin <BRANCH>
gh pr create --base staging --head <BRANCH> \
  --title "<issue title>" \
  --body "<PR body>"
```

PR body should:
- Summarize what changed and why.
- Link the Linear issue (`LEA-XXX` + URL) — Linear auto-links via the branch
  name, but include it explicitly too.
- Restate the acceptance criteria as a checklist.

### 5. Report

Give the user, in a few lines: the worktree path, the pushed branch, and the PR
URL. Optionally move the Linear issue to `In Review` with `save_issue`
(`id: LEA-XXX`, `state: In Review`) now that a PR is open.

---

## Rules

- The two phases are separated by a hard human gate. Phase 1 files a spec and
  stops; only explicit approval starts Phase 2.
- One name throughout: the Linear `gitBranchName` is the branch, the worktree
  suffix, and the PR head. Don't invent variants.
- Confirm the worktree path once before creating it (real filesystem side
  effect), then run the steps.
- Never clobber an existing branch or worktree — reuse it.
- Only commit and push as part of an approved Phase 2. Don't touch the main
  repo's working tree.
