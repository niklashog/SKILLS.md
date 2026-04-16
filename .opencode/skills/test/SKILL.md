---
name: test
description: Run tests, interpret failures, and write missing tests for new or changed code
license: MIT
compatibility: opencode
metadata:
  tags: testing, quality
---

## What I do

Run the test suite, diagnose failures, fix them, and ensure new code has adequate test coverage.

## Steps

### Running tests

1. Identify the test runner from manifests:
   - Node: `"scripts"` in `package.json` — look for `test`, `test:unit`, `test:e2e`
   - Python: `pytest` (check `pyproject.toml` or `setup.cfg`), `unittest`
   - Go: `go test ./...`
   - Rust: `cargo test`
2. Run the full suite first — note which tests fail and which pass
3. Re-run only failing tests to get clean, isolated output:
   - Jest: `npx jest --testPathPattern=auth`
   - pytest: `pytest tests/test_auth.py -v`
   - Go: `go test ./internal/auth/...`

### Diagnosing failures

1. Read the full error — stack trace, actual vs expected values, file and line
2. Identify the failure type:
   - **Assertion failure** — wrong output; trace the logic
   - **Exception/panic** — unexpected error; check null handling, missing setup
   - **Timeout** — async not resolved; check missing `await`, unresolved promises, deadlocks
   - **Flaky test** — passes sometimes; look for time-dependent logic, shared mutable state, missing test isolation
3. Trace back to what changed: `git diff HEAD~1 -- <file>`
4. Fix the root cause — not the assertion — unless the test was wrong to begin with

### Writing new tests

1. Read 2-3 existing tests in the same module to match naming, structure, and assertion style
2. Determine test type needed:
   - **Unit** — pure function, single module, no I/O; mock nothing unless crossing a system boundary
   - **Integration** — multiple modules working together; use real dependencies where feasible
   - **E2E** — full user flow through the system; use sparingly, only for critical paths
3. Cover these cases for each new function/behaviour:
   - Happy path — typical valid input
   - Boundary values — empty string, zero, max int, empty array
   - Invalid input — wrong type, null, negative number
   - Error conditions — what happens when a dependency fails?
4. Write the test name to describe the scenario: `it("returns 404 when user does not exist")`
5. Run the new tests to confirm they pass, then deliberately break the implementation to confirm they fail

## Good vs bad tests

```typescript
// Bad — tests nothing meaningful; passes even if the function is deleted
it('works', () => {
  expect(true).toBe(true);
});

// Bad — assertion too broad; passes for wrong reasons
it('returns a user', async () => {
  const user = await getUser(1);
  expect(user).toBeDefined();
});

// Good — specific assertion, named scenario, fails if implementation is wrong
it('returns 404 when user id does not exist', async () => {
  const res = await request(app).get('/users/99999');
  expect(res.status).toBe(404);
  expect(res.body.error).toBe('User not found');
});

// Good — boundary case
it('returns empty array when no orders exist for user', async () => {
  const orders = await getOrdersForUser(userId);
  expect(orders).toEqual([]);
});
```

## Rules

- Fix the code when it's wrong; fix the test when the test is wrong — never weaken an assertion to make a test pass
- Only mock at system boundaries: HTTP clients, filesystem, external databases — not internal modules
- Do not add tests for code that wasn't changed unless explicitly asked
- A test that always passes regardless of implementation is worse than no test
- Test names must describe the scenario, not the function: `"rejects login when password is wrong"` not `"test login"`
