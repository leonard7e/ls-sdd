---
id: 0001
slug: ls-sdd-skills
created: 2026-07-01T14:30
status: draft
specs: []
---

# Plan: Implementierung der `ls-sdd` Skill-Sammlung

## Kontext

Ziel ist eine Sammlung von Zed-Agent-Skills, die einen Spec-Driven-Development-Workflow (SDD) in jedem Projekt ermöglichen. Die Skills sind **generisch** (nicht an ein spezifisches Projekt gebunden) und werden global unter `~/.agents/skills/` installiert. Sie sollen etablierte SDD-Ansätze – insbesondere GitHub Spec Kit und BMAD-METHOD – adaptieren, ohne deren Komplexität zu übernehmen.

Die Skills tragen den Präfix `ls-` (Leonard Siebeneicher), damit sie als "unseres" wiedererkennbar sind und nicht mit anderen SDD-Skills kollidieren.

## Architekturentscheidungen

### A1: Skill-Set – 9 Skills (Vorschlag, vom Nutzer zu bestätigen)

| Skill | Slash-Command | Zweck | Vorbild |
|---|---|---|---|
| `ls-sdd` | `\ls-sdd` | Meta-Skill: Workflow-Übersicht, aktueller Status, "was als nächstes?" | BMAD `bmad-help` |
| `ls-sdd-init` | `\ls-sdd-init` | Constitution in `AGENTS.md` erstellen/aktualisieren | Spec Kit `/speckit.constitution` |
| `ls-sdd-specify` | `\ls-sdd-specify` | Spec erstellen, ggf. mit scale-adaptiver Tiefe | Spec Kit `/speckit.specify` |
| `ls-sdd-clarify` | `\ls-sdd-clarify` | Strukturierte Klärung vor dem Planen | Spec Kit `/speckit.clarify` |
| `ls-sdd-create-plan` | `\ls-sdd-create-plan` | Plan aus einer oder mehreren Specs | Spec Kit `/speckit.plan` |
| `ls-sdd-tasklist` | `\ls-sdd-tasklist` | Tasks aus Plan erstellen | Spec Kit `/speckit.tasks` |
| `ls-sdd-analyze` | `\ls-sdd-analyze` | Konsistenzprüfung Spec↔Plan↔Tasks | Spec Kit `/speckit.analyze` |
| `ls-sdd-implement` | `\ls-sdd-implement` | Tasks abarbeiten | Spec Kit `/speckit.implement` |
| `ls-sdd-converge` | `\ls-sdd-converge` | Codebase gegen Specs prüfen, Lücken als neue Tasks | Spec Kit `/speckit.converge` |

**Offen**: Soll das Set 9 Skills umfassen (mit `ls-sdd-converge`) oder reicht 8 ohne Converge für die erste Version? Converge ist für Brownfield/Iteration wichtig, kann aber nachgeliefert werden.

### A2: Verzeichnisstruktur in Zielprojekten

Jedes Projekt, das die `ls-sdd`-Skills nutzt, erhält diese Struktur:

```
[projekt]/
├── AGENTS.md                    # Constitution (projektweit, von \ls-sdd-init)
├── plans/
│   └── 001-[identity]/
│       ├── plan.md
│       ├── tasks.md
│       └── [research.md, analysis.md]   # optional
└── specs/
    ├── 001-spec-[identity]/
    │   ├── spec.md
    │   ├── clarifications.md            # von \ls-sdd-clarify
    │   └── [additional markdown files]
    └── 002-spec-[identity]/
        └── spec.md
```

**Konventionen**:
- `AGENTS.md` enthält die Constitution – wird von Zed/Claude Code/Cursor automatisch gelesen.
- Plans und Specs haben **getrennte** ID-Sequenzen (Plan `001` und Spec `001` können beide existieren).
- Plan-Slug: `001-identity` (kein `-plan-` Marker).
- Spec-Slug: `001-spec-identity` (mit `-spec-` Marker zur Unterscheidung).
- Plans und Specs sind N:M gekoppelt: Ein Plan referenziert Specs im Frontmatter (`specs: [0001, 0002]`).
- Zusätzliche Markdown-Dateien pro Spec sind erlaubt (z.B. `menu.md`, `data-model.md`).

### A3: Frontmatter-Modell

