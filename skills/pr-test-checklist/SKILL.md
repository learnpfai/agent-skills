---
name: pr-test-checklist
description: >-
  Review a GitHub pull request and produce a prioritized manual-QA test
  checklist — concrete click-through steps a human can run in the app, ranked
  highest to lowest priority, with special attention to edge cases the diff is
  likely to break. Use this whenever the user gives you a PR number and asks
  what to test, how to QA it, what could break, or wants a test plan / test
  checklist / QA checklist for a pull request — e.g. "what should I test on
  PR 612", "give me a QA checklist for #587", "how do I verify this PR",
  "what edge cases does this PR risk". Also use it before merging when the user
  wants confidence a change won't regress.
---

# PR Test Checklist

Turn a pull request into a prioritized list of things a human should manually
test, ranked so the tester spends their limited time on what's most likely to
break and would hurt the most if it did.

The value of this skill is **not** in listing the obvious happy path — anyone
can see that. It's in reading the diff carefully enough to surface the edge
cases the author probably didn't think about, and in ordering them so the first
five items catch 80% of the real risk.

## Inputs

The user gives you a **PR number** (e.g. `612`, `#612`). Everything is fetched
via the `gh` CLI. If no number is given, ask for one — don't guess from the
current branch.

## Workflow

### 1. Fetch the PR

Run these to load the full picture. Don't skim — the diff is where the edge
cases hide.

```bash
gh pr view <number>                          # title, body, linked issues, author
gh pr diff <number>                          # the actual code change
gh pr view <number> --json files,additions,deletions,baseRefName
```

Read the PR body for the author's stated intent and any "how to test" notes
they left. If the body links an issue, `gh issue view <n>` it — the issue tells
you what the *user-visible behavior* is supposed to be, which is what QA
actually verifies.

### 2. Understand the change, then read around it

For each meaningfully-changed file, open it in the working tree (not just the
diff hunk) so you understand the surrounding code the change sits in. A diff
hunk shows *what* changed; the file shows what it can *break*. Pay attention to:

- **What triggers this code** — a button click, a cron job, an API route, a
  webhook? That's the entry point a tester needs to reach.
- **What it touches on the way out** — the database, an email send, a payment,
  another user's data, a cache. That's the blast radius.
- **What it shares with other features** — a modified helper, a changed shared
  component, a new column. Changes here ripple beyond the PR's stated scope.

Consult the repo's own docs when the change lands in a documented subsystem —
`CLAUDE.md` points to `docs/README_*.md` files (charts, chatbot, gamification,
LLM generation, security/validation, server actions, teacher features, RPCs,
etc.). They tell you the invariants the feature is supposed to hold, which is
exactly what you're testing for.

### 3. Hunt edge cases systematically

This is the core of the skill. Don't freeform it — walk these lenses against
the change and keep the ones that actually apply. Most PRs light up 4–6 of
these; naming them explicitly is how you catch what the author missed.

- **Boundaries** — 0, 1, the max, one past the max. Empty string, empty list,
  a single item, a huge batch. Off-by-one in pagination, limits, quotas.
- **Empty / missing / null** — no data yet, a field the user never filled,
  a first-run state, an account with nothing in it.
- **Volume & performance** — the operation that's fine for 5 rows and melts at
  5,000. Bulk actions, loops over user input, N+1 queries, anything that fans
  out to emails or external calls.
- **Permissions & roles** — does it behave right for each role (student,
  teacher, admin, unauthenticated)? Can a user reach data that isn't theirs?
- **Failure & partial failure** — the external call times out, the email
  half-sends, the upload is corrupt, the network drops mid-action. What state
  is the user left in?
- **State & sequence** — double-click / double-submit, back button, refresh
  mid-flow, doing steps out of order, retrying an action that already
  succeeded. Idempotency.
- **Concurrency** — two users (or two tabs) hitting the same record at once.
- **Backwards compatibility** — existing rows created before this change, users
  mid-session on the old version, saved state in a now-changed format.
- **Cross-feature interaction** — the shared component / helper / column this
  PR changed is used somewhere else that the PR doesn't mention. Name that
  somewhere else as a thing to check.
- **Input validation** — the paste of 10,000 chars, emoji, RTL text, a
  duplicate, a value that's valid-looking but wrong (past date, negative
  number, malformed email).

