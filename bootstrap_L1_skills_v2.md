# Bootstrap L1 — Skill Set v2 (Rivisto)

> Versione: 2.0 — 15 maggio 2026
> Stato: Proposta per review
> Principio: skill custom solo dove Claude non sa fare da solo, o dove serve codificare il modo Deloitte

---

## Panoramica Skill Set

| # | Skill | Tipo | Origine | Scopo |
|---|-------|------|---------|-------|
| 1 | **brownfield-navigator** | Custom Deloitte | Originale | Protocollo per lavorare su codice esistente dei clienti |
| 2 | **spec-to-code** | Custom Deloitte | Originale | Handoff Solaria → Claude Code |
| 3 | **structured-dev** | Riscrittura controllata | Ispirata a Superpowers (obra) | Ciclo completo: design → plan → TDD → review |
| 4 | **security-review** | Custom Deloitte | Principi OWASP | Review sicurezza senza dipendenze esterne |
| 5 | **webapp-testing** | Riscrittura controllata | Ispirata a Anthropic | Test applicazioni web con Playwright |

### Criteri di inclusione
- **Capability Uplift**: dà a Claude capacità che non ha nativamente
- **Encoded Preference con contenuto reale**: codifica un processo specifico Deloitte che Claude non può indovinare
- **Nessuna dipendenza esterna**: tutto autocontenuto, nessun `npx install`, nessun download runtime

---

## SKILL 1: brownfield-navigator

```
Percorso: .claude/skills/brownfield-navigator/SKILL.md
```

```markdown
---
name: brownfield-navigator
description: "Use when working on existing codebases. Enforces codebase exploration, pattern matching, and impact analysis before any modification. Prevents Claude from writing 'clean' code that doesn't integrate with existing patterns."
---

# Brownfield Navigator — Working on Existing Code

You are modifying an existing codebase owned by a client. Your code MUST integrate
seamlessly with what already exists. Writing "better" code that breaks conventions
is worse than writing conventional code that fits.

## Iron Rule

```
NEVER write new code before completing the Reconnaissance phase.
NEVER deviate from existing patterns without documenting why in decisions.md.
```

## Phase 1: Reconnaissance (MANDATORY before any code change)

### 1.1 Codebase Profile
Before touching anything, build a mental model:

```bash
# Project structure
find . -name "*.java" -o -name "*.py" -o -name "*.ts" -o -name "*.cs" | head -50
# Build system
cat pom.xml 2>/dev/null || cat package.json 2>/dev/null || cat build.gradle 2>/dev/null || cat *.csproj 2>/dev/null
# Framework detection
grep -r "springframework\|express\|fastapi\|django\|angular\|react\|blazor" --include="*.xml" --include="*.json" --include="*.py" --include="*.ts" -l | head -10
# Configuration
ls -la src/main/resources/ 2>/dev/null || ls -la config/ 2>/dev/null
# Recent changes (understand what the team is working on)
git log --oneline -20
```

Produce a brief codebase profile:
- **Language/Framework**: [detected]
- **Build system**: [detected]
- **Architecture pattern**: [MVC, hexagonal, layered, microservices...]
- **Naming conventions**: [camelCase, snake_case, prefixes...]
- **Test framework**: [JUnit, pytest, Jest, xUnit...]
- **Key dependencies**: [top 5-10]

### 1.2 Pattern Discovery
For the specific type of change you need to make, find AT LEAST 3 existing examples
of how the codebase handles the same type of operation:

```
EXAMPLE: If adding a new REST endpoint:
1. Find 3 existing controllers/routes
2. Note: URL pattern, HTTP method conventions, request/response format
3. Note: Error handling pattern, validation approach, logging
4. Note: Test structure for existing endpoints
```

If you cannot find 3 examples → the codebase doesn't have this pattern yet.
Flag this to the developer: "No existing pattern found for [X]. Proposing new pattern.
Please confirm before I proceed."

### 1.3 Client Constraints Check
Read `docs/client-constraints.md` if it exists. These are NON-NEGOTIABLE:
- Approved frameworks and libraries
- Security requirements
- UI/UX standards
- Coding standards and linting rules
- Deployment constraints

## Phase 2: Impact Analysis (MANDATORY before modifying any file)

Before modifying a file:

```bash
# Who imports/uses this file?
grep -r "import.*FileName\|from.*filename\|require.*filename" --include="*.java" --include="*.py" --include="*.ts" -l
# What tests cover this file?
grep -r "FileName\|filename" tests/ test/ __tests__/ src/test/ -l
# Recent changes to this file
git log --oneline -5 -- path/to/file
```

Produce impact assessment:
- **Files that depend on this**: [list]
- **Tests that cover this**: [list]
- **Risk level**: LOW (isolated change) | MEDIUM (multiple dependents) | HIGH (core module)

If risk = HIGH → present impact to developer before proceeding.

## Phase 3: Implementation

### Rules
1. **Match existing patterns exactly** — same naming, same structure, same error handling
2. **Same file organization** — if existing services are in `src/services/`, yours goes there too
3. **Same test structure** — if existing tests use `@Test` with `assertThat`, you do too
4. **Same logging** — if they use SLF4J with `log.info()`, you do too. Not `System.out.println()`
5. **Same dependency injection** — if they use constructor injection, you do too
6. **Document deviations** — if you MUST deviate, add entry to `.plan/decisions.md`:
   ```
   ## [Date] Deviation: [what]
   - Existing pattern: [X]
   - New approach: [Y]
   - Reason: [specific technical reason]
   - Approved by: [developer name or "pending"]
   ```

### Anti-Patterns (NEVER do these)
- Introduce a new library when an existing one does the same thing
- Refactor unrelated code "while you're in there"
- Change formatting/style of existing code
- Add abstractions the codebase doesn't use (e.g., adding interfaces where none exist)
- Use a different error handling pattern than the rest of the codebase

## Phase 4: Verification

After implementation:
1. Run existing tests — ALL must pass, not just yours
2. Verify your code follows the patterns documented in Phase 1
3. Check that no unintended files were modified: `git diff --stat`
4. Update `.plan/progress.md` with what was done
```

