---
name: review-pr
description: Review a pull request for correctness, security, and code quality
license: MIT
compatibility: opencode
metadata:
  tags: git, review, quality
---

## What I do

Provide a structured, actionable code review of a pull request — covering correctness, security, edge cases, tests, and complexity.

## Steps

1. Fetch PR details: `gh pr view <number> --json title,body,author,files,additions,deletions`
2. Read the diff: `gh pr diff <number>`
3. For large PRs (>300 lines): read changed files individually to get full context around each change
4. For each changed file, check:
   - **Correctness** — does the logic do what the PR description claims? Are there off-by-one errors, wrong conditions, inverted booleans?
   - **Security** — injection, auth bypasses, exposed secrets, missing ownership checks (see `security-review` skill for full checklist)
   - **Edge cases** — null/undefined, empty collections, zero values, concurrent access, network failure
   - **Tests** — are new code paths covered? Do the tests actually assert the right thing, or do they just not throw?
   - **Complexity** — can this be achieved more simply? Is there an abstraction that obscures what 5 plain lines would make obvious?
5. Group findings by severity and output structured review

## Language-specific things to check

- **TypeScript/JS**: `any` types that bypass safety; missing `await` on async calls; mutations of function arguments
- **Python**: mutable default arguments (`def f(x=[])`); catching bare `except:`; missing `with` for resource management
- **Go**: ignored errors (`_ =`); goroutine leaks; missing context propagation
- **SQL**: queries built with string concatenation; missing indexes on JOIN/WHERE columns; SELECT * in production code

## Output format

```
## Summary
<1-3 sentence overall assessment — is this ready to merge, needs minor work, or has blockers?>

## Blockers
- `src/auth/middleware.ts:42` — JWT signature is verified but expiry is never checked.
  An attacker can use an expired token indefinitely. Add `{ ignoreExpiration: false }` to verify options.

## Suggestions
- `src/users/repository.ts:99` — `getUserById` fetches all columns including `password_hash`.
  Select only the columns the caller needs.

## Nits
- `src/utils/format.ts:12` — variable named `data` — rename to `formattedDate` to match what it holds
```

## Example: real blocker vs non-blocker

```
# Blocker — exploitable, must fix before merge
src/api/orders.ts:88 — order is fetched by ID without checking req.user.id === order.userId.
Any authenticated user can read any order by guessing the ID.

# Suggestion — improvement, not required
src/api/orders.ts:102 — N+1 query: getLineItems() is called inside a loop.
Batch with getLineItemsByOrderIds([...orderIds]) instead.

# Nit — style only
src/api/orders.ts:45 — `isValid` is a double negative when used as `if (!isValid)`.
Consider `isInvalid` or restructure the condition.
```

## Rules

- Blockers must be fixed before merge; suggestions and nits are optional
- Never approve a PR with an unaddressed security issue
- Be specific — cite file and line number, state what is wrong, and explain why it matters
- Do not block on style issues that a linter should enforce — note them as nits or skip entirely
- If the PR description is missing or misleading, flag that too
