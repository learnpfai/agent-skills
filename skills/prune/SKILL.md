---
name: prune
description: >-
  Clean up a git repo: remove all of its worktrees (never the base repo) and
  delete local branches whose work is backed up — tip on origin, or tip exactly
  matching a squash-merged PR head — keeping main, staging, and any branch with
  un-pushed local-only work. Use when the user says /prune, "prune this repo",
  "clean up my branches/worktrees", or asks to tidy up stale local branches.
---

# prune

Safe cleanup of a git repo. Three steps, no data loss.

## 1. Remove worktrees

- `git worktree list` — remove every worktree **except the base repo** with
  `git worktree remove <path>`, then `git worktree prune`.
- Check each is clean (`git status --porcelain`) first; if dirty, stop and ask.

## 2. Delete safe local branches

Delete a local branch **only if its tip commit is already reachable from the
remote** — meaning its work is backed up on origin. Never touch:

- `main` and `staging` (local or remote)
- any branch with local-only commits not on origin (un-pushed work)

The tip-on-remote check (not upstream tracking) is authoritative — a branch can
show `[ahead]`/`[gone]` yet still have its tip pushed to some other origin ref.

```bash
git fetch --prune
for b in $(git for-each-ref --format='%(refname:short)' refs/heads/); do
  case "$b" in main|staging) continue;; esac
  sha=$(git rev-parse "$b")
  if git branch -r --contains "$sha" | grep -q origin/; then
    echo "DELETE $b"        # tip is on origin → safe
  else
    echo "KEEP   $b"        # local-only commits → keep
  fi
done
```

Delete the DELETE set in one call: `git branch -D <b1> <b2> ...`. `-D` is safe
here because every tip was verified on origin.

## 3. Catch squash-merged branches

This repo squash-merges PRs, so a merged branch's original commits never appear
on origin — step 2 wrongly KEEPs them. Cross-check the KEEP set against merged
PRs on GitHub:

```bash
gh pr list --state merged --limit 300 --json headRefName,headRefOid,number \
  --jq '.[] | "\(.headRefName) \(.headRefOid) \(.number)"' > /tmp/merged_prs.txt
```

For each KEEP branch (skip the checked-out branch and any branch attached to a
worktree), look up its name in the merged-PR list and compare
`git rev-parse <branch>` to the PR's `headRefOid`:

- **Exact match** → the local tip is precisely what was squash-merged; the work
  is fully on the target branch. Safe to delete with `git branch -D`.
- **Mismatch or no merged PR** → the branch may hold local commits that never
  made it into the PR. Keep it and report it (with the PR number if one merged).

Only the exact-SHA match is safe. Do not delete on "a PR with this branch name
was merged" alone — the local tip can be behind or diverged from what merged.

## Always

- Print the DELETE / KEEP lists and get a quick confirm before deleting.
- Never delete `main` or `staging`; never remove the base repo worktree.
- The checked-out branch can't be deleted (git refuses) — leave it and note
  it's deletable once the user switches off it.
