---
name: ls-sdd-converge
description: Compare the current codebase against specs/plans/tasks and append missing work as new tasks. Use for brownfield projects adopting SDD, or when implementation revealed gaps in the original plan. Produces appended tasks in tasks.md and a convergence report.
---

# ls-sdd-converge — Codebase ↔ Specs Convergence

**Conventions**: Read `ls-sdd` (§ Directory Layout through § Templates) for directory structure, frontmatter, status, slug, and ID format. Also read § Research and Delegation; it applies regardless of that reading range. Skip § What's Next? — it is only for direct `ls-sdd` invocation.

## Purpose

Reconcile the **actual codebase** with the **spec/plan/task** artifacts. Two
typical situations:

1. **Brownfield adoption**: a project already has code but no SDD artifacts.
   `ls-sdd-converge` reverse-engineers specs from the code and produces a
   task list to bring the artifacts in sync.
2. **Drift after implementation**: a plan was `done` but the codebase has
   since changed (manual fixes, hotfixes, new features) and the artifacts no
   longer match. `ls-sdd-converge` appends missing tasks to close the gap.

## When to Use

- A brownfield project wants to adopt SDD.
- A plan is in `done` (or `implementing`) but the codebase has drifted.
- The user asks "what's missing?" or "is the code in sync with the specs?".

Do **not** use this skill to execute tasks — that is `ls-sdd-implement`. Do
not use it to write the original plan — that is `ls-sdd-create-plan`.

## Procedure

### Mode A — Brownfield Adoption

1. Read `AGENTS.md` if it exists; if not, recommend `ls-sdd-init` first. Ask the user which capabilities to cover before proceeding. Start with at most 3–5 modules or capabilities; do not attempt to reverse-engineer the entire codebase at once.
2. **Delegate bounded codebase research packages** for the selected capabilities. Each package should identify relevant modules, responsibilities, user-facing behavior, and code or test evidence. Keep requests read-only and as non-overlapping as practical. The main session consolidates the findings and drafts the artifacts; research subagents do not create or modify them.
3. **Reverse-engineer specs**: for each major capability, draft a spec
   (`specs/NNNN-spec-<slug>/spec.md`) capturing *what* it does today (not
   what it should do). Set `status: planned` (skip `draft`/`clarified` —
   the code is the source of truth here).
4. **Reverse-engineer a plan**: create `plans/NNNN-<slug>/plan.md`
   describing the current architecture as-is. Set `status: tasked`.
5. **Generate tasks**: create `plans/NNNN-<slug>/tasks.md` with tasks for
   the *gaps* — places where the code does not match the (reverse-engineered)
   spec, or where quality bar (tests, types) is not met. Mark all
   already-done work as `- [x]` with `acceptance: already in place`.
6. **Write a convergence report** to `plans/NNNN-<slug>/convergence.md`
   (see format below).
7. **Recommend** `ls-sdd-analyze` then `ls-sdd-implement`.

### Mode B — Drift After Implementation

1. **Identify the target plan.** If the user did not name one:
   - Scan `plans/` for plans in `done` or `implementing` status.
   - If exactly one, use it. If multiple, ask the user which one.
   - If none, recommend Mode A (brownfield) or `ls-sdd-specify`.

2. **Read the existing artifacts**: plan, tasks, referenced specs, and the
   constitution.

3. **Compare code and tests using bounded research packages.** Delegate independent, non-overlapping checks of the files referenced in `tasks.md` and the plan's module list. Ask each subagent to return findings mapped as `requirement/task → code or test evidence → discrepancy/uncertainty`, with paths and relevant sections or lines. Keep the scope to:
   - **Missing work**: spec requirements that have no corresponding code.
   - **Drifted work**: code that exists but does not match the spec (different behavior, missing edge-case handling, relaxed validation).
   - **Extra work**: code that exists but is not in any spec — either scope creep or undocumented features.
   - **Broken acceptance criteria**: tasks marked `- [x]` whose acceptance criterion no longer holds (e.g. a test that was deleted or now fails).

   The main session consolidates overlapping findings and decides whether to append tasks or recommend `ls-sdd-specify` for undocumented behavior. Do not turn uncertain or undocumented behavior into implementation tasks without that assessment.

4. **Append new tasks** to the existing `tasks.md`:
   - Use the next free task IDs (scan existing IDs, take `max + 1`).
   - Mark each new task `- [ ]` with `depends:`, `files:`, and `acceptance:`.
   - Add a section header to separate convergence tasks:

     ```markdown
     ## Convergence Tasks (added YYYY-MM-DD)

     - [ ] T0042 Add missing input validation for spec 0001 R3
       - depends: -
       - files: src/auth/validate.rs
       - acceptance: invalid emails rejected with 400
     ```

5. **Update the plan status** back to `implementing` (it was `done` but now
   has new work). If the plan was already `implementing`, leave it.

6. **Write a convergence report** to `plans/NNNN-<slug>/convergence.md`:

   Read the template at `ls-sdd/templates/convergence.template.md` (relative to the skills root, typically `~/.agents/skills/`). Substitute `{{placeholders}}`. Omit sections with no findings (write "none" rather than omitting entirely — it proves the check ran).

7. **Confirm to the user**: print the convergence report path, the new task
   count, and recommend the next step:
   - If new specs are needed: `ls-sdd-specify`.
   - If tasks need re-validation: `ls-sdd-analyze`.
   - Otherwise: `ls-sdd-implement` to execute the new tasks.

## Language

→ See `ls-sdd` § Language. Write all artifacts in the project's artifact language (from `AGENTS.md`).

## What Not to Do

- Do not silently accept undocumented features as "in scope" — surface the evidence and let the main session recommend a new spec instead.
- Do not delete or renumber existing tasks — append only.
- Do not mark drifted tasks `- [~]` automatically; surface them and let the user decide.
- Do not skip the convergence report — it is the audit trail of what was found and what was added.
