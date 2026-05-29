# Deloitte Engineering — Bootstrap Livello 1: Rules
# Versione: 2.0 | Data: 2026-05-16
# Posizione nel progetto: .claude/rules/
# Allineato a: bootstrap_L1_skills_v2.md (5 skill riviste)
#
# NOTA: La rule brownfield.md è stata snellita perché la skill brownfield-navigator
# contiene il workflow completo. La rule serve solo come reminder sempre attivo;
# la skill si attiva quando serve il protocollo completo.

---

# FILE: .claude/rules/code-quality.md

```markdown
# Code Quality Standards

## Naming
- Variables and functions: descriptive, intention-revealing names
- No single-letter variables except loop counters (i, j, k)
- Boolean variables: prefix with is/has/can/should
- Constants: UPPER_SNAKE_CASE

## Structure
- One class/component per file
- Group imports: stdlib → third-party → internal
- Keep files under 300 lines — split if larger
- Avoid deep nesting (max 3 levels) — extract early returns or helper functions

## Dependencies
- Never add a new dependency without checking if an existing one covers the need
- Prefer well-maintained libraries with active communities
- Pin dependency versions — no floating ranges in production
- Document WHY a dependency was chosen in .plan/decisions.md

## Logging
- Use structured logging (JSON format preferred)
- Log levels: ERROR for failures, WARN for recoverable issues, INFO for business events, DEBUG for development
- Include correlation IDs in all log entries
- Never log sensitive data (PII, credentials, tokens)
```

---

# FILE: .claude/rules/security.md

```markdown
# Security Standards

## Authentication & Authorization
- Never implement custom crypto — use established libraries
- Validate JWT tokens on every protected endpoint
- Check authorization at the service layer, not just the controller
- Session tokens must have expiration

## Data Protection
- Encrypt sensitive data at rest
- Use TLS for all external communications
- Mask PII in logs and error messages
- Never store passwords in plaintext — use bcrypt/argon2/scrypt

## Input Handling
- Validate all inputs at the boundary (API controllers, message handlers)
- Use allowlists over denylists for input validation
- Limit request body size
- Implement rate limiting on public endpoints

## Secrets Management
- No secrets in code, config files, or environment variable defaults
- Use vault/secret manager references
- Rotate credentials regularly
- .env files must be gitignored

## Deep Analysis
For thorough security review, invoke /security-review skill.
```

---

# FILE: .claude/rules/git-workflow.md

```markdown
# Git Workflow Standards

## Branching
- main/master: production-ready code only
- develop: integration branch (if used)
- feature/TICKET-short-description: new features
- fix/TICKET-short-description: bug fixes
- hotfix/TICKET-short-description: production fixes

## Commits
- Format: type(scope): description
- Types: feat, fix, refactor, test, docs, chore, perf, ci
- Description: imperative mood, lowercase, no period at end
- Body: explain WHY, not WHAT (the diff shows WHAT)
- Max 72 chars for subject line

## Pull Requests
- One logical change per PR
- Include ticket reference in PR title
- Description must explain: what changed, why, how to test
- All tests must pass before merge
- At least one approval required

## Examples
- feat(auth): add OAuth2 login flow
- fix(api): handle null response from payment gateway
- refactor(users): extract validation into shared module
- test(orders): add edge cases for discount calculation
```

---

# FILE: .claude/rules/brownfield.md
# NOTA: Versione snellita. Il workflow completo è nella skill brownfield-navigator.
# Questa rule è un reminder sempre attivo — la skill si carica quando serve il protocollo completo.

```markdown
# Brownfield Development — Quick Rules

These rules are ALWAYS active. For the full brownfield protocol
(reconnaissance, pattern discovery, impact analysis), the brownfield-navigator
skill activates automatically when working on existing code.

## Core Principles
- Read existing code BEFORE writing new code — always
- Follow existing patterns even if you'd do it differently
- Use the same libraries already in the project for similar tasks
- Keep changes minimal — don't refactor unrelated code in the same PR
- Add tests BEFORE modifying legacy code (characterization tests)

## When Patterns Conflict
- The most local pattern wins (module > package > project)
- If you must deviate, document the reason in .plan/decisions.md
- Never mix two different patterns in the same module
```

---

# FILE: .claude/rules/documentation.md

```markdown
# Documentation Standards

## Code Documentation
- Public APIs: document parameters, return values, exceptions, and usage examples
- Complex algorithms: explain the approach in a comment block above
- Non-obvious decisions: add a comment explaining WHY, not WHAT
- Keep comments up to date — stale comments are worse than no comments

## Project Documentation
- Architecture decisions: record in .plan/decisions.md using ADR format
- API changes: update docs/api-contracts.md
- New dependencies: document rationale in .plan/decisions.md
- Setup instructions: keep docs/architecture.md current

## ADR Format (Architecture Decision Record)
- Title: short description of the decision
- Status: proposed | accepted | deprecated | superseded
- Context: what is the issue we're facing?
- Decision: what did we decide?
- Consequences: what are the trade-offs?
```

---

# FILE: .claude/rules/testing.md
# Questo file ha un path filter: si carica solo quando Claude lavora su file di test

```markdown
---
paths:
  - "**/*test*"
  - "**/*spec*"
  - "**/__tests__/**"
  - "**/test/**"
  - "**/tests/**"
---

# Testing Standards

## Test Structure
- Arrange-Act-Assert pattern (or Given-When-Then)
- One assertion per test when possible
- Descriptive test names: "should [expected behavior] when [condition]"
- Group related tests with describe/context blocks

## Test Quality
- Tests must be independent — no shared mutable state
- Tests must be deterministic — same result every run
- No sleep/delay in tests — use mocks or async utilities
- Clean up test data after each test (or use transactions)

## What to Test
- Happy path: the main success scenario
- Edge cases: null, empty, boundary values
- Error paths: invalid input, service failures, timeouts
- Security: unauthorized access, injection attempts

## What NOT to Test
- Framework internals (trust the framework)
- Trivial getters/setters with no logic
- Third-party library behavior
- Implementation details that may change
```