For each surviving edge case, write it as a **concrete action a human can
perform in the running app**, not an abstract worry. "Invite 200 students in
one batch and confirm every one receives an email and none are duplicated"
beats "test bulk invite at scale."

### 4. Prioritize

Rank every item by **risk = likelihood it breaks × how much it hurts if it
does**. A cosmetic glitch on an obscure path is P3; a data-loss or
wrong-recipient bug on the main flow is P0. Concretely:

- **P0 — Blocker.** Test before merge. The core happy path of the PR, plus any
  edge case whose failure means data loss, wrong-user data exposure, money, or
  a broken primary flow. If P0 fails, the PR isn't shippable.
- **P1 — High.** Likely-hit paths and edge cases with real user impact but a
  workaround or limited blast radius.
- **P2 — Medium.** Less common paths, degraded-but-not-broken states, secondary
  roles.
- **P3 — Low.** Cosmetic, rare, or defense-in-depth checks worth a glance if
  there's time.

Order matters *within* the whole list too: a tester reading top-to-bottom
should hit the scariest thing first.

### 5. Write the checklist to a file

After you've composed the checklist, save it to a Markdown file so the tester
can open, edit, and check items off outside the chat. Do this every run — it's
not optional.

- **Path:** `pr-test-checklists/pr-<number>-checklist.md` in the repo root.
  Create the `pr-test-checklists/` directory if it doesn't exist.
- **Filename** uses the PR number, e.g. `pr-587-checklist.md`. If a file for
  that PR already exists, overwrite it (a re-run reflects the latest diff).
- Write the **exact same Markdown** you produce in the Output format below —
  the file is the source of truth, not a summary of it.
- After writing, **tell the user the path** you saved it to and show the
  checklist in the chat as well, so they see it inline without opening the file.

## Output format

Produce exactly this structure, then **write it to a Markdown file** (see step 5
below). Keep it scannable — this is a working document someone checks off, not a
report.

```markdown
# Test Checklist — PR #<number>: <title>

## Risk summary
<3–6 sentences, grounded in the actual diff: what this PR changes, the 1–3
things most likely to break, and why. Name the specific files/areas so the
tester knows where a failure points. No generic filler.>

## P0 — Test before merge
- [ ] <concrete action> → <expected result>
- [ ] ...

## P1 — High priority
- [ ] <concrete action> → <expected result>

## P2 — Medium priority
- [ ] ...

## P3 — Low priority / if time permits
- [ ] ...
```

Rules for the checklist items:

- Every item is a **checkbox** with an action and its **expected result**
  (`→ what should happen`). Without the expected result the tester can't tell
  pass from fail.
- Be specific to *this* PR. If an item could be copy-pasted onto any random PR,
  it's too generic — cut it or sharpen it.
- Skip empty priority sections. A small PR might only have P0 and P1 — that's
  fine, don't pad it.
- If the diff is trivial (a copy tweak, a version bump), say so plainly and give
  the two or three checks that actually matter rather than inventing edge cases
  to fill a template.

## Example (abridged)

For a PR that changes bulk student-invite email sending:

```markdown
# Test Checklist — PR #587: Bulk invite email batching

## Risk summary
This PR reworks how `bulk-invite` fans out emails (src/app/api/bulk-invite/
route.ts and supabase/functions/bulk-invite/lib/email.ts), moving from
per-invite sends to batched sends. The highest risk is partial-batch failure —
if one send in a batch throws, the rest of the batch may be silently dropped,
leaving some students never invited with no error surfaced to the teacher.
Duplicate sends on retry are the second concern.

## P0 — Test before merge
- [ ] Invite 3 students, confirm all 3 receive exactly one email each → all delivered, no dupes
- [ ] Invite a batch where one address is invalid (e.g. `foo@`) → valid students still get emails, teacher sees which one failed
- [ ] Invite the same student twice in one submission → only one email sent

## P1 — High priority
- [ ] Invite 200 students at once → all delivered within reasonable time, no timeout, no rate-limit drop
- [ ] Submit the invite form, then double-click submit → students not emailed twice

## P2 — Medium priority
- [ ] Invite with a student whose name contains emoji / RTL characters → email renders correctly
```

Notice the items name real failure modes from *this* change (partial-batch
drop, dupes on retry) rather than generic "test the emails" filler.
