# Deloitte Engineering — Bootstrap Livello 1: Indice e Guida alla Distribuzione
# Versione: 2.0 | Data: 2026-05-16
# Allineato a: skill set v2, CLAUDE.md v2, rules v2, settings v2

## Struttura Completa del Bootstrap

Questa è la struttura di directory che viene distribuita come template.
Quando si inizia un nuovo progetto, si copia questa struttura nella root del repository.

```
project-root/
│
├── CLAUDE.md                              # Istruzioni progetto (< 200 righe)
├── CLAUDE.local.md                        # Preferenze personali (gitignored)
│
├── .claude/                               # Directory configurazione Claude Code
│   ├── settings.json                      # Permessi, hook, configurazione
│   │
│   ├── rules/                             # Regole modulari (sempre attive)
│   │   ├── code-quality.md                # Standard qualità codice
│   │   ├── security.md                    # Vincoli sicurezza
│   │   ├── git-workflow.md                # Convenzioni git
│   │   ├── brownfield.md                  # Quick rules codice esistente (reminder)
│   │   ├── documentation.md              # Standard documentazione
│   │   └── testing.md                     # Standard test (path-scoped: solo file test)
│   │
│   ├── skills/                            # Skill operative
│   │   ├── brownfield-navigator/SKILL.md  # Auto-attiva su codice esistente
│   │   ├── spec-to-code/SKILL.md          # /spec-to-code — handoff Solaria → Claude Code
│   │   ├── structured-dev/SKILL.md        # /structured-dev — ciclo completo dev
│   │   ├── security-review/SKILL.md       # /security-review — analisi OWASP
│   │   └── webapp-testing/SKILL.md        # /webapp-testing — test browser Playwright
│   │
│   ├── agents/                            # Subagent
│   │   └── codebase-explorer.md           # Esplorazione codebase (usato da brownfield-navigator)
│   │
│   └── hooks/                             # Script per automazione
│       └── post-edit-lint.sh              # Lint dopo edit (opzionale)
│
├── docs/                                  # Documentazione tecnica progetto
│   ├── architecture.md                    # Architettura sistema (template)
│   ├── api-contracts.md                   # Contratti API (template)
│   └── client-constraints.md             # Vincoli cliente (template)
│
├── .plan/                                 # Piano e tracking progetto
│   ├── plan.md                            # Piano implementazione
│   ├── progress.md                        # Log avanzamento + traceability matrix
│   ├── decisions.md                       # ADR — decisioni architetturali
│   └── lessons-learned.md                 # Lesson learned
│
└── specs/                                 # Specifiche da Solaria (output SDLC)
    └── README.md                          # Istruzioni su come usare le specs
```

## Mappa dei File del Bootstrap

| Componente | File sorgente su Solaria | Cosa diventa nel repo |
|------------|--------------------------|----------------------|
| CLAUDE.md + CLAUDE.local.md | bootstrap_L1_CLAUDE.md (v2) | `CLAUDE.md` e `CLAUDE.local.md` nella root |
| 6 regole modulari | bootstrap_L1_rules.md (v2) | 6 file in `.claude/rules/` |
| 5 skill operative | bootstrap_L1_skills_v2.md | 5 cartelle in `.claude/skills/` |
| settings.json + hook + agent + templates | bootstrap_L1_settings_hooks_agents.md (v2) | `.claude/`, `docs/`, `.plan/`, `specs/` |

## Come Distribuire

### Opzione A: GitHub Enterprise Template Repository (RACCOMANDATA)
1. Creare repository `deloitte-claude-bootstrap` su GitHub Enterprise
2. Committare l'intera struttura
3. Configurarlo come "Template Repository" nelle settings GitHub
4. Quando si inizia un progetto: "Use this template" → crea nuovo repo con la struttura
5. Oppure: copiare la directory `.claude/` in un repo esistente (brownfield)

### Opzione B: Script di Setup
```bash
#!/bin/bash
# setup-claude-bootstrap.sh
# Scarica e installa il bootstrap L1 nel progetto corrente

BOOTSTRAP_REPO="https://github.deloitte.com/engineering/claude-bootstrap"
BRANCH="main"

echo "🚀 Installing Deloitte Claude Code Bootstrap L1..."

# Clone temporaneo
git clone --depth 1 --branch $BRANCH $BOOTSTRAP_REPO /tmp/claude-bootstrap

# Copia struttura
cp -r /tmp/claude-bootstrap/.claude .
cp -r /tmp/claude-bootstrap/docs .
cp -r /tmp/claude-bootstrap/.plan .
cp -r /tmp/claude-bootstrap/specs .

# Template files
cp /tmp/claude-bootstrap/CLAUDE.md .
cp /tmp/claude-bootstrap/CLAUDE.local.md .

# Aggiungi al gitignore
echo "CLAUDE.local.md" >> .gitignore

# Cleanup
rm -rf /tmp/claude-bootstrap

echo "✅ Bootstrap installed. Next steps:"
echo "   1. Edit docs/architecture.md with your project architecture"
echo "   2. Edit docs/client-constraints.md with client rules"
echo "   3. Edit CLAUDE.local.md with your personal preferences"
echo "   4. Add specs from Solaria to specs/"
echo "   5. Customize .claude/settings.json for your stack"
```

