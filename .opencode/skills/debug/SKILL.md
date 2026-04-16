---
name: debug
description: Systematically find and fix a bug — reproduce, isolate, identify root cause, fix
license: MIT
compatibility: opencode
metadata:
  tags: debugging, quality
---

## What I do

Work through a bug methodically: reproduce it reliably, narrow down where it happens, find the root cause, and fix it without breaking surrounding code.

## Steps

1. **Reproduce** — confirm the bug is reproducible with a specific input or sequence of steps. If it can't be reproduced, stop and ask for more information.
2. **Read the error** — full stack trace, not just the last line. Note the exact file, line, and error type.
3. **State expected vs actual** — write it down explicitly: *"Given X, I expect Y, but I get Z."* Do this before touching code.
4. **Isolate** — find the smallest input that triggers the bug. Remove variables until only the essential trigger remains.
5. **Trace** — follow the data flow from input to failure. Read each function in the call stack; don't assume.
6. **Hypothesize** — form one specific hypothesis: *"I think the bug is on line 42 because the null check happens after the property access."*
7. **Verify** — test the hypothesis by reading the code carefully, or add a temporary log/assertion. Do NOT change production logic to test a hypothesis.
8. **Fix** — change the root cause only. If the fix requires touching more than ~5 lines, reconsider whether you've found the real root cause.
9. **Confirm** — reproduce the original scenario and verify it no longer fails.
10. **Regression test** — write a test that would have caught this bug before it reached you.
11. **Clean up** — remove any temporary logs or debug assertions added during investigation.

## Language-specific debugging tools

| Language | Interactive debugger | Logging | Profiling |
|---|---|---|---|
| Node.js | `node --inspect` + Chrome DevTools, or `debugger;` with VSCode | `console.log`, `debug` package | `--prof` |
| Python | `python -m pdb script.py` or `breakpoint()` | `logging` module | `cProfile` |
| Go | `dlv debug` (Delve) | `log.Printf` | `pprof` |
| Rust | `rust-gdb` / `rust-lldb` or VSCode CodeLLDB | `dbg!()` macro | `cargo flamegraph` |
| Browser JS | Chrome DevTools → Sources → breakpoints | `console.log/warn/error` | Performance tab |

## Example: hypothesis/verify cycle

```
Bug: "Users can't log in after password reset"

Step 3 — Expected vs actual:
  Expected: POST /auth/login with new password → 200 OK
  Actual:   POST /auth/login with new password → 401 Unauthorized

Step 6 — Hypothesis:
  The password reset stores the hash correctly, but the login compare
  function is using the OLD hash cached in the session.

Step 7 — Verify (add temporary log, don't change logic):
  console.log('stored hash:', user.passwordHash);
  console.log('compare result:', await bcrypt.compare(password, user.passwordHash));
  → stored hash matches new password ✓
  → compare result: false ✗
  → bcrypt.compare is receiving the right hash but returning false

Step 6b — Revised hypothesis:
  bcrypt.compare is being called with arguments in wrong order:
  bcrypt.compare(hash, plaintext) instead of bcrypt.compare(plaintext, hash)

Step 8 — Fix: swap arguments. One line change.
```

## Rules

- Never change code to "see if it fixes it" without a hypothesis — that is guessing, not debugging
- Fix the root cause; do not paper over symptoms with try/catch or default values
- If the bug is in a dependency, work around it with a `// WORKAROUND: <issue URL>` comment explaining why
- If unable to reproduce after reasonable effort, say so and ask for more context rather than guessing
- Remove all temporary debug logs before committing the fix
