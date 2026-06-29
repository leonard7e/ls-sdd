---
name: ls-sdd-specify
description: Create a functional specification describing what to build and why, before any technical planning. Use when the user wants to define requirements, user stories, and acceptance criteria for a feature or change. Does not cover implementation — that is the plan's job.
---

# ls-sdd-specify — Create a Specification

**Conventions**: Read the `ls-sdd` meta-skill for directory structure, frontmatter, status, slug, and ID format. This skill does not duplicate them.

## Purpose

Produce a **functional specification** — a document that describes *what* to
build and *why*, deliberately avoiding *how*. The spec is the input to
`ls-sdd-clarify`, `ls-sdd-create-plan`, and downstream skills.

## When to Use

- The user wants to define a feature, change, or capability before planning.
- A spec is in `draft` and the user wants to refine it (re-run this skill, or
  use `ls-sdd-clarify` for gap-finding).

Do **not** use this skill to write the plan (architecture, modules) — that is
`ls-sdd-create-plan`.

## Procedure

1. **Determine the scale.** Ask the user (once, briefly):
   - **trivial** — bugfix or small tweak. Minimal spec, skip the plan later, go straight to tasks.
   - **standard** — feature or new module. Full spec → plan → tasks.
   - **enterprise** — system spanning multiple modules/teams. Spec with non-functional requirements, architecture plan, dependency-aware tasks.

   Record the answer in the spec frontmatter as `scale: trivial | standard | enterprise`.

2. **Determine the next ID and slug.**
   - Scan `specs/` in the project root.
   - Next ID = `max(existing spec IDs) + 1`, zero-padded to 4 digits (`0001`, `0002`, ...). If `specs/` does not exist, start at `0001`.
   - Slug: lowercase kebab-case derived from the feature name (e.g. `auth-login`, `pdf-export`). Confirm with the user if unclear.
   - Directory: `specs/NNNN-spec-<slug>/`.

3. **Interview the user** about the spec content. Ask in one structured block, not a long interrogation:
   - **Context**: current situation, why this matters now.
   - **Problem**: what problem is being solved, cost of not solving it.
   - **Requirements**: numbered, testable statements of behavior.
   - **User stories** (optional): *As a `<actor>`, I want `<capability>`, so that `<benefit>`.*
   - **Acceptance criteria**: each requirement maps to one or more verifiable criteria.
   - **Out of scope**: what this work explicitly does not cover.
   - **Non-functional requirements** (only for `scale: enterprise` or when explicitly relevant): performance, security, scalability, observability.
   - **Open questions**: unresolved items that `ls-sdd-clarify` will tackle.

4. **Read the template** at `~/.agents/skills/ls-sdd/templates/spec.template.md` (or the equivalent path in the installed skill). Substitute the `{{placeholders}}`:
   - `{{NNNN}}` → the spec ID
   - `{{identity}}` → the slug
   - `{{YYYY-MM-DDTHH:MM}}` → current timestamp
   - `{{Title}}` → human-readable title

5. **Fill the body** with the user's answers. Keep it about *what* and *why*.
   If the user starts describing implementation, gently move that to the plan
   phase ("we'll capture that in the plan").

6. **Write the file** to `specs/NNNN-spec-<slug>/spec.md`. Create the directory if it does not exist.

7. **Set initial frontmatter**:
   ```yaml
   status: draft
   scale: <trivial | standard | enterprise>
   parent: null
   ```

8. **Confirm to the user**: print the spec path, ID, and slug, and recommend
   the next step:
   - For `trivial` scale: recommend `ls-sdd-tasklist` directly (skip plan).
   - For `standard` / `enterprise`: recommend `ls-sdd-clarify` to surface gaps before planning.

## Language

Write the spec in the **project's artifact language** — read it from the
`Artifact Language` section of `AGENTS.md`. If `AGENTS.md` is silent and no
existing artifacts exist, ask the user once and recommend running
`ls-sdd-init` to record the choice. The conversation language does not
determine the artifact language.

## What Not to Do

- Do not include architecture, module breakdown, or file paths in the spec — that belongs in the plan.
- Do not skip the `scale` question; it shapes downstream skills.
- Do not invent requirements the user did not confirm. Mark inferred items as open questions instead.
- Do not duplicate the constitution (`AGENTS.md`). The spec assumes it as context.