## Processo di Governance

### Chi Mantiene il Bootstrap
- **Owner**: Team Technical Governance (Archetipo 6)
- **Contributors**: CoE GenAI, Cloud Native Dev, App Modernization
- **Review**: ogni modifica al bootstrap richiede review da almeno 2 team

### Come Aggiungere Nuove Skill
1. Lo sviluppatore crea la skill nel proprio progetto
2. Test di ammissione: "Claude sa già farlo senza questa skill?" → Se sì, non è una skill
3. Se è utile cross-progetto, propone PR al repository bootstrap
4. Review da Technical Governance + almeno 1 team diverso
5. Se approvata, entra nel bootstrap L1 (universale) o L2 (stack-specific)

### Versioning
- Il bootstrap segue semantic versioning: MAJOR.MINOR.PATCH
- MAJOR: cambiamenti breaking (nuove regole obbligatorie)
- MINOR: nuove skill o regole opzionali
- PATCH: fix e miglioramenti
- I progetti NON sono obbligati ad aggiornare immediatamente
- Changelog mantenuto nel repository

## Flusso Operativo Completo

```
Solaria (SDLC Suite)
    │
    │ Genera: analisi funzionale, tecnica, architettura, piano
    │ Output: file .md in formato Claude Code-ready
    │
    ▼
specs/ directory nel progetto
    │
    │ Lo sviluppatore invoca /spec-to-code
    │
    ▼
Spec-to-Code Bridge
    │
    ├── Parsing specifiche Solaria
    ├── Gap detection (cosa manca? cosa è ambiguo?)
    ├── Risoluzione gap con lo sviluppatore
    ├── Creazione piano implementativo → .plan/plan.md
    └── Setup traceability matrix → .plan/progress.md
    │
    ▼
Claude Code (con Bootstrap L1)
    │
    ├── CLAUDE.md → regole base (sempre attivo)
    ├── .claude/rules/ → standard specifici (sempre attivi)
    ├── docs/ → contesto progetto (caricato on demand)
    │
    │ Per task non-triviali: /structured-dev
    │
    ▼
Structured Development Cycle
    │
    ├── UNDERSTAND → conferma scope con sviluppatore
    ├── DESIGN → approccio tecnico (semplice o dettagliato)
    ├── PLAN → task list con test-first steps
    ├── IMPLEMENT (TDD) → test → code → refactor → commit
    │   │
    │   ├── brownfield-navigator (auto) → reconnaissance, pattern matching
    │   └── codebase-explorer (subagent) → esplorazione parallela
    │
    ├── REVIEW → self-review checklist
    │   │
    │   └── /security-review (su richiesta) → analisi OWASP
    │
    └── COMPLETE → aggiorna progress.md, traceability, git status
    │
    │ Per applicazioni web: /webapp-testing
    │
    ▼
Webapp Testing (opzionale)
    │
    └── Test in browser reale con Playwright
```

## Relazione tra Componenti

```
┌─────────────────────────────────────────────────────────┐
│ SEMPRE ATTIVI (caricati ad ogni sessione)                │
│                                                         │
│  CLAUDE.md          → regole base, workflow, mindset    │
│  .claude/rules/*    → standard qualità, sicurezza, git  │
│  settings.json      → permessi, hook, deny list         │
│  SessionStart hook  → reminder skill disponibili        │
└─────────────────────────────────────────────────────────┘
         │
         │ attivano quando servono
         ▼
┌─────────────────────────────────────────────────────────┐
│ ON-DEMAND (attivati dallo sviluppatore o auto)          │
│                                                         │
│  brownfield-navigator  → AUTO quando lavora su codice   │
│                          esistente                      │
│  /spec-to-code         → MANUALE all'inizio progetto    │
│  /structured-dev       → MANUALE per task complessi     │
│  /security-review      → MANUALE prima di merge         │
│  /webapp-testing       → MANUALE per test frontend      │
│  codebase-explorer     → SUBAGENT invocato dalle skill  │
└─────────────────────────────────────────────────────────┘
         │
         │ producono/aggiornano
         ▼
┌─────────────────────────────────────────────────────────┐
│ ARTEFATTI DI PROGETTO                                   │
│                                                         │
│  .plan/plan.md          → piano implementativo          │
│  .plan/progress.md      → log + traceability matrix     │
│  .plan/decisions.md     → ADR                           │
│  .plan/lessons-learned  → lesson learned                │
│  docs/*                 → documentazione tecnica        │
└─────────────────────────────────────────────────────────┘
```
