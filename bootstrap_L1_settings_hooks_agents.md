# Deloitte Engineering — Bootstrap Livello 1: Settings, Hooks, Templates
# Versione: 2.0 | Data: 2026-05-16
# Allineato a: bootstrap_L1_skills_v2.md (5 skill riviste)
#
# NOTA v2: Rimossi i subagent security-reviewer e test-validator.
# Motivazione: le skill security-review e structured-dev (fase REVIEW) coprono
# gli stessi casi d'uso in modo più completo. I subagent nella v1 erano
# ridondanti — facevano le stesse cose delle skill ma con meno contesto.
# Le skill possono invocare subagent internamente quando serve (structured-dev
# lo fa nella fase REVIEW per task complessi).

---

# FILE: .claude/settings.json
# Configurazione base del progetto: permessi e hook

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Glob",
      "Grep",
      "Bash(npm test *)",
      "Bash(npm run lint *)",
      "Bash(npm run build *)",
      "Bash(mvn test *)",
      "Bash(mvn compile *)",
      "Bash(gradle test *)",
      "Bash(pytest *)",
      "Bash(python -m pytest *)",
      "Bash(git status *)",
      "Bash(git log *)",
      "Bash(git diff *)",
      "Bash(git blame *)",
      "Bash(git branch *)",
      "Bash(git add *)",
      "Bash(git commit *)",
      "Bash(cat *)",
      "Bash(ls *)",
      "Bash(find *)",
      "Bash(wc *)",
      "Bash(head *)",
      "Bash(tail *)",
      "Bash(grep *)"
    ],
    "deny": [
      "Bash(rm -rf /)",
      "Bash(rm -rf /*)",
      "Bash(rm -rf ~)",
      "Bash(rm -rf ~/*)",
      "Bash(chmod 777 *)",
      "Bash(curl * | bash)",
      "Bash(wget * | bash)",
      "Bash(git push --force *)",
      "Bash(git push -f *)",
      "Bash(DROP TABLE *)",
      "Bash(DROP DATABASE *)",
      "Bash(TRUNCATE *)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "echo '[HOOK] File modified — remember to run tests before committing'",
            "timeout": 5
          }
        ]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "if": "Bash(rm -rf *)",
            "command": "echo 'BLOCKED: Recursive delete is not allowed' >&2 && exit 2",
            "timeout": 5
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "echo '{\"hookSpecificOutput\":{\"hookEventName\":\"SessionStart\",\"additionalContext\":\"Deloitte L1 bootstrap active. Skills: /spec-to-code, /structured-dev, /security-review, /webapp-testing. Brownfield-navigator auto-activates. Specs in /specs/, docs in /docs/, plan in .plan/. Read specs before implementing.\"}}'",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

## Note sulla Configurazione

### Permessi Allow
I comandi che Claude può eseguire SENZA chiedere conferma:
- **Lettura**: tutti i tool di lettura file (Read, Glob, Grep)
- **Test**: comandi di test per i principali stack (npm, mvn, gradle, pytest)
- **Build**: comandi di build
- **Git**: operazioni git di lettura e commit (ma NON push)
- **Utility**: comandi di ispezione file (cat, ls, find, wc, head, tail, grep)

### Permessi Deny
Comandi bloccati in modo ASSOLUTO:
- Cancellazioni ricorsive di directory critiche
- Permessi troppo aperti (chmod 777)
- Esecuzione di script scaricati da internet (curl|bash)
- Force push su git
- Operazioni distruttive su database

### Personalizzazione per Stack
Ogni team aggiunge i propri comandi di test/build/lint nella sezione `allow`.

Esempio Node.js:
```json
"Bash(npx jest *)",
"Bash(npx eslint *)",
"Bash(npx prettier *)"
```

Esempio Java/Spring:
```json
"Bash(mvn spring-boot:run *)",
"Bash(mvn clean install *)"
```

Esempio Python:
```json
"Bash(ruff check *)",
"Bash(ruff format *)",
"Bash(python -m playwright *)"
```

---

# HOOKS AVANZATI (opzionali — per team che vogliono più automazione)