**Spec-Frontmatter** (`specs/001-spec-x/spec.md`):
```yaml
---
id: 0001
slug: identity
created: 2026-07-01T14:30
status: draft          # draft | clarified | planned | superseded | done
scale: standard        # trivial | standard | enterprise
parent: null           # bei überarbeiteten Specs: ID des Vorgängers
---
```

**Plan-Frontmatter** (`plans/001-identity/plan.md`):
```yaml
---
id: 0001
slug: identity
created: 2026-07-01T14:30
status: draft          # draft | planned | tasked | analyzed | implementing | done | superseded
specs: [0001, 0002]    # referenzierte Spec-IDs
parent: null
---
```

**Tasks-Frontmatter** (`plans/001-identity/tasks.md`):
```yaml
---
id: 0001
plan: 0001
created: 2026-07-01T14:30
status: draft          # draft | in-progress | blocked | done
---
```

### A4: Status-Modelle

**Spec-Status**: `draft → clarified → planned → done` (mit `superseded` als Terminal bei Überarbeitung)

**Plan-Status**: `draft → planned → tasked → analyzed → implementing → done` (mit `superseded`)

**Task-Status**: pro Task `- [ ]` (offen) / `- [x]` (erledigt) / `- [~]` (blockiert)

### A5: Scale-adaptive Tiefe (Vorschlag)

`ls-sdd-specify` fragt zu Beginn nach dem Umfang:
- **trivial** (Bugfix, kleine Anpassung): minimale Spec, kein Plan, direkte Tasklist
- **standard** (Feature, neues Modul): volle Spec → Plan → Tasks
- **enterprise** (System, mehrere Teams): Spec mit nicht-funktionalen Anforderungen, Architektur-Plan, Tasks mit Abhängigkeiten

**Offen**: Soll diese Unterscheidung im Skill implementiert werden, oder ist das Overhead für die erste Version?

### A6: Task-Format

Checkbox-Liste mit Metadaten (keine reine Prosa), damit der Agent Status parsen kann:

```markdown
- [ ] T0001 Provider-Trait definieren
  - depends: -
  - files: src/provider/mod.rs
  - acceptance: Trait mit complete(), stream()
```

### A7: Skill-Scope

Global unter `~/.agents/skills/`. Begründung: SDD ist projektunabhängig; die Skills sollen in jedem Projekt verfügbar sein.

### A8: Sprache

- Skill-Bodies (SKILL.md): **Englisch** (Standard im Skills-Ökosystem, bessere Auffindbarkeit über `npx skills find`)
- Generierte Artefakte (specs, plans, tasks): **Deutsch**, wenn der Nutzer deutsch kommuniziert (ansonsten Sprache des Nutzers)

**Offen**: Soll die Sprache der Artefakte konfigurierbar sein, oder soll der Agent automatisch der Nutzer-Sprache folgen?

### A9: Konventions-Single-Source-of-Truth

Die Verzeichnisstruktur, Frontmatter-Modelle und Status-Modelle werden **nicht** in jeden Skill kopiert. Stattdessen definiert `ls-sdd` (der Meta-Skill) alle Konventionen, und die anderen Skills referenzieren ihn: "Lies `ls-sdd` für Format-Details." Das verhindert Drift bei Änderungen.

## Skill-Design (pro Skill)

### S1: `ls-sdd` (Meta-Skill)

**Frontmatter**:
```yaml
---
name: ls-sdd
description: Spec-Driven Development workflow overview and conventions. Use when the user asks about SDD, wants to know the current status of specs/plans/tasks, or asks "what's next?" in an SDD project.
---
```

**Inhalt**:
- Erklärt den SDD-Workflow (Constitution → Spec → Clarify → Plan → Tasks → Analyze → Implement → Converge)
- Definiert alle Konventionen (Verzeichnisstruktur, Frontmatter, Status, Slug-Format, ID-Format)
- Enthält Templates für spec.md, plan.md, tasks.md als Code-Blöcke
- Bietet "was als nächstes?"-Logik: scannt `specs/` und `plans/`, zeigt aktuellen Status, empfiehlt nächsten Schritt

### S2: `ls-sdd-init`

**Frontmatter**:
```yaml
---
name: ls-sdd-init
description: Create or update the project constitution in AGENTS.md. Use at project start or when foundational principles (tech stack, quality standards, conventions) should be established or revised.
---
```

