---
name: document
description: Write or update inline documentation, README sections, or API docs for changed code
license: MIT
compatibility: opencode
metadata:
  tags: documentation
---

## What I do

Add or update documentation — inline comments, docstrings, README sections, or API reference — scoped to code that was actually changed or explicitly requested.

## Steps

1. **Identify what needs documenting**:
   - `git diff HEAD~1` or the specified files
   - Focus on: public APIs, exported functions, non-obvious algorithms, configuration options, error conditions
2. **Read existing docs** in the same file to match the established style and format
3. **Determine doc type**:
   - **Inline comment** — for non-obvious *why* inside a function body
   - **Docstring / JSDoc / TSDoc** — for exported functions, classes, types
   - **README section** — for setup, usage, configuration, architecture
   - **API reference** — for HTTP endpoints or public SDK methods
4. **Write documentation** that answers: *why does this exist, when should I use it, what does it return, what can go wrong?*
5. **Remove outdated docs** for code that changed — stale documentation is worse than none

## Language-specific formats with examples

### TypeScript / JavaScript (JSDoc)
```typescript
/**
 * Retrieves a user by ID.
 *
 * Returns null if the user does not exist rather than throwing,
 * so callers must handle the null case explicitly.
 *
 * @param id - The user's UUID
 * @returns The user object, or null if not found
 * @throws {DatabaseError} If the database connection fails
 *
 * @example
 * const user = await getUserById('abc-123');
 * if (!user) return res.status(404).json({ error: 'Not found' });
 */
async function getUserById(id: string): Promise<User | null>
```

### Python (Google-style docstring)
```python
def get_user_by_id(user_id: str) -> Optional[User]:
    """Retrieve a user by their ID.

    Returns None if the user does not exist rather than raising,
    so callers must handle the None case explicitly.

    Args:
        user_id: The user's UUID string.

    Returns:
        The User object if found, None otherwise.

    Raises:
        DatabaseError: If the database connection fails.

    Example:
        user = get_user_by_id("abc-123")
        if user is None:
            raise HTTPException(status_code=404)
    """
```

### Go
```go
// GetUserByID retrieves a user by their UUID.
//
// Returns (nil, nil) when the user does not exist — callers must
// check for nil before using the result. Returns a non-nil error
// only for infrastructure failures (DB connection, timeout).
func GetUserByID(ctx context.Context, id string) (*User, error)
```

### Rust
```rust
/// Retrieves a user by their UUID.
///
/// Returns `None` if the user does not exist. Returns `Err` only
/// for infrastructure failures such as database connection errors.
///
/// # Examples
/// ```
/// let user = get_user_by_id(&pool, "abc-123").await?;
/// match user {
///     Some(u) => println!("Found: {}", u.name),
///     None => return Err(AppError::NotFound),
/// }
/// ```
pub async fn get_user_by_id(pool: &PgPool, id: &str) -> Result<Option<User>>
```

## Good vs bad comments

```typescript
// BAD — restates the code, adds no information
// increment counter
counter++;

// BAD — describes what, not why
// check if user is admin
if (user.role === 'admin') { ... }

// GOOD — explains a non-obvious decision
// Admin check must happen before quota check — admins are exempt
// from rate limits but the quota middleware runs first in the chain
if (user.role === 'admin') { ... }

// GOOD — explains why a seemingly wrong thing is intentional
// Intentionally not awaited — fire-and-forget analytics event,
// we don't want a tracking failure to block the response
trackEvent('login', { userId: user.id });
```

## Rules

- Only document code that changed or was explicitly requested — no drive-by additions
- Do not document obvious one-liners — a comment should add information the code cannot express
- Keep examples runnable — copy-paste them into the REPL to verify before committing
- Do not invent behaviour — read the implementation before writing what a function "does"
- Remove the old doc when you update the code — never leave docs that describe the old behaviour