## Hook: Lint automatico dopo ogni modifica file
# Decommentare nel settings.json e adattare il comando.

```json
{
  "PostToolUse": [
    {
      "matcher": "Edit|Write",
      "hooks": [
        {
          "type": "command",
          "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/post-edit-lint.sh",
          "args": [],
          "timeout": 30
        }
      ]
    }
  ]
}
```

### Script: .claude/hooks/post-edit-lint.sh

```bash
#!/bin/bash
# Post-edit lint hook
# Reads the modified file path from stdin JSON and runs the appropriate linter

FILE_PATH=$(jq -r '.tool_input.file_path // .tool_input.filePath // empty' < /dev/stdin)

if [ -z "$FILE_PATH" ]; then
  exit 0
fi

EXTENSION="${FILE_PATH##*.}"

case "$EXTENSION" in
  js|jsx|ts|tsx)
    if command -v npx &> /dev/null && [ -f "package.json" ]; then
      npx eslint --fix "$FILE_PATH" 2>/dev/null
    fi
    ;;
  py)
    if command -v ruff &> /dev/null; then
      ruff check --fix "$FILE_PATH" 2>/dev/null
    elif command -v flake8 &> /dev/null; then
      flake8 "$FILE_PATH" 2>/dev/null
    fi
    ;;
  java)
    echo "Java file modified: $FILE_PATH"
    ;;
esac

exit 0
```

---

# SUBAGENT
# Posizione: .claude/agents/
#
# NOTA v2: I subagent security-reviewer e test-validator della v1 sono stati rimossi.
# Le skill security-review e structured-dev coprono gli stessi casi d'uso.
# Manteniamo UN solo subagent generico: il codebase-explorer, utile per la
# reconnaissance del brownfield-navigator e per qualsiasi esplorazione parallela.

## FILE: .claude/agents/codebase-explorer.md

```markdown
---
name: codebase-explorer
description: Explores and maps an existing codebase structure, patterns, and conventions
tools: Read, Grep, Glob, Bash
model: sonnet
---

You are a codebase analyst. Your job is to explore an existing codebase and report
its structure, patterns, and conventions. You do NOT modify code — only read and analyze.

## Tasks

When asked to explore a codebase or module:

1. **Structure**: Map the directory layout, identify layers (controller/service/repository, etc.)
2. **Patterns**: Identify naming conventions, error handling patterns, logging approach
3. **Stack**: Detect language, framework, build system, test framework, key dependencies
4. **Conventions**: How are similar things done? Find 3+ examples of the same pattern
5. **Configuration**: Read config files, environment setup, deployment descriptors

## Output Format

```markdown
## Codebase Profile: [module/project name]

### Stack
- Language: [X]
- Framework: [X]
- Build: [X]
- Test: [X]

### Structure
[directory tree of key folders]

### Patterns Observed
- Error handling: [description + example file:line]
- Logging: [description + example file:line]
- Naming: [description + examples]
- Data access: [description + example file:line]

### Conventions
[3+ examples of recurring patterns with file references]

### Notes
[anything unusual, inconsistent, or noteworthy]
```

Do NOT suggest improvements. Do NOT judge the code. Just report what you find.
```

---

# DIRECTORY TEMPLATES
# Struttura delle cartelle di progetto che il bootstrap crea

## FILE: docs/architecture.md (template)

```markdown
# System Architecture

## Overview
[Brief description of the system and its purpose]

## Components
[List main components/services and their responsibilities]

## Technology Stack
- Language: [e.g., Java 17, Python 3.11, TypeScript 5.x]
- Framework: [e.g., Spring Boot 3.x, FastAPI, Next.js]
- Database: [e.g., PostgreSQL 15, MongoDB 7]
- Message Queue: [e.g., Kafka, RabbitMQ — if applicable]
- Cache: [e.g., Redis — if applicable]

## Architecture Diagram
[Description or reference to diagram]

## Key Design Decisions
See .plan/decisions.md for Architecture Decision Records

## External Dependencies
[List external services, APIs, third-party integrations]
```

## FILE: docs/client-constraints.md (template)

