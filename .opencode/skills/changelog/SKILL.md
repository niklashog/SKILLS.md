---
name: changelog
description: Generate a human-readable changelog from git history
license: MIT
compatibility: opencode
metadata:
  tags: git, documentation, release
---

## What I do

Produce a structured changelog from git commits between two refs (tags, branches, or commits), in Keep a Changelog format.

## Steps

1. Determine the range:
   - List recent tags: `git tag --sort=-creatordate | head -10`
   - If no tags exist, use the first commit: `git rev-list --max-parents=0 HEAD`
   - Default range: `<latest-tag>...HEAD`
   - Confirm with user if unclear
2. Fetch commits in range: `git log <from>..<to> --format="%H|||%s|||%b" --no-merges`
3. Filter out noise: skip automated commits matching patterns like `chore(deps): bump`, `Merge pull request`, `ci:`
4. Group commits by section:
   - **Added** — `feat` commits
   - **Fixed** — `fix` commits
   - **Changed** — `refactor`, `perf` commits
   - **Removed** — commits removing functionality (look for `remove`, `delete`, `drop` in subject)
   - **Security** — commits mentioning CVE, vulnerability, injection, XSS, auth
   - **Deprecated** — commits mentioning `deprecat`
5. Translate commit subjects into plain language — remove type prefixes, expand abbreviations
6. For **monorepos**: prefix each entry with the affected package: `**@myapp/api**: ...`
7. Check for existing `CHANGELOG.md`:
   - If it exists: prepend the new section above the previous entry
   - If it doesn't exist: create it with a `# Changelog` header and an `[Unreleased]` link block
8. Determine the version:
   - If releasing: use the new semver tag
   - If not yet released: use `[Unreleased]`

## Output format

```markdown
## [1.4.0] — 2026-04-16

### Added
- Users can now export their data as CSV from the account settings page

### Fixed
- Password reset emails were not sent when the username contained uppercase letters
- Dashboard failed to load when the user had zero completed tasks

### Changed
- Replaced custom date picker with native browser input for better accessibility

### Security
- Upgraded `jsonwebtoken` to 9.0.2 to address CVE-2022-23529

[1.4.0]: https://github.com/org/repo/compare/v1.3.0...v1.4.0
```

## Good vs bad entries

```
# Bad — raw commit message, jargon, no user context
- fix(auth): jwt expiry npe
- chore(deps): bump lodash from 4.17.20 to 4.17.21

# Good — plain language, user-facing framing
- Fixed a crash when logging in with an account that has no expiry date set
- (security) Patched lodash to address prototype pollution vulnerability (CVE-2021-23337)
```

## Rules

- Use today's date if generating for an unreleased version
- Never delete existing changelog entries
- Skip pure refactors, CI changes, and test-only commits from user-facing sections
- Dependency bumps belong in **Security** only if there's a CVE — otherwise omit
- Write in plain language a non-developer user can understand
- For monorepos without a root version, generate per-package changelogs