---

## SKILL 2: spec-to-code

```
Percorso: .claude/skills/spec-to-code/SKILL.md
```

```markdown
---
name: spec-to-code
description: "Use when starting implementation from Solaria-generated specifications. Parses spec files from /specs/, creates implementation plan, maps requirements to tasks, and tracks traceability."
---

# Spec-to-Code Bridge — From Solaria Specs to Implementation

This skill handles the handoff between design (done on Solaria) and implementation
(done here in Claude Code). Solaria produces Markdown specification files that follow
a structured format. This skill parses them and creates an actionable implementation plan.

## When to Use
- Starting a new implementation from Solaria specs
- Resuming work on a partially implemented spec
- Checking coverage of spec requirements against existing code

## Step 1: Load and Parse Specs

Read all files in `/specs/` directory:

```bash
ls -la specs/
```

Expected files from Solaria:
- `functional-spec.md` — What the system should do (user stories, acceptance criteria)
- `technical-spec.md` — How it should be built (architecture, data model, integrations)
- `implementation-plan.md` — Suggested implementation sequence with prompts

Parse each file and extract:
- **Requirements**: numbered items with acceptance criteria
- **Architecture decisions**: technology choices, patterns, constraints
- **Data model**: entities, relationships, validations
- **Integration points**: APIs, external systems, events
- **Non-functional requirements**: performance, security, scalability

## Step 2: Gap Detection (BEFORE starting implementation)

Check the specs for completeness. Flag any of these:

| Gap Type | Example | Action |
|----------|---------|--------|
| **Missing acceptance criteria** | "User can search" without defining what fields, filters, result format | Flag → ask developer to clarify or decide |
| **Ambiguous requirement** | "System should be fast" without defining latency target | Flag → propose a concrete target |
| **Missing error scenarios** | Happy path defined but no error handling spec | Flag → propose error scenarios |
| **Undefined integration contract** | "Calls external API" without request/response format | Flag → cannot implement without this |
| **Conflicting requirements** | Spec says "real-time" but also "batch processing" | Flag → ask which takes priority |

Present gaps as a numbered list. Wait for developer response before proceeding.
If no gaps found, state: "Specs are complete. Proceeding to planning."

## Step 3: Create Implementation Plan

Transform specs into an ordered task list in `.plan/plan.md`:

```markdown
# Implementation Plan
Generated from: specs/functional-spec.md, specs/technical-spec.md
Date: [today]

