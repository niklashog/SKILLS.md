---
name: deps-update
description: Safely update project dependencies — check changelogs, apply updates, run tests, flag breaking changes
license: MIT
compatibility: opencode
metadata:
  tags: dependencies, maintenance
---

## What I do

Update dependencies in a controlled way: discover what's outdated, check changelogs, apply updates in safe batches, run tests, and surface anything that requires developer action.

## Steps

1. **Identify outdated packages**:
   - Node: `npm outdated` — shows current/wanted/latest columns
   - Node (more detail): `npx npm-check-updates` — shows all packages including transitive
   - Python: `pip list --outdated`
   - Rust: `cargo outdated` (requires `cargo install cargo-outdated`)
   - Go: `go list -u -m all`

2. **Categorize by semver** — determines how much caution to apply:
   - **Patch** (`1.2.3 → 1.2.4`) — bug/security fixes only; apply in bulk, run tests
   - **Minor** (`1.2.x → 1.3.0`) — new features, backward-compatible; scan changelog for deprecations
   - **Major** (`1.x → 2.0`) — may have breaking changes; read migration guide before updating

3. **Check changelogs** for minor and major updates:
   - Find changelog: GitHub releases page, `CHANGELOG.md` in repo, or `npm info <pkg> homepage`
   - Scan for keywords: `breaking`, `removed`, `renamed`, `no longer`, `deprecated`, `migration`
   - For major versions: look for an explicit migration guide

4. **Handle peer dependency conflicts** (Node):
   - `npm outdated` shows `MISSING` for peer deps — resolve these before updating
   - Check peer dep requirements: `npm info <pkg> peerDependencies`
   - For workspace/monorepo: update the dep in all relevant packages to the same version

5. **Apply updates in safe groups**:
   - Patches first: `npx ncu --target patch -u && npm install`
   - Minor next: one package or related group at a time
   - Major last: one package at a time, run tests after each

6. **Run tests after each group** — stop if tests fail, investigate before continuing

7. **Audit security**: run after all updates are applied
   - Node: `npm audit` — fix automatically with `npm audit fix` for patch-level fixes
   - Python: `pip-audit`
   - Rust: `cargo audit`
   - Flag any `high` or `critical` CVEs that remain

## Reading a changelog — example

```
# Checking express 4 → 5 migration

npm info express homepage
→ https://expressjs.com

Search GitHub releases for "express v5":
- res.json() no longer accepts non-objects — breaking for APIs returning arrays
- app.router is removed — middleware must use express.Router()
- req.param() is removed — use req.params, req.body, req.query explicitly

Action needed before updating:
- Audit all res.json() calls for array responses
- Grep for app.router and req.param(
```

## Output format

```
## Updated

### Patches (bulk)
- lodash 4.17.20 → 4.17.21
- axios 1.2.3 → 1.2.5

### Minor
- zod 3.21.0 → 3.23.8 — new .pipe() method added, no breaking changes

### Major
- express 4.18.2 → 5.0.1 — see breaking changes below

## Breaking changes to address
- express 5: `res.json()` rejects non-objects — found 3 array responses in src/routes/*.ts
- express 5: `req.param()` removed — found 1 usage in src/middleware/auth.ts:42

## Security issues
- None — npm audit clean after updates

## Skipped (requires manual review)
- react 18 → 19: major version, migration guide not yet reviewed
```

## Rules

- Never update all packages in one commit — group by patch / minor / major
- Do not update a package with unresolved breaking changes without flagging them to the user
- Always run the full test suite after updating — never assume it's safe
- Commit lock file changes (`package-lock.json`, `Cargo.lock`) alongside manifest changes in the same commit
- For workspaces/monorepos: update a shared dep to the same version across all packages
