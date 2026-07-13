---
name: prune
description: >-
  Clean up a git repo: remove all of its worktrees (never the base repo) and
  delete local branches whose tip commit is already on the remote — keeping
  main, staging, and any branch with un-pushed local-only work. Use when the
  user says /prune, "prune this repo", "clean up my branches/worktrees", or
  asks to tidy up stale local branches.
---

# prune

Safe cleanup of a git repo. Two steps, no data loss.

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

## Always

- Print the DELETE / KEEP lists and get a quick confirm before deleting.
- Never delete `main` or `staging`; never remove the base repo worktree.
