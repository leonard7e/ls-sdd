# Prompt: ls-sdd-skills Projekt

## Projekt

Wir bauen eine Sammlung von Zed-Agent-Skills für Spec-Driven Development (SDD). Die Skills sind generisch (nicht an ein spezifisches Projekt gebunden) und werden global unter `~/.agents/skills/` installiert. Präfix `ls-` (Leonard Siebeneicher) zur Wiedererkennung und Namensraum-Trennung.

Vorbild: GitHub Spec Kit (`github/spec-kit`) und BMAD-METHOD, adaptiert ohne deren Komplexität zu übernehmen.

## Skill-Set (9 Skills)

| Skill | Zweck |
|---|---|
| `ls-sdd` | Meta-Skill: Workflow-Übersicht, Konventionen, "was als nächstes?" |
| `ls-sdd-init` | Constitution in `AGENTS.md` erstellen/aktualisieren |
| `ls-sdd-specify` | Spec erstellen (Was & Warum, nicht Wie) |
| `ls-sdd-clarify` | Strukturierte Klärung vor dem Planen (Lücken aufdecken) |
| `ls-sdd-create-plan` | Plan aus einer oder mehreren Specs (N:M) |
| `ls-sdd-tasklist` | Tasks aus Plan erstellen (mit Abhängigkeiten) |
| `ls-sdd-analyze` | Konsistenzprüfung Spec↔Plan↔Tasks |
| `ls-sdd-implement` | Tasks abarbeiten |
| `ls-sdd-converge` | Codebase gegen Specs prüfen, Lücken als neue Tasks |

## Verzeichnisstruktur in Zielprojekten

```
[projekt]/
├── AGENTS.md                    # Constitution (projektweit)
├── plans/
│   └── 001-[identity]/
│       ├── plan.md
│       ├── tasks.md
│       └── [research.md, analysis.md]   # optional
└── specs/
    ├── 001-spec-[identity]/
    │   ├── spec.md
    │   ├── clarifications.md
    │   └── [additional markdown files]
    └── 002-spec-[identity]/
        └── spec.md
```

**Konventionen**:
- `AGENTS.md` enthält die Constitution – wird von Zed/Claude Code/Cursor automatisch gelesen.
- Plans und Specs haben **getrennte** ID-Sequenzen (Plan `001` und Spec `001` können beide existieren).
- Plan-Slug: `001-identity` (kein `-plan-` Marker).
- Spec-Slug: `001-spec-identity` (mit `-spec-` Marker zur Unterscheidung).
- Plans und Specs sind N:M gekoppelt: Plan referenziert Specs im Frontmatter.
- Zusätzliche Markdown-Dateien pro Spec erlaubt (z.B. `menu.md`, `data-model.md`).

## Frontmatter-Modelle

**Spec** (`specs/001-spec-x/spec.md`):
```yaml
---
id: 0001
slug: identity
created: 2026-07-01T14:30
status: draft          # draft | clarified | planned | superseded | done
scale: standard        # trivial | standard | enterprise
parent: null
---
```

**Plan** (`plans/001-identity/plan.md`):
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

**Tasks** (`plans/001-identity/tasks.md`):
```yaml
---
id: 0001
plan: 0001
created: 2026-07-01T14:30
status: draft          # draft | in-progress | blocked | done
---
```

## Status-Modelle

- **Spec**: `draft → clarified → planned → done` (mit `superseded` als Terminal)
- **Plan**: `draft → planned → tasked → analyzed → implementing → done` (mit `superseded`)
- **Task**: `- [ ]` offen / `- [x]` erledigt / `- [~]` blockiert

## Task-Format

Checkbox-Liste mit Metadaten (parsbar vom Agent):

```markdown
- [ ] T0001 Provider-Trait definieren
  - depends: -
  - files: src/provider/mod.rs
  - acceptance: Trait mit complete(), stream()
```

Parallele Tasks mit `[P]` markieren.

## Skill-Design-Prinzipien

- **Single Source of Truth**: `ls-sdd` definiert alle Konventionen (Verzeichnisstruktur, Frontmatter, Status, Slug-Format, ID-Format). Die anderen Skills referenzieren ihn: "Lies `ls-sdd` für Format-Details." Keine Konventions-Duplizierung.
- **Templates als Dateien**: `ls-sdd/templates/{spec,plan,tasks}.template.md` – der Meta-Skill referenziert diese statt alles inline zu halten.
- **Klare Skill-Discovery**: Beschreibungen im Frontmatter so abgegrenzt, dass der Agent nicht mehrere Skills gleichzeitig lädt ("Use *before* creating a plan" vs "Use *after* tasks are generated").
- **ID-Vergabe**: Skills scannen das Zielverzeichnis und nehmen `max(existing_ids) + 1`.

## Sprache

- Skill-Bodies (SKILL.md): **Englisch** (Standard im Skills-Ökosystem, bessere Auffindbarkeit über `npx skills find`)
- Generierte Artefakte (specs, plans, tasks): **Sprache des Nutzers** (Deutsch, wenn der Nutzer deutsch kommuniziert)

## Skill-Scope

Global unter `~/.agents/skills/`. SDD ist projektunabhängig.

## Implementierungs-Reihenfolge

1. Projektverzeichnis anlegen mit `README.md` und `.gitignore`
2. Meta-Skill `ls-sdd` schreiben (Konventionen, Templates, Workflow-Übersicht)
3. Templates als Dateien ablegen
4. `ls-sdd-init` (Constitution → `AGENTS.md`)
5. `ls-sdd-specify` (Spec erstellen)
6. `ls-sdd-clarify` (Klärung)
7. `ls-sdd-create-plan` (Plan aus Specs)
8. `ls-sdd-tasklist` (Tasks aus Plan)
9. `ls-sdd-analyze` (Konsistenzprüfung)
10. `ls-sdd-implement` (Task-Abarbeitung)
11. `ls-sdd-converge` (Brownfield/Converge)
12. `README.md` mit Installationsanweisung (`npx skills add`-kompatibel)

## Abgrenzung (Out of Scope)

- Keine CLI – Skills sind reine Markdown-Anweisungen für Zed (und kompatible Agents)
- Keine eigene Skill-Registry – Nutzung von `npx skills` / skills.sh
- Keine Automatisierung über Zed hinaus (keine GitHub-Action, kein CI)
- Keine Mehr-Agenten-Orchestrierung (BMAD "Party Mode" etc.)
- Keine UI für den Workflow – der Agent führt durch

## Offene Fragen (vor Implementierung klären)

1. **Scale-adaptive Tiefe**: Soll `ls-sdd-specify` nach Projektgröße fragen (trivial/standard/enterprise)? Trivial = minimale Spec, direkte Tasklist; Standard = volle Spec→Plan→Tasks; Enterprise = Spec mit nicht-funktionalen Anforderungen, Architektur-Plan.
2. **Sprache der Artefakte**: Automatisch nach Nutzer-Sprache folgen, oder konfigurierbar machen?

## Referenzen

- GitHub Spec Kit: https://github.com/github/spec-kit
- BMAD-METHOD: https://github.com/bmad-code-org/BMAD-METHOD
- Zed Skill Format: SKILL.md mit YAML-Frontmatter (`name`, `description`), Body mit Anweisungen. Supporting files im Skill-Verzeichnis möglich.
