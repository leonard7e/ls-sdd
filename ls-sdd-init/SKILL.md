---
name: ls-sdd-init
description: Create or update the project constitution in AGENTS.md. Use at project start, or when foundational principles (tech stack, quality standards, test strategy, conventions) should be established or revised. Run before creating any spec.
---

# ls-sdd-init — Project Constitution

**Conventions**: Read `ls-sdd` (§ Directory Layout through § Templates) for directory structure, frontmatter, status, slug, and ID format. Also read § Research and Delegation; it applies regardless of that reading range. Skip § What's Next? — it is only for direct `ls-sdd` invocation.

## Purpose

Establish a project-wide **constitution** in `AGENTS.md`. The constitution is
the foundational layer that every spec, plan, and task builds on. It captures
non-negotiable principles: tech stack, quality bar, test strategy, naming,
architecture guardrails.

`AGENTS.md` is read automatically by Zed, Claude Code, and Cursor, so the
constitution is always in context for the agent.

## When to Use

- At the start of a new project that will use SDD.
- When the tech stack or quality standards change materially.
- When the existing `AGENTS.md` is missing or stale.

Do **not** use this skill to record feature work — that belongs in a spec.

## Procedure

1. **Check for an existing `AGENTS.md`** in the project root.
   - If it exists, read it fully. Respect what is there; this skill **extends**, it does not overwrite.
2. **Establish purpose and inspect existing project context.** Determine the project's purpose from the user's request or existing project instructions; ask briefly if it is unclear. For a non-empty existing project, delegate a bounded, read-only inventory of applicable project guidance not already read, manifests and versions, directory structure, tests, CI, and evident conventions. Pass known facts from `AGENTS.md` as context so the subagent does not repeat that reading. Return only findings relevant to establishing the constitution. For a genuinely empty greenfield project, skip exploratory codebase search. Check official documentation only when a concrete support or compatibility question needs it.

3. **Interview the user** about the project's foundations. Ask one block of questions, not a long interrogation. Use inventory findings as observed context, not as desired policy. Explicitly distinguish what the repository currently contains from what the user wants the constitution to require. Cover:
   - **Purpose**: Confirm the one- or two-sentence project purpose if it has not already been established.
   - **Tech stack**: Confirm the intended languages, frameworks, runtimes, and key libraries (with versions if they matter); do not assume the observed stack is the desired one.
   - **Artifact language**: The language used in specs, plans, tasks, and other SDD artifacts. This is a **project property**, independent of the conversation language — a German-speaking user may run an English-language project. Record as `Artifact language: <English | German | ...>`.
   - **Quality bar**: What does "done" mean here? Linting, types, review requirements.
   - **Test strategy**: Unit / integration / e2e split, where tests live, coverage expectations.
   - **Conventions**: Naming, file layout, commit message style, branch model.
   - **Architecture guardrails**: Any hard rules (e.g. "no business logic in handlers", "all I/O behind a trait").
   - **Out of scope**: What the project will explicitly not do.
4. **Draft the constitution** in this structure:

   ```markdown
   # AGENTS.md — Project Constitution

   ## Purpose
   ...

   ## Tech Stack
   - ...

   ## Artifact Language
   English   # or German, French, ... — used for specs, plans, tasks, docs

   ## Quality Standards
   - ...

   ## Test Strategy
   - ...

   ## Conventions
   - ...

   ## Architecture Guardrails
   - ...

   ## Out of Scope
   - ...
   ```

5. **If `AGENTS.md` already exists**: merge new content into the existing
   structure. Preserve user-written prose. Add a short changelog note at the
   bottom if the change is significant:

   ```markdown
   ---
   Updated YYYY-MM-DD: added test strategy section.
   ```

6. **Write the file** to `<project-root>/AGENTS.md`.
7. **Confirm to the user** what was added/changed and recommend the next
   step: typically `ls-sdd-specify` to create the first spec.

## Language

→ See `ls-sdd` § Language. The constitution itself is written in the project's artifact language (as determined in step 3).

## What Not to Do

- Do not invent a tech stack the user did not confirm. If unsure, ask.
- Do not duplicate spec content (requirements, acceptance criteria) here — the constitution is about principles, not features.
- Do not delete or rewrite an existing constitution without explicit user consent.
