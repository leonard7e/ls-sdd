---
name: ls-sdd-tasklist
description: Generate an actionable, dependency-aware task list from an implementation plan. Use after a plan exists in status 'planned' (or 'tasked' for revision), to break it into ordered tasks with IDs, dependencies, target files, and acceptance criteria. Produces tasks.md and advances the plan to 'tasked'.
---

# ls-sdd-tasklist — Generate Tasks from a Plan

**Conventions**: Read the `ls-sdd` meta-skill for directory structure, frontmatter, status, slug, and ID format. This skill does not duplicate them.

## Purpose

Turn a plan into an **ordered, dependency-aware checklist** that
`ls-sdd-implement` can execute. Each task is small enough to be done in one
sitting, has a verifiable acceptance criterion, and declares which files it
touches and which tasks must be done first.

## When to Use

- A plan is in `planned` status and the user wants to start implementation soon.
- An existing `tasks.md` needs regeneration after a plan revision.

Do **not** use this skill to design architecture — that is
`ls-sdd-create-plan`. Do not use it to execute tasks — that is
`ls-sdd-implement`.

## Procedure

1. **Identify the target plan.** If the user did not name one:
   - Scan `plans/` for plans in `planned` status.
   - If exactly one, use it. If multiple, ask the user which one.
   - If none, recommend `ls-sdd-create-plan` first.

2. **Read the plan** (`plans/NNNN-<slug>/plan.md`) and its frontmatter.
   Read the referenced specs for acceptance criteria. Read the constitution
   (`AGENTS.md`) for test-strategy and quality-bar expectations — every task
   should produce tests where the constitution requires them.

3. **Decompose into tasks.** Walk the plan's modules and components and
   break them into tasks following these rules:
   - **Small**: each task is one coherent unit of work, doable in one sitting.
   - **Verifiable**: each task has an `acceptance:` criterion that can be checked (test passes, file exists, type checks, command succeeds).
   - **File-scoped**: each task lists the files it creates or modifies under `files:`.
   - **Dependency-ordered**: if task B needs task A's output, set `depends: T000A` on B.
   - **Parallelizable**: if a task can run alongside others once its `depends:` are satisfied, mark it `[P]`.

4. **Number tasks** `T0001`, `T0002`, ... in roughly the order they should be
   executed. Dependencies may force reordering at execution time; that is fine.

5. **Read the template** at `~/.agents/skills/ls-sdd/templates/tasks.template.md`. Substitute:
   - `{{NNNN}}` → the tasks file ID (use the plan ID for simplicity, or scan `tasks.md` files if you want a separate sequence — the convention is one `tasks.md` per plan, so the ID equals the plan ID)
   - `{{plan-id}}` → the plan ID
   - `{{YYYY-MM-DDTHH:MM}}` → current timestamp
   - `{{Plan Title}}` → the plan's title

6. **Write the file** to `plans/NNNN-<slug>/tasks.md`. Create the directory if it does not exist (it should already exist from `ls-sdd-create-plan`).

7. **Set frontmatter**:
   ```yaml
   id: <plan-id>      # one tasks.md per plan; ID mirrors the plan ID
   plan: <plan-id>
   status: draft
   ```

8. **Update the plan frontmatter**: set `status: tasked`.

9. **Confirm to the user**: print the tasks path, the task count, and a
   short summary of the dependency graph (e.g. "T0001 → T0002 → T0003;
   T0004 parallel to T0002"). Recommend the next step: `ls-sdd-analyze`.

## Task Format (reminder)

```markdown
- [ ] T0001 Provider-Trait definieren
  - depends: -
  - files: src/provider/mod.rs
  - acceptance: Trait mit complete(), stream()

- [P] T0002 OpenAI-Provider implementieren
  - depends: T0001
  - files: src/provider/openai.rs
  - acceptance: OpenAI-Provider implementiert das Trait, Tests grün
```

- `- [ ]` open · `- [x]` done · `- [~]` blocked
- `[P]` after the checkbox marks a parallelizable task.
- `depends:` lists task IDs that must be done first, or `-` for none.
- `files:` lists the files this task will create or modify.
- `acceptance:` states the verifiable condition for "done".

## Revision Flow

If `tasks.md` already exists and the user wants to regenerate:
- Read the old `tasks.md`.
- Preserve the IDs of tasks that are already `- [x]` done — do not renumber them.
- Append new tasks with the next free IDs.
- If a done task is now obsolete, mark it `- [x]` with a note `obsolete: <reason>` rather than deleting it (preserves history).

## Language

Write `tasks.md` in the **project's artifact language** — read it from the
`Artifact Language` section of `AGENTS.md`. Match the language of the plan
it derives from. The conversation language does not determine the artifact
language.

## What Not to Do

- Do not write tasks that lack an `acceptance:` criterion — unverifiable tasks cannot be marked done safely.
- Do not create tasks without `files:` — `ls-sdd-implement` uses this to know where to work.
- Do not invent dependencies; only set `depends:` when task B truly cannot start before task A.
- Do not skip the `[P]` marker on parallelizable tasks — it lets future multi-agent execution parallelize safely.
