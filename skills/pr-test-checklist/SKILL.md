---
name: pr-test-checklist
description: >-
  Review a GitHub pull request and produce a simple, risk-scored, role-based
  manual tester checklist in plain English. Use when the user gives a PR number
  and asks what to test, how to QA it, how to verify it, or wants a manual test
  checklist / QA checklist for a pull request.
---

# PR Test Checklist

Turn a pull request into a plain-English checklist a manual tester can follow in
the app. The checklist should be risk-scored, organized around user roles, and
written like user stories:

`As a learner, I should be able to ...`

Keep the final output non-technical. The tester should not need to understand
the codebase, file names, API routes, database tables, or implementation
details. Still use the diff to think deeply about edge cases before writing the
checklist.

## Inputs

The user gives a PR number, such as `612` or `#612`. If no number is given, ask
for one.

## Workflow

### 1. Understand the PR

Use the PR title, description, linked issue, changed files, and diff to
understand what behavior changed.

```bash
gh pr view <number>
gh pr diff <number>
gh pr view <number> --json files,additions,deletions,baseRefName
```

If the PR links an issue, read it too. The issue often explains the expected
user behavior more clearly than the code.

### 2. Identify the tester roles

Decide which roles are relevant to this PR. Common roles include:

- learner
- teacher
- admin
- unauthenticated visitor
- parent
- school manager

Only include roles that matter for the PR. Do not add empty role sections.

### 3. Write manual test items

Each checklist item should describe what the tester does and what should happen.
Use this shape:

`- [ ] As a <role>, I should be able to <action> → <expected result>`

Good checklist items:

- `As a learner, I should be able to open my lesson and see the new progress message → the message appears in the right place and does not block the lesson.`
- `As a teacher, I should be able to invite a learner with a valid email → the learner receives one invite and appears in my class list.`
- `As an admin, I should not be able to save the form with a missing required field → I see a clear error and no incomplete record is created.`

Avoid technical checklist items:

- Don't say "verify API response."
- Don't say "check database row."
- Don't say "test route handler."
- Don't mention file paths, function names, tables, caches, or internal services
  unless the user specifically asks for technical detail.

### 4. Look for edge cases

Before writing the final checklist, look for edge cases this PR could break.
Only include edge cases that a manual tester can realistically try in the app.

Consider:

- empty state
- first-time user
- invalid input
- very long input
- duplicate records
- duplicate click or retry
- back button or page refresh
- switching roles
- permission boundaries
- using an existing account with old data
- many items at once
- partial success, such as one invite failing while others succeed
- mobile or small screen, if the PR changes the UI

Write edge cases in the same role-based style:

`As a teacher, I should be able to submit a list where one email is invalid → valid learners are still invited and the invalid email is clearly shown.`

Do not invent a long list. Include only edge cases that are relevant to the PR.

### 5. Prioritize by risk

Keep risk scoring in the final checklist. Rank checks by:

`risk = how likely this is to break × how bad it would be if it broke`

Use these priority sections:

- **P0 — Must pass before merge:** core flow, data loss, privacy, payments, or anything that blocks real users.
- **P1 — High priority:** common flows or important edge cases with a workaround.
- **P2 — Medium priority:** less common flows, secondary roles, or non-blocking problems.
- **P3 — If time permits:** cosmetic, rare, or nice-to-check behavior.

### 6. Write the checklist to a file

Save the checklist to:

`pr-test-checklists/pr-<number>-checklist.md`

Create the `pr-test-checklists/` directory if it does not exist. If the file
already exists, overwrite it with the latest checklist.

After writing the file, tell the user the path and show the same checklist in
the chat.

## Output Format

Produce exactly this structure:

```markdown
# Manual Test Checklist — PR #<number>: <title>

## What changed
<1-3 plain-English sentences about what a tester should expect to be different.>

## P0 — Must pass before merge
- [ ] As a <role>, I should be able to <action> → <expected result>
- [ ] As a <role>, I should not be able to <blocked action> → <expected result>

## P1 — High priority
- [ ] As a <role>, I should be able to <action> → <expected result>

## P2 — Medium priority
- [ ] As a <role>, I should be able to <action> → <expected result>

## P3 — If time permits
- [ ] As a <role>, I should be able to <action> → <expected result>
```

Rules:

- Keep the P0/P1/P2/P3 sections, but skip empty ones.
- Start every role-based item with `As a ...`.
- Include role names like `learner`, `teacher`, `admin`, or `visitor` inside
  the checklist item.
- Include relevant edge cases, but keep them role-based and user-facing.
- Keep each item short enough for a tester to scan quickly.
- Use plain product language, not code language.
- Include expected results with `→`.
- For very small PRs, write only the checks that matter.

## Example

```markdown
# Manual Test Checklist — PR #587: Bulk learner invites

## What changed
Teachers can invite multiple learners at once. The main thing to confirm is that
each learner gets exactly one invite and the teacher can clearly see any failed
emails.

## P0 — Must pass before merge
- [ ] As a teacher, I should be able to invite three learners with valid emails → all three learners receive one invite each.
- [ ] As a teacher, I should be able to see which invite failed when one email is invalid → valid learners are still invited and the invalid email is clearly shown.
- [ ] As a learner, I should be able to open my invite link → I land on the expected signup or class join page.

## P1 — High priority
- [ ] As a teacher, I should not be able to accidentally invite the same learner twice in one submission → the learner receives only one invite.
- [ ] As a teacher, I should be able to double-click the invite button by mistake → learners are not invited twice.

## P2 — Medium priority
- [ ] As a teacher, I should be able to submit an empty invite list → I see a clear message explaining what to fix.
```
