---
name: explore
description: Map an unfamiliar codebase and explain how its parts connect
license: MIT
compatibility: opencode
metadata:
  tags: onboarding, documentation
---

## What I do

Build a mental model of an unfamiliar project by systematically exploring its structure, entry points, and key data flows — then explain it clearly with concrete file references.

## Steps

1. **Project type** — identify language, framework, and runtime from manifest files:
   - `package.json` → Node.js; check `"dependencies"` for Express/Fastify/NestJS/Next.js/React
   - `Cargo.toml` → Rust; check for `actix-web`, `axum`, `tokio`
   - `go.mod` → Go; check for `gin`, `echo`, `chi`
   - `pyproject.toml` / `setup.py` → Python; check for `fastapi`, `django`, `flask`
   - `*.csproj` → .NET; check `<TargetFramework>`

2. **Entry points** — find where execution starts:
   - Look for `main.ts`, `index.ts`, `app.ts`, `server.ts`, `main.go`, `main.py`, `Program.cs`
   - For frontend: look for `App.tsx`, `_app.tsx`, `layout.tsx` (Next.js)
   - Read the entry file fully — this reveals the app's top-level shape

3. **Directory map** — list top-level dirs and label each:
   ```
   src/
     routes/     HTTP route handlers
     services/   Business logic
     models/     Data access / ORM models
     middleware/ Express middleware
     utils/      Shared helpers
   tests/        Test files mirroring src/
   infra/        Terraform / CDK
   ```

4. **Key abstractions** — find types/interfaces that appear everywhere:
   - Grep for `interface`, `type`, `class`, `struct` in core directories
   - Look for what's imported in the most files: `grep -r "from.*types" src/ | sort | uniq -c | sort -rn | head -20`

5. **Data flow** — trace one representative operation end-to-end:
   - For APIs: pick one GET and one POST endpoint; trace from router → handler → service → DB → response
   - For CLIs: pick the main command; trace from arg parsing → logic → output
   - For workers: pick one job type; trace from queue consume → process → result

6. **For large codebases (>50k lines) or monorepos**:
   - Start at the `packages/` or `apps/` level — map each package's purpose before diving in
   - Identify which package owns the change you need to make before reading any code in detail
   - Check `nx.json`, `turbo.json`, or `pnpm-workspace.yaml` for the dependency graph between packages

## Output format

```
## Project type
Node.js 20 — Express 4 REST API with Prisma ORM and PostgreSQL

## Entry points
- `src/server.ts:1` — creates Express app, registers middleware, starts HTTP server
- `src/routes/index.ts:1` — registers all route groups

## Directory structure
src/
  routes/      one file per resource (users.ts, orders.ts, products.ts)
  services/    business logic called by route handlers
  prisma/      Prisma schema and generated client
  middleware/  auth.ts, rateLimit.ts, errorHandler.ts
tests/         mirrors src/, uses Jest + Supertest

## Key abstractions
- `src/types/index.ts` — AuthenticatedRequest (extends Express Request with req.user)
- `prisma/schema.prisma` — single source of truth for all DB models

## Data flow example — GET /users/:id
1. `src/routes/users.ts:34` — authenticate middleware validates JWT, sets req.user
2. `src/routes/users.ts:41` — handler calls UserService.findById(req.params.id)
3. `src/services/user.service.ts:18` — queries DB via Prisma: prisma.user.findUnique({ where: { id } })
4. Returns User object or null; handler returns 200 or 404

## External dependencies
- `prisma` — ORM and DB client (schema in prisma/schema.prisma)
- `jsonwebtoken` — JWT sign/verify in src/middleware/auth.ts
- `zod` — request validation in src/routes/*.ts
```

## Rules

- Cite `file:line` for every claim about where something lives
- Do not speculate about intent — only describe what the code actually does
- For monorepos: map the package graph before diving into any single package
- If a data flow crosses a package boundary (event bus, RPC, shared DB), name it explicitly
- Keep each section to 3-5 bullets; go deep on what matters, skip what's obvious from the directory name