## Task Overview
| # | Task | Spec Reference | Complexity | Dependencies |
|---|------|---------------|------------|--------------|
| 1 | [name] | functional-spec.md §2.1 | S/M/L | none |
| 2 | [name] | technical-spec.md §3.2 | M | Task 1 |

## Task Details

### Task 1: [Name]
**Spec reference**: functional-spec.md §2.1
**What**: [one sentence]
**Acceptance criteria**:
- [ ] [from spec]
- [ ] [from spec]
**Files to create/modify**:
- Create: `src/path/to/file.ext`
- Modify: `src/path/to/existing.ext`
**Test approach**:
- Unit: [what to test]
- Integration: [what to test]
```

### Ordering Rules
1. Data model / entities first
2. Core business logic second
3. API / interface layer third
4. Integration points fourth
5. UI components last
6. Cross-cutting concerns (auth, logging, error handling) woven in at each layer

## Step 4: Traceability

During implementation, every piece of code must trace back to a spec requirement.

### In progress.md
```markdown
## Traceability Matrix
| Spec Requirement | Task | Status | Files |
|-----------------|------|--------|-------|
| FS §2.1 - User login | Task 1 | ✅ Done | src/auth/login.ts, tests/auth/login.test.ts |
| FS §2.2 - Password reset | Task 3 | 🔄 In Progress | src/auth/reset.ts |
| TS §3.1 - Database schema | Task 2 | ✅ Done | migrations/001_initial.sql |
```

### In code (comments)
```typescript
// Implements: FS §2.1 - User login
// Acceptance: email + password → JWT token, 401 on invalid credentials
export async function login(email: string, password: string): Promise<AuthToken> {
```

## Step 5: Completion Check

Before marking implementation as complete:
1. Every spec requirement has at least one task
2. Every task has tests
3. Traceability matrix is complete (no gaps)
4. All acceptance criteria have corresponding test assertions
5. Gap items from Step 2 have been resolved

If any item is missing → flag it, don't mark as complete.
```

---

## SKILL 3: structured-dev

```
Percorso: .claude/skills/structured-dev/SKILL.md
```

```markdown
---
name: structured-dev
description: "Use for any non-trivial development task. Enforces a structured cycle: understand → design → plan → test-first implement → review. Prevents jumping straight to code."
disable-model-invocation: true
---

# Structured Development — Design, Plan, Build, Review

This skill enforces a disciplined development cycle. Use it for any task that involves
more than a simple bug fix or one-file change.

Invoke with: /structured-dev

## The Cycle

```
UNDERSTAND → DESIGN → PLAN → IMPLEMENT (TDD) → REVIEW → COMPLETE
```

You MUST complete each phase before moving to the next.
You MUST NOT write implementation code before the PLAN phase is complete.

---

## Phase 1: UNDERSTAND

**Goal**: Know exactly what you're building and why.

1. Read the task description / spec / issue
2. If working on existing code: run brownfield-navigator reconnaissance
3. Ask clarifying questions — ONE at a time, prefer multiple choice
4. Identify: scope, constraints, success criteria, edge cases

**Output**: Brief summary (3-5 sentences) of what you're building.
Confirm with developer before proceeding.

**Anti-pattern**: Jumping to code because "it's obvious". It's not. Spend 2 minutes understanding.

---

## Phase 2: DESIGN

**Goal**: Decide HOW to build it before writing code.

For simple tasks (1-3 files, clear pattern):
- 3-5 sentences describing approach
- List files to create/modify

For complex tasks (4+ files, new patterns, integrations):
- Architecture approach (2-3 options with trade-offs, recommend one)
- Component breakdown with responsibilities
- Data flow description
- Error handling strategy
- Save design to `docs/design-[feature].md`

**Output**: Design approved by developer.

**Anti-pattern**: Over-designing simple tasks. Match design depth to task complexity.

---

## Phase 3: PLAN

**Goal**: Break work into small, testable steps.

Create plan in `.plan/plan.md`:

```markdown
### Task N: [Component]
**Files**: create `path/file.ext`, modify `path/existing.ext`
**Steps**:
- [ ] Write failing test for [specific behavior]
- [ ] Implement minimal code to pass
- [ ] Verify all tests pass
- [ ] Commit: "feat: [description]"
```

### Planning Rules
- Each step = 2-5 minutes of work
- Each step ends with a commit
- Every implementation step has a test step BEFORE it
- No placeholders: "add error handling" → specify WHICH errors, HOW to handle
- Exact file paths, exact test commands

**Output**: Plan saved to `.plan/plan.md`

---

## Phase 4: IMPLEMENT (Test-Driven)

**Goal**: Build it, test-first.

### The TDD Loop
```
1. Write ONE failing test → run it → confirm it FAILS
2. Write MINIMAL code to pass → run it → confirm it PASSES
3. Refactor if needed → run tests → confirm still GREEN
4. Commit
5. Next test
```

### TDD Rules (Non-Negotiable)
- **No production code without a failing test first**
- If you wrote code before the test: DELETE IT. Start over.
- "Too simple to test" → test takes 30 seconds. Do it.
- "I'll test after" → tests written after prove nothing. They pass immediately.
- Match existing test patterns (see brownfield-navigator)

### When to Skip TDD (developer must explicitly approve)
- Configuration files
- Generated/scaffolded code
- Throwaway prototypes

### Commit Discipline
- One logical change per commit
- Conventional commits: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`
- Never commit failing tests (except as first step of a TDD cycle, immediately followed by implementation)

**Output**: Working, tested code. All tests green.

---

## Phase 5: REVIEW

**Goal**: Catch issues before they ship.

### Self-Review Checklist
Before presenting work as complete:

- [ ] All tests pass (run full suite, not just new tests)
- [ ] No unintended file changes: `git diff --stat`
- [ ] Code follows existing patterns (brownfield rule)
- [ ] No hardcoded values that should be configurable
- [ ] Error handling is complete (no silent failures)
- [ ] No security issues (no secrets, no SQL concatenation, input validation)
- [ ] Documentation updated if public API changed

### For Complex Changes
If the change touches 5+ files or modifies core logic:
- Use a subagent for independent code review
- Subagent gets: diff, spec reference, codebase context
- Subagent checks: correctness, patterns, edge cases, security

**Output**: Review complete, issues fixed.

---

## Phase 6: COMPLETE

1. Update `.plan/progress.md` with completed tasks
2. Update traceability matrix if working from specs
3. Final `git status` — clean working directory
4. Report to developer: what was done, what was tested, any open items

---

## Quick Reference

| Phase | Time | Output | Gate |
|-------|------|--------|------|
| Understand | 2-5 min | Summary | Developer confirms |
| Design | 5-15 min | Approach | Developer approves |
| Plan | 5-10 min | Task list | Saved to .plan/ |
| Implement | Variable | Working code | All tests green |
| Review | 5-10 min | Checklist done | No open issues |
| Complete | 2 min | Status update | Clean git status |
```

---

## SKILL 4: security-review

```
Percorso: .claude/skills/security-review/SKILL.md
```

```markdown
---
name: security-review
description: "Use to perform security review of code changes. Checks OWASP Top 10, authentication, authorization, data handling, and common vulnerability patterns. No external tools required."
disable-model-invocation: true
---

# Security Review — Code Security Analysis

Invoke with: /security-review

This skill performs a structured security review of code changes without requiring
external tools. It uses Claude's code understanding to identify vulnerability patterns.

## Scope

Review the current diff or specified files for security issues:

```bash
# Review current changes
git diff --unified=10
# Or review specific files
git diff --unified=10 -- path/to/file
```

## Review Checklist

### 1. Injection (OWASP A03)
- [ ] SQL queries use parameterized statements (no string concatenation)
- [ ] NoSQL queries use safe query builders
- [ ] OS commands use safe APIs (no shell=True, no exec with user input)
- [ ] LDAP queries are parameterized
- [ ] XML parsers disable external entities (XXE)

**Pattern to find**:
```
# DANGEROUS
query = f"SELECT * FROM users WHERE id = {user_input}"
cursor.execute(query)

# SAFE
cursor.execute("SELECT * FROM users WHERE id = %s", (user_input,))
```

### 2. Authentication & Session (OWASP A07)
- [ ] Passwords hashed with bcrypt/scrypt/argon2 (not MD5/SHA1)
- [ ] Session tokens are cryptographically random
- [ ] Session expiration is configured
- [ ] Failed login attempts are rate-limited
- [ ] Password reset tokens expire

### 3. Authorization (OWASP A01)
- [ ] Every endpoint checks authorization (not just authentication)
- [ ] No direct object references without ownership check (IDOR)
- [ ] Admin functions have role verification
- [ ] API endpoints validate caller permissions

**Pattern to find**:
```
# DANGEROUS — no authorization check
@app.get("/api/users/{user_id}/data")
def get_user_data(user_id: int):
    return db.get_user(user_id)  # Any authenticated user can access any user's data

# SAFE
@app.get("/api/users/{user_id}/data")
def get_user_data(user_id: int, current_user: User = Depends(get_current_user)):
    if current_user.id != user_id and not current_user.is_admin:
        raise HTTPException(403)
    return db.get_user(user_id)
```

### 4. Data Exposure (OWASP A02)
- [ ] Sensitive data not logged (passwords, tokens, PII)
- [ ] API responses don't include unnecessary fields
- [ ] Error messages don't expose stack traces or internal details to users
- [ ] Secrets not hardcoded (check for API keys, passwords, connection strings)

**Pattern to find**:
```bash
# Search for hardcoded secrets
grep -rn "password\s*=\s*['\"]" --include="*.py" --include="*.java" --include="*.ts"
grep -rn "api_key\s*=\s*['\"]" --include="*.py" --include="*.java" --include="*.ts"
grep -rn "secret\s*=\s*['\"]" --include="*.py" --include="*.java" --include="*.ts"
```

### 5. Input Validation (OWASP A03)
- [ ] All external inputs validated (type, length, range, format)
- [ ] File uploads: type validation, size limits, no path traversal
- [ ] URL redirects: whitelist allowed destinations
- [ ] HTML output: escaped/sanitized (XSS prevention)

### 6. Configuration & Dependencies
- [ ] Debug mode disabled in production config
- [ ] CORS configured restrictively (not `*`)
- [ ] HTTPS enforced
- [ ] Security headers present (CSP, X-Frame-Options, etc.)
- [ ] Dependencies don't have known CVEs (check versions)

### 7. Cryptography
- [ ] Using standard libraries (not custom crypto)
- [ ] TLS 1.2+ for all connections
- [ ] Adequate key lengths (RSA ≥ 2048, AES ≥ 256)
- [ ] Random values use cryptographic RNG (not Math.random())

## Output Format

```markdown
## Security Review Results

### Critical Issues (MUST fix before merge)
- **[INJECTION]** File: `src/api/users.py:45` — SQL concatenation with user input
  - Risk: SQL injection allows data exfiltration
  - Fix: Use parameterized query

### Important Issues (SHOULD fix before merge)
- **[AUTH]** File: `src/api/admin.py:12` — Missing role check on admin endpoint
  - Risk: Any authenticated user can access admin functions
  - Fix: Add `@require_role("admin")` decorator

### Observations (Consider fixing)
- **[CONFIG]** File: `config/cors.py:3` — CORS allows all origins in staging
  - Risk: Low in staging, but ensure production config is restrictive

### Passed Checks
- ✅ No hardcoded secrets found
- ✅ Password hashing uses bcrypt
- ✅ Input validation present on all public endpoints
```

## Severity Definitions

| Severity | Meaning | Action |
|----------|---------|--------|
| **Critical** | Exploitable vulnerability, data at risk | Block merge. Fix immediately. |
| **Important** | Security weakness, exploitable under conditions | Fix before merge. |
| **Observation** | Best practice deviation, low immediate risk | Track, fix when convenient. |
```

---

## SKILL 5: webapp-testing

```
Percorso: .claude/skills/webapp-testing/SKILL.md
```

```markdown
---
name: webapp-testing
description: "Use to test web applications in a real browser using Playwright. Supports navigating pages, clicking elements, filling forms, taking screenshots, and verifying UI behavior."
---

# Web Application Testing with Playwright

Test web applications by writing Python scripts that use Playwright to control
a real browser. Use this to verify that your frontend changes actually work.

## Prerequisites

Playwright must be installed in the project:
```bash
pip install playwright
playwright install chromium
```

## Decision Tree

```
Is the server already running?
├─ No → Start it first, then test
└─ Yes → Go directly to testing

Is it static HTML?
├─ Yes → Use file:// URL, read HTML to find selectors
└─ No (dynamic app) → Navigate, wait for networkidle, then inspect
```

## Basic Pattern

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()

    # Navigate
    page.goto("http://localhost:3000")
    page.wait_for_load_state("networkidle")  # CRITICAL for dynamic apps

    # Inspect (reconnaissance first)
    page.screenshot(path="/tmp/page.png", full_page=True)

    # Act
    page.fill('input[name="email"]', 'test@example.com')
    page.click('button[type="submit"]')

    # Assert
    page.wait_for_selector('.success-message')
    assert page.is_visible('.success-message')

    browser.close()
```

## Key Rules

1. **Always `wait_for_load_state("networkidle")`** before inspecting dynamic pages
2. **Screenshot first, act second** — see what's on the page before clicking
3. **Use descriptive selectors**: `text=Submit`, `role=button[name="Save"]`, `#login-form`
4. **Always close the browser** when done
5. **Headless mode always** — `launch(headless=True)`

## Common Tasks

### Fill and submit a form
```python
page.fill('#username', 'testuser')
page.fill('#password', 'testpass')
page.click('button:text("Login")')
page.wait_for_url("**/dashboard")
```

### Check page content
```python
content = page.text_content('.main-content')
assert "Welcome" in content
```

### Handle multiple pages (tabs)
```python
with page.expect_popup() as popup_info:
    page.click('a[target="_blank"]')
new_page = popup_info.value
```

### Capture console errors
```python
errors = []
page.on("console", lambda msg: errors.append(msg.text) if msg.type == "error" else None)
page.goto("http://localhost:3000")
page.wait_for_load_state("networkidle")
assert len(errors) == 0, f"Console errors: {errors}"
```

## Anti-Patterns
- ❌ Don't use `page.wait_for_timeout(5000)` — use explicit waits
- ❌ Don't inspect DOM before `networkidle` on SPAs
- ❌ Don't hardcode viewport sizes without reason
- ❌ Don't leave browsers open (always `browser.close()`)
```

---

## Note sulla Distribuzione

Tutti i file SKILL.md vanno nella directory `.claude/skills/[nome-skill]/SKILL.md` del repository template su GitHub Enterprise.

Struttura risultante:
```
.claude/skills/
├── brownfield-navigator/
│   └── SKILL.md
├── spec-to-code/
│   └── SKILL.md
├── structured-dev/
│   └── SKILL.md
├── security-review/
│   └── SKILL.md
└── webapp-testing/
    └── SKILL.md
```

### Skill rimosse (erano nella v1)
- ~~spec-review~~ → sostituita da spec-to-code (più specifica, non generica)
- ~~code-review~~ → integrata in structured-dev fase REVIEW
- ~~test-first~~ → integrata in structured-dev fase IMPLEMENT
- ~~legacy-analyze~~ → sostituita da brownfield-navigator (più operativa)
- ~~project-status~~ → ridondante, Claude sa già farlo

### Governance
- Nuove skill proposte dai team → review del Technical Governance team → merge nel template
- Ogni skill deve superare il test: "Claude sa già farlo senza questa skill?" Se sì → non è una skill, è una riga nel CLAUDE.md