```markdown
# Client Constraints

## Technology Constraints
- Approved frameworks: [list]
- Prohibited libraries: [list with reasons]
- Required Java/Node/Python version: [version]
- Required database: [type and version]

## Security Requirements
- Authentication method: [e.g., OAuth2, SAML, client certificates]
- Data classification: [e.g., confidential, internal, public]
- Encryption requirements: [at rest, in transit]
- Compliance standards: [e.g., PCI-DSS, GDPR, SOX]

## UI/UX Standards
- Design system: [e.g., Material UI, client custom]
- Accessibility requirements: [e.g., WCAG 2.1 AA]
- Browser support: [list]
- Responsive requirements: [breakpoints]

## Infrastructure Constraints
- Deployment target: [e.g., Kubernetes, VM, serverless]
- Cloud provider: [e.g., AWS, Azure, GCP, on-premise]
- CI/CD pipeline: [e.g., Jenkins, GitLab CI, GitHub Actions]
- Monitoring: [e.g., Datadog, Prometheus, ELK]

## Process Constraints
- Code review requirements: [e.g., 2 approvals]
- Release process: [e.g., weekly releases, continuous deployment]
- Documentation requirements: [e.g., API docs mandatory]
```

## FILE: docs/api-contracts.md (template)

```markdown
# API Contracts

## Base URL
- Development: [URL]
- Staging: [URL]
- Production: [URL]

## Authentication
[How API authentication works]

## Endpoints
[Document each endpoint with: method, path, request body, response, error codes]

## Error Format
[Standard error response format used in this project]

## Versioning
[How API versioning is handled]
```

## FILE: .plan/plan.md (template)

```markdown
# Implementation Plan

## Current Sprint/Phase
[Description of current work phase]

## Tasks
| # | Task | Spec Ref | Status | Notes |
|---|------|----------|--------|-------|
| 1 | [task description] | [FS §X.X] | Not Started / In Progress / Done / Blocked | [notes] |

## Dependencies
[List any external dependencies or blockers]

## Timeline
[Key dates and milestones]
```

## FILE: .plan/progress.md (template)

```markdown
# Progress Log

## Format
- [YYYY-MM-DD] [Task] — [Outcome] — [Notes]

## Traceability Matrix
| Spec Requirement | Task | Status | Files |
|-----------------|------|--------|-------|
| [FS §X.X - description] | Task N | ⬜ Not Started | [files] |

## Log
[entries added as work progresses]
```

## FILE: .plan/decisions.md (template)

```markdown
# Architecture Decision Records

## Format: ADR-NNN: Title

### ADR-001: [Title]
- **Date**: YYYY-MM-DD
- **Status**: Proposed | Accepted | Deprecated | Superseded
- **Context**: [What is the issue?]
- **Decision**: [What did we decide?]
- **Consequences**: [What are the trade-offs?]
```

## FILE: .plan/lessons-learned.md (template)

```markdown
# Lessons Learned

## Format
- [YYYY-MM-DD] **[Category]**: [What happened] → [What we learned] → [Action taken]

## Categories
- BUG: bugs found and root causes
- PERF: performance issues and solutions
- ARCH: architectural insights
- TOOL: tooling discoveries
- PROCESS: process improvements

## Log
[entries added as lessons are learned]
```

## FILE: specs/README.md (template)

```markdown
# Specifications from Solaria

This directory contains the specification files generated by the Solaria SDLC Suite.
These files are the source of truth for what needs to be implemented.

## Expected Files
- `functional-spec.md` — What the system should do (requirements, acceptance criteria)
- `technical-spec.md` — How it should be built (architecture, data model, integrations)
- `implementation-plan.md` — Suggested implementation sequence

## How to Use
1. Place spec files from Solaria in this directory
2. Run /spec-to-code to parse specs, detect gaps, and create implementation plan
3. Implementation plan is saved to .plan/plan.md
4. Track progress in .plan/progress.md with traceability back to spec sections

## Do NOT
- Modify spec files manually (they come from Solaria)
- Start implementing without running /spec-to-code first
- Ignore gaps flagged during spec parsing
```
