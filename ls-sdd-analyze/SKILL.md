---
name: ls-sdd-analyze
description: Cross-artifact consistency analysis between a spec, its plan, and the task list. Use after tasks are generated (plan status 'tasked'), before implementation, to catch gaps, contradictions, and missing acceptance criteria. Produces analysis.md and advances the plan to 'analyzed' (or back to 'tasked' if issues are found).
---

# ls-sdd-analyze — Consistency Analysis

**Conventions**: Read `ls-sdd` (§ Directory Layout through § Templates) for directory structure, frontmatter, status, slug, and ID format. Also read § Research and Delegation; it applies regardless of that reading range. Skip § What's Next? — it is only for direct `ls-sdd` invocation.

## Purpose

Catch gaps and contradictions **before** implementation, when they are cheap
to fix. The analysis checks that every spec requirement is covered by a task,
that every task traces back to a spec, and that acceptance criteria are
verifiable.

## When to Use

- A plan is in `tasked` status (tasks.md exists) and the user wants a green light to implement.
- After a `tasks.md` revision, to re-validate.

Do **not** use this skill to execute tasks — that is `ls-sdd-implement`. Do
not use it to regenerate tasks — that is `ls-sdd-tasklist`.

## Procedure

1. **Identify the target plan.** If the user did not name one:
   - Scan `plans/` for plans in `tasked` status.
   - If exactly one, use it. If multiple, ask the user which one.
   - If none, recommend `ls-sdd-tasklist` first.

2. **Read all three artifacts**:
   - The plan: `plans/NNNN-<slug>/plan.md`
   - The tasks: `plans/NNNN-<slug>/tasks.md`
   - Each referenced spec: `specs/MMMM-spec-<slug>/spec.md` and `clarifications.md` if present.
   - The constitution: `AGENTS.md` (for quality bar and test-strategy expectations).

3. **Verify only material codebase assumptions.** The core of this skill is consistency among the spec, plan, tasks, and constitution—not a full implementation or drift audit. If a decision-relevant claim depends on a current code fact (for example, whether a referenced path exists or an API boundary is real), delegate a targeted, read-only verification and cite its evidence. Leave unsupported claims explicitly unverified; do not expand this into a general codebase survey.

4. **Run these checks.** For each, record pass/fail with evidence.

   - **C1 — Requirement coverage**: For every numbered requirement and
     acceptance criterion in each spec, find at least one task that
     implements it. List any requirement with no covering task as a **gap**.
   - **C2 — Task traceability**: For every task, identify which spec
     requirement it serves. Tasks with no spec trace are **orphans** — either
     they are scaffolding (acceptable, note it) or they are scope creep
     (flag to the user).
   - **C3 — Acceptance verifiability**: Every task's `acceptance:` criterion
     must be checkable (test passes, file exists, command succeeds, type
     checks). Vague criteria ("works", "done") are **failures**.
   - **C4 — Dependency soundness**: For every `depends: T####`, the depended-on
     task must exist and must logically precede. Cycles are **failures**.
   - **C5 — File conflicts**: Two tasks that list the same file in `files:`
     without a dependency between them are a **conflict** (they may edit
     concurrently and clash).
   - **C6 — Plan/spec alignment**: The plan's `specs:` field must list every
     spec whose requirements appear in the tasks. Missing entries are
     **failures**.
   - **C7 — Constitution alignment**: Tasks must respect the constitution's
     test strategy (e.g. if the constitution requires tests for new modules,
     every task creating a module must include test files in `files:` and a
     test-passing `acceptance:`).

5. **Write the analysis** to `plans/NNNN-<slug>/analysis.md`:

   Read the template at `ls-sdd/templates/analysis.template.md` (relative to the skills root, typically `~/.agents/skills/`). Substitute `{{placeholders}}` with the plan ID, spec IDs, task counts, and results. Fill each `## C#` section with the findings from the checks above.

6. **Update the plan status**:
   - If all checks pass: set `status: analyzed`.
   - If any check fails: set `status: tasked` (back) and list the issues. Do **not** advance to `analyzed`.

7. **Confirm to the user**: print the analysis path, the result (PASS/FAIL),
   and the issue count. If FAIL, recommend the next step:
   - Re-run `ls-sdd-tasklist` to fix task issues, or
   - Re-run `ls-sdd-clarify` if a spec gap was found, or
   - Re-run `ls-sdd-create-plan` if the plan itself is the problem.

   If PASS, recommend `ls-sdd-implement`.

## Language

→ See `ls-sdd` § Language. Write all artifacts in the project's artifact language (from `AGENTS.md`).

## What Not to Do

- Do not auto-fix issues — surface them and let the user decide which skill to re-run.
- Do not mark a plan `analyzed` with open FAIL items.
- Do not skip checks silently; if a check is not applicable (e.g. no constitution), note "N/A" with a reason.
