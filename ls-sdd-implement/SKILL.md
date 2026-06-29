---
name: ls-sdd-implement
description: Execute tasks from a task list in dependency order. Use when the plan is in status 'analyzed' (or 'implementing') and the user wants to start or continue implementation. Respects task dependencies, marks tasks done as they complete, and updates the plan status to 'done' when all tasks are finished.
---

# ls-sdd-implement — Execute Tasks

**Conventions**: Read the `ls-sdd` meta-skill for directory structure, frontmatter, status, slug, and ID format. This skill does not duplicate them.

## Purpose

Execute the task list produced by `ls-sdd-tasklist` and validated by
`ls-sdd-analyze`. The agent works through tasks in dependency order, marks
each `- [x]` when its acceptance criterion is met, and updates the plan
status as implementation progresses.

## When to Use

- A plan is in `analyzed` status and the user wants to start coding.
- A plan is in `implementing` status and the user wants to continue.

Do **not** use this skill to add new tasks — that is `ls-sdd-tasklist` or
`ls-sdd-converge`. Do not use it to re-validate — that is `ls-sdd-analyze`.

## Procedure

1. **Identify the target plan.** If the user did not name one:
   - Scan `plans/` for plans in `analyzed` or `implementing` status.
   - If exactly one, use it. If multiple, ask the user which one.
   - If none in those statuses, recommend `ls-sdd-analyze` first.

2. **Read the task list** (`plans/NNNN-<slug>/tasks.md`) and the plan
   (`plans/NNNN-<slug>/plan.md`). Read the referenced specs and the
   constitution (`AGENTS.md`) for context.

3. **Set the plan status** to `implementing` if it is not already.

4. **Pick the next task(s) to work on**:
   - A task is **ready** if it is `- [ ]` (open) and all tasks in its `depends:` are `- [x]` (done).
   - Tasks marked `[P]` (parallelizable) can be picked together if the agent supports parallel work; otherwise pick them in order.
   - If a task is `- [~]` (blocked), do not pick it — surface the blocker to the user.

5. **For each picked task**:
   - Read the `files:` it touches. If a file does not exist yet, create it. If it exists, read it before editing.
   - Implement the work described by the task title and the plan's module design.
   - Verify the `acceptance:` criterion. Run tests, type checks, linters, or whatever the criterion specifies. **Do not mark the task done unless the acceptance criterion actually passes.**
   - If the acceptance criterion cannot be met (e.g. a test fails and the root cause is unclear), stop, leave the task `- [ ]`, and report the blocker to the user. Optionally mark it `- [~]` with a note.
   - If the acceptance criterion passes, mark the task `- [x]` in `tasks.md`.

6. **Update `tasks.md`** after each task completion (or blocker). The file is
   the source of truth for progress — keep it current.

7. **When all tasks are `- [x]`**:
   - Set the plan `status: done`.
   - Set the referenced specs' `status: done`.
   - Run a final check: tests, type checks, linters as required by the constitution.
   - Confirm to the user: print the plan path, the task count, and recommend the next step (typically `ls-sdd-converge` for brownfield review, or commit the work).

## Resuming

If the agent is resuming an `implementing` plan:
- Read `tasks.md` to see which tasks are `- [x]` and which are `- [ ]`.
- Resume from the next ready task.
- Do not redo done tasks unless their acceptance criterion is now broken.

## Blocked Tasks

If a task cannot proceed:
- Mark it `- [~]`.
- Add a note line under it: `- blocked: <reason>`.
- Surface the blocker to the user and recommend either fixing the root cause or re-running `ls-sdd-tasklist` to restructure.

## Language

Communicate with the user in their language. Code, commit messages, and
artifacts follow the **project's artifact language** — read it from the
`Artifact Language` section of `AGENTS.md`. The conversation language does
not determine the artifact language.

## What Not to Do

- Do not mark a task `- [x]` without verifying its acceptance criterion.
- Do not skip the dependency check — picking a task whose `depends:` are not done leads to rework.
- Do not edit `tasks.md` to lower the acceptance bar; if a criterion is wrong, re-run `ls-sdd-tasklist`.
- Do not implement features not in the task list — that is scope creep. Propose new tasks via `ls-sdd-converge` instead.
