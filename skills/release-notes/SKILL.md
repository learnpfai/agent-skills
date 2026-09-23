---
name: release-notes
description: >-
  Write the PR description for a staging → main release PR as a short, grouped
  bullet list named after the cycle (e.g. "Cycle 42.3"). Use when the user
  asks for a release PR description, release notes for a cycle, or "what's in
  this release", optionally giving a cycle number.
---

# Release Notes

Write the description for the release PR that merges `staging` into `main`.
The output is a plain bullet list under the cycle name. It is pasted into
the PR body and into Slack unchanged, so it uses plain-text bullets that
both render the same way. It is read by the whole company, so it leads with
what people will notice and hides implementation detail under the Docs,
Infra, CI, Tweaks and Bugs headings.

## Inputs

The user gives a cycle number, such as `42.3`. If none is given, ask for one.
If a PR number is given, use that PR's commit range instead of
`origin/main..origin/staging`.

## Workflow

### 1. Collect the commits and read the PRs

```bash
git fetch -q origin
git log --oneline origin/main..origin/staging
```

Every commit subject ending in `(#NNN)` and every `Merge pull request #NNN`
is a PR. Read all of them; the subject alone is not enough:

```bash
gh pr view <NNN> --json title,body -q '.title + "\n---\n" + .body'
```

The PR body is where the real change is described: a "tweak" PR often fixes
a bug, a "fix" PR often ships a feature, and a schema-only PR usually has no
visible effect. Group by what the body says, not by the title.

For commits with no PR, read the body and touched files:

```bash
git show --stat --format='%s%n%b' <sha>
```

Squash-merged PRs land as one commit. Feature branches merged with a merge
commit land as several commits; fold them into one bullet.

### 2. Group into bullets

Top-level bullets are changes someone in the company would notice: a
learner, a teacher, the content team, or an admin. Product features, internal
tools, admin pages, MCP tools, reports people run. Content Hub, Syllabus
Ordering, My Progress page, Invent Cheatsheet upload, Read Only MCP all
qualify. One bullet each, biggest change first. Name it as a product person
would say it: "My Progress page", not "progress route".

Everything else goes under one of five headings, as sub-bullets, in this
order:

- **Docs**: doc-only commits. Sub-bullet only if the doc is worth naming.
- **Infra**: nothing visible changes. Deploy, migrations with no visible
  effect, prompt plumbing, refactors, dead-code culls, table moves, scripts.
- **CI**: workflows, checks, auto review, test tooling, e2e setup.
- **Tweaks**: small visible changes not worth a top-level bullet.
- **Bugs**: fixes for things that were broken.

Rules:

- Slack-safe formatting only. Top-level bullets are `•`, sub-bullets are
  `◦` indented four spaces. No markdown list markers (`*`, `-`), no bold,
  no headers, no links. Backticks are the only markup, since they render in
  both GitHub and Slack.
- One line per bullet, sentence fragment, no full stops.
- No Linear ticket IDs anywhere. Strip `LEA-123` from commit subjects.
- No PR numbers, SHAs, file paths, or function names.
- If a commit reverts or supersedes another commit in the same range, list
  only the final state.
- If several commits build one feature, write one bullet for the feature.
- Skip merge commits, "resync", "fix test", and format-only commits.
- Backtick a table or column name only when it *is* the change
  (e.g. options moved to a `problem_options` table).
- Drop a heading that has nothing under it.

### 3. Output

Print the list in a fenced block so it can be copied as-is into the PR body
or a Slack message. No preamble and no notes after it unless a grouping call
is genuinely ambiguous, in which case add at most two one-line notes.

## Examples

```
Cycle 42.3

• Help Center refresh + learner What's New page (Modules, Progress Review, Revision, hints and AI chat; teacher updates coming soon)
• Revision gets Show hint / Talk it through on a wrong answer, same as Apply and Mastery
• Learning gain BQ reports (school-level and per-module student gain)
• Docs
    ◦ Post-ready review sequence
    ◦ Authors keep money amounts outside math
• Infra
    ◦ School admin flag on teacher users (column only, no UI yet)
    ◦ Shared math rules across every prompt (tutor, chatbot, checkers, MCP resource)
• CI
    ◦ Auto review now actually runs (its prompt never resolved, so every run was a silent no-op)
    ◦ Auto review labels renamed to `Review: Sonnet-medium` / `Sonnet-high` / `Opus-medium` / `Opus-high`
    ◦ PostHog surveys blocked in e2e browsers
• Tweaks
    ◦ Angles and Perpendicular & Parallel Lines lessons opened to MCP problem writes
    ◦ Syllabus tree drops module/lesson descriptions so titles stop truncating
• Bugs
    ◦ Tutor maths written as \(…\) rendered as raw code
    ◦ Money written inside math (`$\$10$`) broke rendering
    ◦ Revision charged every retry as helped, so unaided retries paid 1 XP instead of 2
```

```
Cycle 41.x

• Mastery Cross-module revision
• Content Hub merged into app
• Apply/mastery problem chats + hints
• My Progress page
• Learner Module Page (replace dashboard)
• Add Syllabus Ordering
• Infra
    ◦ Problem options → `problem_options` table
    ◦ Unified chatbot function
    ◦ Dead-code cull
    ◦ Lesson engine refactor
• CI
    ◦ Preview Branches
    ◦ Auto Review
• Tweaks
    ◦ Chatbot restyled purple → cream/amber
```

```
Cycle 40.2

• Content Checker Tweaks
    ◦ Disable Prompt Cache
    ◦ Open lesson editor directly to question
```

```
Cycle 39.4

• Upgraded Read Only MCP (List modules, lessons, LOs, AUs)
• Invent Cheatsheet UI + Admin Upload
• Allow Red Flags to be Resolved
• Tweaks
    ◦ Invent Submission Tweaks
• Bugs
    ◦ Fix problem order for mastery lesson
    ◦ Content Checker Bug Fixes
```
