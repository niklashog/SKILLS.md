---
name: research
description: Deep research combining web search, documentation reading, and codebase exploration to answer complex technical questions
license: MIT
compatibility: opencode
metadata:
  tags: research, web, documentation
---

## What I do

Answer complex questions that require combining multiple sources: web searches, official docs, GitHub repos, and the local codebase. Used when a single web search is not enough to produce an actionable answer.

## When to use (vs web-search)

Use `research` when:
- The question requires combining information from multiple sources
- The answer must be validated against the local codebase
- You need to find an external format/schema and then generate files from it
- The question involves comparing approaches or making a recommendation

Use `web-search` when:
- A single specific fact needs to be looked up
- You're resolving a single error message
- You need the current version or release date of a package

## Steps

1. **Decompose** the question into independent sub-questions:
   - "How should I integrate Stripe webhooks into this Express app?" →
     - What does the Stripe webhook API require? (web search)
     - How is our Express middleware structured? (codebase)
     - Are there existing webhook handlers to follow? (codebase)

2. **Search** for each sub-question — use `web-search` skill techniques; prefer official sources

3. **Fetch** the most relevant pages; extract only what's needed to answer the sub-question

4. **Cross-reference with the local codebase**:
   - Does the documentation match what's already implemented?
   - Are there existing patterns in the codebase that should be followed?
   - Does the recommended approach conflict with anything already in place?

5. **Synthesize** — combine findings into a single actionable answer:
   - If sources agree: state the answer and cite sources
   - If sources conflict: surface the conflict with the reasoning for which to trust
   - If the codebase already does it differently: acknowledge the discrepancy and ask before overriding

6. **Apply** — if research was in service of a task, execute the task using the findings

## Handling conflicting sources

```
Example: "Should I use JWT or sessions for auth in this app?"

Source A (auth0.com/blog): recommends JWT for stateless APIs
Source B (owasp.org): notes JWT has significant footgun risk vs sessions
Source C (existing codebase): already uses session-based auth in src/middleware/auth.ts

Resolution:
- Sources A and B conflict on approach — both are credible but have different priorities
- The codebase already uses sessions → changing to JWT would be a large refactor
- Recommendation: stay with sessions unless there's a specific reason to change; cite the
  OWASP guidance for the session security requirements that must be met.
```

## When to stop researching

Stop when you have enough to act confidently. Signs you're over-researching:
- You're reading a 4th or 5th source that confirms what the first two said
- You're reading tangentially related content "just in case"
- You already know what to do but are looking for more confidence

When uncertain after thorough research: say what you found, state the uncertainty, and ask the user for direction rather than continuing to search.

## Output format

```
## Findings

### What does Stripe require for webhook verification?
Stripe sends a `Stripe-Signature` header. Verify it with `stripe.webhooks.constructEvent(body, sig, secret)`.
The raw request body must be passed — not a parsed JSON object.
Source: https://stripe.com/docs/webhooks/signatures

### How is our Express middleware structured?
Middleware lives in src/middleware/, registered in src/app.ts:34.
Existing example: src/middleware/auth.ts uses async middleware with next(error) for failures.

### Are there existing webhook handlers?
No existing webhook handlers found in the codebase.

## Conclusion
Add a new route at POST /webhooks/stripe. Use express.raw() for body parsing on that route only
(not the global express.json() parser). Verify signature in middleware before passing to handler.
Follow the middleware pattern in src/middleware/auth.ts.

## Action taken
Created src/routes/webhooks.ts and src/middleware/verifyStripeSignature.ts.
Registered in src/app.ts:41.
```

## Rules

- Always cite sources with the actual URL — never invent links
- Surface conflicting sources rather than silently picking one
- Verify format specs against a real example before generating files
- If the codebase already has an established pattern, default to it unless research shows a clear reason not to
- Stop researching when you have enough to act — present findings and proceed
