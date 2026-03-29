# Code Review Guidelines

## Principles

- **Be kind, not clever.** Reviews are a conversation, not a competition. Assume positive intent.
- **Review the code, not the author.** Feedback should address the change, never the person.
- **Smaller is faster.** Prefer pull requests under 400 lines. Large diffs obscure bugs and slow reviews.
- **Context first.** Read the PR description and linked issue/ticket *before* reading the diff.

## Reviewer checklist

### Correctness
- [ ] Does the code do what the description claims?
- [ ] Are edge cases and error conditions handled?
- [ ] Are there adequate tests covering happy paths, edge cases, and error paths?
- [ ] Do existing tests still pass?

### Design
- [ ] Does the change fit the existing architecture and patterns?
- [ ] Is the solution appropriately simple? (avoid over-engineering)
- [ ] Are responsibilities clearly separated?
- [ ] Are interfaces/abstractions stable and reusable?

### Security
- [ ] Is user input validated and sanitised?
- [ ] Are secrets, tokens, or PII handled correctly (not logged, not committed)?
- [ ] Are new dependencies vetted for known vulnerabilities?
- [ ] Are authorisation checks applied where needed?

### Performance
- [ ] Are there any obvious N+1 queries or expensive loops?
- [ ] Are database queries indexed appropriately?
- [ ] Is caching used where appropriate?

### Readability & maintainability
- [ ] Are names (variables, functions, files) clear and consistent with the codebase style?
- [ ] Is complex logic explained with comments or broken into well-named helpers?
- [ ] Is dead code removed?
- [ ] Is the change covered by updated documentation or ADR if needed?

## Tone guide

| Instead of… | Try… |
|-------------|------|
| "This is wrong." | "I think this might cause X in scenario Y — what do you think?" |
| "Why did you do it this way?" | "I'd love to understand the reasoning here — could you share more context?" |
| "You should use X." | "Have you considered X? It might help with Y." |
| "This code is messy." | "This section is a bit hard for me to follow — could we add a comment or break it up?" |

## Labelling comments (optional conventions)

Use prefixes to signal intent and reduce ambiguity:

- **nit:** Minor stylistic preference; author can ignore if they disagree.
- **suggestion:** Worth considering, but not blocking.
- **question:** Genuine curiosity — no change required unless the answer reveals a bug.
- **blocking:** Must be addressed before merge.
- **praise:** Highlight good work — keeps morale up.

## SLA expectations

| PR size | Expected first review | Expected merge |
|---------|----------------------|----------------|
| < 100 lines | 4 business hours | 1 business day |
| 100–400 lines | 1 business day | 2 business days |
| > 400 lines | Discuss with author | Negotiate |
