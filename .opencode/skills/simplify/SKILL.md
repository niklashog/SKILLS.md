---
name: simplify
description: Review recently changed code for unnecessary complexity and clean it up
license: MIT
compatibility: opencode
metadata:
  tags: refactor, quality
---

## What I do

Audit the diff of recent changes (or a specified file/function) for complexity, redundancy, and poor reuse — then fix what I find.

## Steps

1. Get the scope: `git diff HEAD~1` or read the specified file
2. Identify issues in this order of priority:
   - **Dead code** — variables assigned but never read, imports never used, branches that can never be reached
   - **Duplication** — identical or near-identical logic repeated in 3+ places
   - **Premature abstraction** — a helper, wrapper, or interface that exists for only one callsite
   - **Overly defensive code** — null checks, try/catch, or fallbacks guarding against states that are impossible given the surrounding code
   - **Naming** — a variable or function whose name does not match what it actually contains or does
3. Apply fixes with Edit — one issue at a time, minimal changes
4. Do NOT refactor code outside the stated scope

## Good vs bad examples

```typescript
// BAD — premature abstraction, used only once, obscures a one-liner
function formatUserDisplayName(user: User): string {
  return `${user.firstName} ${user.lastName}`;
}
// Called exactly once: formatUserDisplayName(user)

// GOOD — just inline it
const displayName = `${user.firstName} ${user.lastName}`;

---

// BAD — defensive code for an impossible state
// (userId is typed string, already validated by middleware — can't be null here)
const userId = req.user?.id ?? 'unknown';
if (!userId || userId === 'unknown') {
  return res.status(401).json({ error: 'Unauthorized' });
}

// GOOD — trust the type system and middleware
const userId = req.user.id;

---

// BAD — duplication across three route handlers
router.get('/users', async (req, res) => {
  try { ... } catch (e) { logger.error(e); res.status(500).json({ error: 'Internal error' }); }
});
router.post('/users', async (req, res) => {
  try { ... } catch (e) { logger.error(e); res.status(500).json({ error: 'Internal error' }); }
});
router.delete('/users/:id', async (req, res) => {
  try { ... } catch (e) { logger.error(e); res.status(500).json({ error: 'Internal error' }); }
});

// GOOD — extract when used 3+ times
function asyncHandler(fn) {
  return (req, res) => fn(req, res).catch(e => {
    logger.error(e);
    res.status(500).json({ error: 'Internal error' });
  });
}
router.get('/users', asyncHandler(async (req, res) => { ... }));
```

## Rules

- Fix only what was changed or explicitly asked about — do not wander into surrounding code
- Remove unused code completely; do not comment it out or leave `// unused` markers
- Do not add docstrings, type annotations, or comments to code that wasn't changed
- Do not add features, make things "configurable", or design for hypothetical future use
- Three similar lines is fine; extract a helper only when there are 3+ actual usages
- A simplification that makes the code harder for a new reader to understand is not a simplification
