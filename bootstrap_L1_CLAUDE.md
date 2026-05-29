# Deloitte Engineering — Bootstrap Livello 1: CLAUDE.md Base
# Versione: 2.0 | Data: 2026-05-16
# Posizione nel progetto: CLAUDE.md nella root del progetto
# NOTE: Questo file deve restare SOTTO le 200 righe. Ogni riga deve giustificare la sua presenza.
# Allineato a: bootstrap_L1_skills_v2.md (5 skill riviste)

---

# FILE: CLAUDE.md (root del progetto)

```markdown
# Deloitte Engineering — Project Standards

## Identity
You are working on a Deloitte delivery project. Follow these standards strictly.
Client code is not yours — treat it with respect. Consistency beats cleverness.

## Workflow: Spec-Driven Development
1. Read specs in /specs/ BEFORE writing any code
2. For new implementations from Solaria specs: use /spec-to-code to parse, detect gaps, and plan
3. Review /docs/ for architecture, constraints, and existing patterns
4. For non-trivial tasks (3+ files or new patterns): use /structured-dev
5. For any work on existing code: brownfield-navigator activates automatically
6. Update .plan/progress.md after completing each task
7. Never modify files outside the scope defined in the current spec

## Brownfield First (Default Mindset)
Most of our work is on existing client codebases. Assume brownfield unless told otherwise.
- ALWAYS explore existing code before writing new code
- Match existing patterns: naming, structure, error handling, logging
- If the codebase uses a pattern you disagree with, follow it anyway — consistency wins
- Document any deviation from existing patterns in .plan/decisions.md with rationale
- When adding to an existing module, read the ENTIRE module first

## Code Quality
- Follow the language conventions already established in this project
- No hardcoded credentials, tokens, API keys, or connection strings
- All public functions/methods must have documentation
- Error handling is mandatory — no empty catch blocks, no silent failures
- Keep functions under 50 lines — extract if longer
- No TODO/FIXME without a ticket reference

## Security
- Input validation on ALL external inputs
- Parameterized queries only — never concatenate SQL
- Sanitize all user-facing output to prevent XSS
- No sensitive data in logs (mask PII, tokens, passwords)
- For thorough security analysis: use /security-review before merge

## Git Workflow
- Branch: feature/TICKET-description or fix/TICKET-description
- Commits: conventional format — feat:, fix:, refactor:, test:, docs:, chore:
- One logical change per commit — atomic commits
- Never commit directly to main/master/develop
- Run tests before committing

## Testing
- Unit tests for all business logic
- Integration tests for API endpoints and external service interactions
- Test edge cases: null inputs, empty collections, boundary values, error paths
- Test coverage target: defined per project in CLAUDE.local.md
- Tests must be deterministic — no dependency on external state or timing

## Available Skills
- /spec-to-code — Parse Solaria specs, detect gaps, create implementation plan
- /structured-dev — Full development cycle: understand → design → plan → TDD → review
- /security-review — OWASP-based security analysis of code changes
- /webapp-testing — Test web apps in real browser with Playwright
- brownfield-navigator — Activates automatically on existing codebases

## Project Context
See @docs/architecture.md for system architecture
See @docs/api-contracts.md for API specifications
See @docs/client-constraints.md for client-specific rules and restrictions
See @.plan/plan.md for current implementation plan
```

---

# FILE: CLAUDE.local.md (template — ogni sviluppatore lo personalizza, gitignored)

```markdown
# Personal Project Preferences
# This file is gitignored. Customize for your local setup.

## Environment
- Local API URL: http://localhost:8080
- Database: local PostgreSQL on port 5432
- Test command: [INSERT YOUR TEST COMMAND]
- Lint command: [INSERT YOUR LINT COMMAND]
- Build command: [INSERT YOUR BUILD COMMAND]

## Test Coverage Target
- Minimum coverage: 80%

## Personal Preferences
- Preferred test framework: [jest/pytest/junit/etc.]
- IDE: [vscode/intellij/etc.]

## Client-Specific Notes
- [Add any client-specific context that applies only to your work]
```
