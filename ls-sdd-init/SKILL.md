---
name: ls-sdd-init
description: Create or update the project constitution in AGENTS.md. Use at project start, or when foundational principles (tech stack, quality standards, test strategy, conventions) should be established or revised. Run before creating any spec.
---

# ls-sdd-init — Project Constitution

**Conventions**: Read the `ls-sdd` meta-skill for directory structure, frontmatter, status, slug, and ID format. This skill does not duplicate them.

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
2. **Interview the user** about the project's foundations. Ask one block of questions, not a long interrogation. Cover:
   - **Purpose**: What is this project? One or two sentences.
   - **Tech stack**: Languages, frameworks, runtimes, key libraries (with versions if it matters).
   - **Artifact language**: The language used in specs, plans, tasks, and other SDD artifacts. This is a **project property**, independent of the conversation language — a German-speaking user may run an English-language project. Record as `Artifact language: <English | German | ...>`.
   - **Quality bar**: What does "done" mean here? Linting, types, review requirements.
   - **Test strategy**: Unit / integration / e2e split, where tests live, coverage expectations.
   - **Conventions**: Naming, file layout, commit message style, branch model.
   - **Architecture guardrails**: Any hard rules (e.g. "no business logic in handlers", "all I/O behind a trait").
   - **Out of scope**: What the project will explicitly not do.
3. **Draft the constitution** in this structure:

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

4. **If `AGENTS.md` already exists**: merge new content into the existing
   structure. Preserve user-written prose. Add a short changelog note at the
   bottom if the change is significant:

   ```markdown
   ---
   Updated YYYY-MM-DD: added test strategy section.
   ```

5. **Write the file** to `<project-root>/AGENTS.md`.
6. **Confirm to the user** what was added/changed and recommend the next
   step: typically `ls-sdd-specify` to create the first spec.

## Language

The constitution itself is written in the **project's artifact language**
(the one declared in the `Artifact Language` section). If the user
communicates in German but the project is English-language, the constitution
is in English. The conversation language only affects how the agent talks to
the user, not what it writes into files.

## What Not to Do

- Do not invent a tech stack the user did not confirm. If unsure, ask.
- Do not duplicate spec content (requirements, acceptance criteria) here — the constitution is about principles, not features.
- Do not delete or rewrite an existing constitution without explicit user consent.
