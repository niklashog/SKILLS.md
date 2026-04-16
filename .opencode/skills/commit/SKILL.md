---
name: commit
description: Create a well-structured git commit with a conventional commit message
license: MIT
compatibility: opencode
metadata:
  tags: git, workflow
---

## What I do

Stage and commit changes with a conventional commit message that explains *why*, not just *what*.

## Steps

1. Run `git status` and `git diff` in parallel to understand current changes
2. Run `git log --oneline -10` to match the repo's existing commit style
3. Identify the commit type:
   - `feat` — new feature visible to users
   - `fix` — bug fix
   - `refactor` — code change that neither fixes a bug nor adds a feature
   - `test` — adding or updating tests
   - `docs` — documentation only
   - `chore` — build process, tooling, deps
   - `perf` — performance improvement
   - `ci` — CI/CD config changes
4. Identify the scope (optional) — the module, package, or area affected: `feat(auth)`, `fix(api)`, `chore(deps)`
5. Stage specific files by name — never `git add -A` or `git add .`
6. Write the subject line (≤72 chars): `type(scope): short description in imperative mood`
7. Add a body paragraph when the *why* isn't obvious from the diff — skip it when the subject line is self-sufficient
8. Commit using a HEREDOC to preserve formatting

## Good vs bad subject lines

```
# Bad — describes what, not why; past tense; too vague
fixed bug
updated auth
changed the way tokens are handled in the middleware

# Good — imperative mood, scoped, explains the change
fix(auth): reject expired JWTs before reaching route handlers
feat(billing): add proration when upgrading mid-cycle
chore(deps): upgrade eslint to v9 for flat config support
refactor(db): replace raw SQL in UserRepository with query builder
```

## Body example

```
fix(queue): prevent duplicate job processing on worker restart

Workers were not checking for an in-progress lock before claiming
a job, causing two workers to occasionally process the same item
when one restarted mid-job. Added a Redis SETNX lock with TTL.

Fixes #412
```

## Rules

- Never skip hooks (`--no-verify`)
- Never amend unless explicitly asked — always create a new commit
- Never commit `.env`, credentials, or generated build artifacts
- Never push unless explicitly asked
- If multiple unrelated changes are staged, split into separate commits
- Breaking changes must include a `BREAKING CHANGE:` footer in the body
