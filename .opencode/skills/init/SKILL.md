---
name: init
description: Initialize an AGENTS.md file documenting the project for AI assistants
license: MIT
compatibility: opencode
metadata:
  tags: documentation, onboarding
---

## What I do

Create (or update) an `AGENTS.md` at the project root that gives any AI assistant — or a new developer on their first day — the context needed to work effectively in this codebase without reading every file.

## Steps

1. **Check if AGENTS.md already exists**:
   - If it exists: read it fully, then extend or correct it rather than replacing it
   - If it doesn't exist: create from scratch using the template below
2. **Explore the project structure**:
   - Root files: identify `package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, `*.csproj`
   - Directory layout: list top-level dirs and identify their roles
   - Entry points: find `main`, `index`, `app`, `server`, or framework bootstrap
3. **Read key files**:
   - Main config file
   - CI config: `.github/workflows/`, `Jenkinsfile`, `.gitlab-ci.yml`
   - Any existing `README.md` — extract facts, don't duplicate prose
4. **Run commands to discover the dev workflow**:
   - Try `npm run` / `make help` / `cargo --list` to find available commands
   - Identify: how to install deps, run locally, run tests, lint/format
5. **Draft AGENTS.md** — fill in only what you can verify; leave a `<!-- TODO -->` comment for anything uncertain
6. **Validate**: re-read the finished file — would a new developer or AI have everything they need to make a safe first change?

## AGENTS.md template

```markdown
# <Project name>

<1-2 sentences: what this project does and who uses it>

## Architecture

| Directory | Purpose |
|-----------|---------|
| `src/` | Application source code |
| `tests/` | Test files, mirroring src/ structure |
| `infra/` | Infrastructure-as-code (Terraform / CDK) |

## Development

### Setup
\`\`\`bash
npm install        # install dependencies
cp .env.example .env  # configure environment
npm run dev        # start dev server at http://localhost:3000
\`\`\`

### Testing
\`\`\`bash
npm test           # unit tests
npm run test:e2e   # end-to-end tests (requires running DB)
\`\`\`

### Lint & format
\`\`\`bash
npm run lint       # ESLint
npm run format     # Prettier
\`\`\`

## Conventions

- **Naming**: files are kebab-case (`user-service.ts`), classes are PascalCase
- **Imports**: use path aliases (`@/services/...`) not relative paths from `src/`
- **Error handling**: throw typed errors from `src/errors/`; never throw plain strings
- **Tests**: co-located with source in `__tests__/` directories; one file per module

## Important files

- `src/app.ts` — Express app setup and middleware registration
- `src/routes/index.ts` — all route registrations in one place
- `src/db/schema.ts` — database schema definitions
- `.env.example` — all required environment variables with descriptions

## AI assistant notes

- Always run `npm test` after changes to verify nothing broke
- The `src/generated/` directory is auto-generated — never edit it manually
- Database migrations live in `db/migrations/` — use the migrate skill before touching schema
```

## Handling an existing AGENTS.md

- **Outdated commands**: verify them by running the commands; update if they fail
- **Missing sections**: add them without removing anything correct
- **Wrong information**: correct it and note what changed in a comment or commit message
- **Bloated file**: trim prose but keep all technical facts; target <150 lines

## Rules

- Never invent information — only document what you can verify from the code or by running commands
- Keep it short enough that it's actually read (target <150 lines)
- Do not duplicate what's in README unless it's AI-specific guidance (e.g., "never edit generated files")
- Leave `<!-- TODO: verify this -->` markers rather than guessing at commands or conventions
