# SKILLS.md

A collection of agent skills for AI coding tools — organized by tool so each can grow independently.

## Structure

```
.opencode/          # Skills for OpenCode
  skills.md         # Quality overview of all skills
  skills/
    angular/        # Angular-specific additions (scaffold, test, debug, document)
    dotnet/         # .NET/C# additions (scaffold, test, document, migrate, deploy, ...)
    commit/         # Git commit conventions
    pr-create/      # Pull request creation
    review-pr/      # Code review
    test/           # Testing strategies
    ...
```

Future tool directories (`.cursor/`, `.claude/`, etc.) follow the same pattern.

## Skills (OpenCode)

| Category | Skills |
|---|---|
| Git | commit, pr-create, changelog |
| Review | review-pr, security-review |
| Quality | test, debug, simplify |
| Code gen | scaffold |
| Docs | document, init, explore |
| Maintenance | deps-update |
| Database | migrate |
| Performance | profile |
| DevOps | deploy |
| Research | web-search, research |
| Frontend | angular |
| Backend | dotnet |

See [`.opencode/skills.md`](.opencode/skills.md) for quality ratings and notes on each skill.