**Inhalt**:
- Interviewt den Nutzer zu Projektprinzipien (Tech-Stack, Qualitätsstandards, Test-Strategie, Konventionen)
- Schreibt/aktualisiert `AGENTS.md` mit Constitution
- Falls `AGENTS.md` existiert: bestehende Constitution respektieren, nur erweitern

### S3: `ls-sdd-specify`

**Frontmatter**:
```yaml
---
name: ls-sdd-specify
description: Create a functional specification. Use when the user wants to define what to build and why, before any technical planning. Focuses on requirements and user stories, not implementation.
---
```

**Inhalt**:
- Fragt ggf. nach Scale (trivial/standard/enterprise) – falls A5 bestätigt
- Interviewt den Nutzer: Kontext, Problem, Anforderungen, Akzeptanzkriterien, Out-of-Scope, offene Fragen
- Erstellt `specs/NNNN-spec-slug/spec.md` mit Frontmatter
- NNNN: nächste freie ID (scannt `specs/`)

### S4: `ls-sdd-clarify`

**Frontmatter**:
```yaml
---
name: ls-sdd-clarify
description: Structured clarification of an existing specification. Use before creating a plan, to surface gaps in the spec (edge cases, non-functional requirements, error handling, scale).
---
```

**Inhalt**:
- Liest Spec
- Stellt systematische Fragen zu Lücken: Edge-Cases, Fehlerfälle, nicht-funktionale Anforderungen (Performance, Sicherheit, Skalierung), Abgrenzung
- Schreibt Antworten nach `specs/NNNN-spec-slug/clarifications.md`
- Setzt Spec-Status auf `clarified`

### S5: `ls-sdd-create-plan`

**Frontmatter**:
```yaml
---
name: ls-sdd-create-plan
description: Create a technical implementation plan from one or more specifications. Use when specs are clarified and the user wants to decide how to build (architecture, modules, tech stack).
---
```

**Inhalt**:
- Liest eine oder mehrere Specs (Nutzer gibt Spec-IDs an)
- Entwirft Architektur, Module, Abhängigkeiten, Risiken
- Erstellt `plans/NNNN-slug/plan.md` mit Frontmatter (`specs: [...]`)
- Optional: `research.md` für Tech-Stack-Recherche
- Setzt Plan-Status auf `planned`, referenzierte Specs auf `planned`

### S6: `ls-sdd-tasklist`

**Frontmatter**:
```yaml
---
name: ls-sdd-tasklist
description: Generate an actionable task list from an implementation plan. Use after a plan exists, to break it into ordered, dependency-aware tasks with acceptance criteria.
---
```

**Inhalt**:
- Liest Plan
- Zerlegt in Tasks mit: ID, Abhängigkeiten, Dateien, Akzeptanzkriterien
- Markiert parallelisierbare Tasks mit `[P]`
- Erstellt `plans/NNNN-slug/tasks.md`
- Setzt Plan-Status auf `tasked`

### S7: `ls-sdd-analyze`

**Frontmatter**:
```yaml
---
name: ls-sdd-analyze
description: Cross-artifact consistency analysis between spec, plan, and tasks. Use after tasks are generated, before implementation, to catch gaps and contradictions.
---
```

**Inhalt**:
- Prüft: Jede Spec-Anforderung in Tasks abgedeckt? Tasks ohne Spec-Bezug? Akzeptanzkriterien übersetzt?
- Schreibt Ergebnis nach `plans/NNNN-slug/analysis.md`
- Setzt Plan-Status auf `analyzed` (oder zurück auf `tasked` bei Problemen)

### S8: `ls-sdd-implement`

**Frontmatter**:
```yaml
---
name: ls-sdd-implement
description: Execute tasks from a task list in order. Use when the plan is analyzed and the user wants to start implementation. Respects task dependencies and marks tasks done as they complete.
---
```

**Inhalt**:
- Liest `tasks.md`
- Arbeitet Tasks in Abhängigkeitsreihenfolge ab
- Markiert erledigte Tasks mit `- [x]`
- Setzt Plan-Status auf `implementing`, am Ende auf `done`

### S9: `ls-sdd-converge` (optional, falls A1 bestätigt)

**Frontmatter**:
```yaml
---
name: ls-sdd-converge
description: Compare the current codebase against specs/plans/tasks and append missing work as new tasks. Use for brownfield projects or when implementation revealed gaps in the original plan.
---
```

