---
name: web-search
description: Search the web to answer questions about libraries, APIs, error messages, or anything not in the codebase
license: MIT
compatibility: opencode
metadata:
  tags: research, web
---

## What I do

Search the web when the answer cannot be derived from the local codebase — library documentation, error messages, changelogs, format specifications, best practices, and current version information.

## When to use

- "How does X work in library Y?"
- "What does this error message mean?"
- "What is the current stable version of X?"
- "Does X support feature Y?"
- "What changed between version X and Y?"
- Anything requiring information beyond training data cutoff

## Steps

1. **Formulate a precise query** — be specific:
   - Include the library/framework name and version: `"express 5 migration res.json array"`
   - Include the error message verbatim (in quotes): `"Cannot read properties of undefined (reading 'map')" react`
   - Include the language/runtime when ambiguous: `"null pointer exception golang goroutine"`
   - Use `site:` to target authoritative sources: `site:docs.python.org pathlib glob`

2. **Evaluate source quality**:
   | Source type | Trustworthiness | Notes |
   |---|---|---|
   | Official docs (docs.*, *.dev, *.io/docs) | High | Prefer for API reference |
   | GitHub repo README / releases | High | Best for changelogs, migration guides |
   | GitHub issues / discussions | Medium | Good for bug workarounds |
   | Stack Overflow (accepted answer, high votes) | Medium | Good for error messages |
   | Personal blogs / Medium | Low | May be outdated or wrong |
   | AI-generated content (obvious paraphrasing) | Very low | Verify against primary source |

3. **Fetch the most relevant page** — extract the specific answer, don't summarize the whole page

4. **Check the date** — for rapidly-evolving libraries, check when the page was last updated; answers from 2+ years ago may be outdated

5. **Cite the source URL** alongside the answer

## Query crafting examples

```
# Bad — too vague
how to use jwt

# Good — specific library, specific question
"jsonwebtoken verify options expiry check node.js"

# Bad — no context
error ENOENT no such file or directory

# Good — error in quotes, with runtime
"ENOENT: no such file or directory, open" node.js fs.readFile

# Good — targeting official docs
site:react.dev useEffect cleanup function

# Good — finding changelog / migration
"prisma 5 migration guide" OR site:prisma.io/docs/guides/migrate
```

## Rules

- Prefer official documentation over blog posts when both are available for the same answer
- If top results conflict (e.g., the API changed between versions), note the discrepancy and cite both
- Do not invent URLs — only use URLs returned in actual search results
- If search yields nothing useful after 2-3 queries, say so rather than fabricating an answer
- Always note the version the answer applies to, especially for packages with major version differences
