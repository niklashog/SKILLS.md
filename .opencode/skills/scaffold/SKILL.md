---
name: scaffold
description: Create boilerplate for a new feature or module that follows existing project conventions
license: MIT
compatibility: opencode
metadata:
  tags: codegen, workflow
---

## What I do

Generate the files and structure needed to start a new feature or module, derived from what already exists in the project — not from generic templates.

## Steps

1. **Understand the request** — what is being added? (REST route, service, React component, CLI command, DB model, background job, etc.)
2. **Find 1-2 existing examples** of the same type in the project:
   - Grep for similar file names: `find src/routes -name "*.ts"` or `glob "src/**/*controller*"`
   - Read the closest example in full — note structure, imports, naming, export style
3. **Map what needs to be created** based on the example:
   - Source file
   - Test file
   - Any wiring: route registration, DI container entry, barrel export, feature flag, config key
4. **Identify naming conventions** from existing code:
   - File names: `user.controller.ts`, `userController.ts`, `user_controller.py`
   - Class/function names: PascalCase, camelCase, snake_case
   - Test file: `user.test.ts`, `test_user.py`, `user_test.go`
5. **Generate files** that copy the pattern exactly — same import style, same export style, same structure
6. **Wire it up** — update every file that needs a reference to the new module
7. **For monorepos**: check if the new module belongs in an existing package or needs a new workspace entry
8. **Run the project** (or at least `tsc --noEmit` / `cargo check`) to confirm the scaffold compiles

## Framework-specific wiring examples

```
# Express route → must register in src/routes/index.ts
import { userRouter } from './user.router';
app.use('/users', userRouter);

# NestJS module → add to providers/controllers array in the parent module
@Module({ controllers: [UserController], providers: [UserService] })

# React component → export from src/components/index.ts barrel
export { UserCard } from './UserCard';

# Python FastAPI router → include in main.py
app.include_router(user_router, prefix="/users")

# Go handler → register in router setup
r.GET("/users/:id", handlers.GetUser)
```

## Rules

- Copy conventions from existing code — never impose a generic pattern if the project does it differently
- If conventions are inconsistent across the codebase, ask the user which pattern to follow before generating
- Do not add features or business logic beyond the skeleton — leave implementation for the next step
- Do not create files the user didn't ask for (no auto-generated READMEs, no extra config files)
- For monorepos: check workspace structure (`pnpm-workspace.yaml`, `nx.json`, `turbo.json`) before deciding where the file goes
- The scaffold is done when the project compiles with the new empty module in place