**Inhalt**:
- Prüft Codebase gegen Spec/Plan/Tasks
- Generiert zusätzliche Tasks, hängt sie an `tasks.md` an
- Dokumentiert Abweichungen

## Implementierungs-Schritte

Die Umsetzung erfolgt in dieser Reihenfolge. Jeder Schritt ist unabhängig commit-fähig.

### Phase 1: Fundament

1. **Projektverzeichnis anlegen**: `/home/leonard/Dokumente/Programmieren/2026-06/ls-sdd-skills/` mit `README.md` und `.gitignore`.
2. **Meta-Skill `ls-sdd` schreiben**: Definiert alle Konventionen, Templates, Workflow-Übersicht. Dies ist die Single Source of Truth – alle anderen Skills referenzieren ihn.
3. **Templates als Dateien ablegen**: `ls-sdd/templates/spec.template.md`, `plan.template.md`, `tasks.template.md` – der Meta-Skill referenziert diese.

### Phase 2: Kern-Workflow

4. **`ls-sdd-init` schreiben**: Constitution → `AGENTS.md`.
5. **`ls-sdd-specify` schreiben**: Spec erstellen, scale-adaptive Tiefe (falls A5 bestätigt).
6. **`ls-sdd-clarify` schreiben**: Strukturierte Klärung.
7. **`ls-sdd-create-plan` schreiben**: Plan aus Specs.
8. **`ls-sdd-tasklist` schreiben**: Tasks aus Plan.

### Phase 3: Qualitätssicherung

9. **`ls-sdd-analyze` schreiben**: Konsistenzprüfung.
10. **`ls-sdd-implement` schreiben**: Task-Abarbeitung.

### Phase 4: Iteration (optional)

11. **`ls-sdd-converge` schreiben**: Brownfield/Converge.
12. **Dokumentation**: `README.md` mit Installationsanweisung (`npx skills add`-kompatibel), Beispielen, Workflow-Diagramm.

### Phase 5: Validierung

13. **Self-Test**: Wende die Skills auf sich selbst an – erstelle eine Spec für eine Erweiterung der Skills (z.B. "Web-Bundle für Gemini/ChatGPT"), gehe durch den Workflow.
14. **Review**: Leonard reviewt die Skills in einem echten Projekt.

## Offene Fragen (vom Nutzer zu entscheiden)

1. **Skill-Set**: 9 Skills (mit `ls-sdd-converge`) oder 8 (ohne)?
2. **Scale-adaptive Tiefe (A5)**: Implementieren oder in v1 weglassen?
3. **Sprache der Artefakte (A8)**: Automatisch nach Nutzer-Sprache, oder konfigurierbar?
4. **Task-Format (A6)**: Checkbox mit Metadaten – ok, oder anderes Format?
5. **Status-Modelle (A4)**: So ok, oder Anpassungen?

## Risiken

- **R1: Skill-Kontext-Größe**: Wenn `ls-sdd` alle Konventionen + Templates enthält, wird er groß. Lösung: Templates in separate Dateien auslagern, nur referenzieren.
- **R2: Skill-Discovery**: Zed lädt Skills anhand der `description`. Wenn die Beschreibungen zu ähnlich sind, lädt der Agent mehrere gleichzeitig. Lösung: Beschreibungen klar abgrenzen ("Use *before* creating a plan" vs "Use *after* tasks are generated").
- **R3: Konventions-Drift**: Wenn Skills `ls-sdd` nicht konsistent referenzieren. Lösung: In jedem Skill am Anfang "Lies `ls-sdd` für Konventionen" verankern.
- **R4: ID-Kollisionen**: Wenn manuell angelegte Specs/Plans die ID-Sequenz durcheinanderbringen. Lösung: Skills scannen immer das Verzeichnis und nehmen `max(existing_ids) + 1`.

## Abgrenzung (Out of Scope)

- Keine CLI – die Skills sind reine Markdown-Anweisungen für Zed (und kompatible Agents).
- Keine eigene Skill-Registry – Nutzung von `npx skills` / skills.sh.
- Keine Automatisierung über Zed hinaus (keine GitHub-Action, kein CI-Integration).
- Keine Mehr-Agenten-Orchestrierung (BMAD "Party Mode" etc.).
- Keine UI für den Workflow – der Agent führt durch.
