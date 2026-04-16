---
name: pr-create
description: Create a pull request with a clear title, description, and test plan
license: MIT
compatibility: opencode
metadata:
  tags: git, workflow
---

## What I do

Create a well-structured pull request that gives reviewers everything they need to understand, test, and merge the changes.

## Steps

1. Run in parallel:
   - `git status` — confirm nothing unstaged that should be included
   - `git log main...HEAD --oneline` — list all commits going into the PR
   - `git diff main...HEAD --stat` — files changed and line counts
2. Check remote state: `git branch -vv` — is the branch pushed and up to date?
3. If behind base: `git rebase origin/main` before pushing
4. Push if needed: `git push -u origin <branch>`
5. Check for an existing PR: `gh pr view` — avoid duplicates
6. Draft the PR body (see format below)
7. Create: `gh pr create --title "..." --body "$(cat <<'EOF' ... EOF)"`
8. If it should not be merged yet: `gh pr create --draft`
9. Assign reviewers if known: `--reviewer username1,username2`
10. Link related issues: include `Closes #<number>` in the body

## Output format

```markdown
## Summary
- <what changed — be specific about files/modules affected>
- <why it was needed — the problem being solved or feature being added>

## Changes
- `src/auth/middleware.ts` — added JWT expiry check before route handler
- `tests/auth.test.ts` — added tests for expired and malformed tokens

## Test plan
- [ ] Existing tests pass: `npm test`
- [ ] Login with a valid token works
- [ ] Login with an expired token returns 401
- [ ] Login with a malformed token returns 400

## Notes
<anything reviewers should pay special attention to, or decisions made>

Closes #42
```

## Full example

```
gh pr create \
  --title "fix(auth): reject expired JWTs before reaching route handlers" \
  --body "$(cat <<'EOF'
## Summary
- Added expiry validation to the JWT middleware so expired tokens are
  rejected with a 401 before hitting any route handler
- Fixes a gap where expired tokens were silently treated as valid

## Changes
- \`src/auth/middleware.ts\` — added \`verifyExpiry()\` call in \`authenticate()\`
- \`tests/auth.test.ts\` — 3 new test cases for expired/malformed tokens

## Test plan
- [ ] \`npm test\` passes
- [ ] Expired token → 401 Unauthorized
- [ ] Valid token → request proceeds normally

Closes #412
EOF
)"
```

## Rules

- Never force push to main/master
- Never push without confirming with the user first
- Title must explain the change, not just reference a ticket number (`fix #42` is not a title)
- If the diff is >500 lines, add a **Changes** section listing key files and what changed in each
- If the branch is behind base, rebase — do not merge base into the branch
- Use `--draft` when the work is complete but not ready for review
