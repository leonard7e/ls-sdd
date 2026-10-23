---
name: ls-sdd-create-plan
description: Create a technical implementation plan from one or more clarified specifications (or a draft spec when scale is trivial). Use when specs are in status 'clarified' or 'draft' (trivial scale), and the user wants to decide how to build — architecture, modules, dependencies, risks. Produces plan.md and advances specs to 'planned'.
---

# ls-sdd-create-plan — Create an Implementation Plan

**Conventions**: Read `ls-sdd` (§ Directory Layout through § Templates) for directory structure, frontmatter, status, slug, and ID format. Also read § Research and Delegation; it applies regardless of that reading range. Skip § What's Next? — it is only for direct `ls-sdd` invocation.

## Purpose

Translate one or more **clarified specs** into a technical implementation plan:
architecture, modules, dependencies, risks, trade-offs. The plan describes
*how* the spec's *what* will be built. It does **not** contain the task list —
that is `ls-sdd-tasklist`.

A plan can reference **multiple specs** (N:M): e.g. one plan covering spec
`0001` (auth) and spec `0002` (sessions).

## When to Use

- One or more specs are in `clarified` status and the user wants to plan implementation.
- An existing plan needs revision (re-run this skill; the old plan becomes `superseded`).

Do **not** use this skill to write tasks — that is `ls-sdd-tasklist`. Do not
use it to define requirements — that is `ls-sdd-specify`.

## Procedure

1. **Identify the input specs.** If the user did not name them:
   - Scan `specs/` for specs in `clarified` status (or `draft` status if `scale: trivial`).
   - If exactly one, use it. If multiple, ask the user which to plan together.
   - If none, recommend `ls-sdd-clarify` first (or `ls-sdd-specify` if no specs exist).

2. **Read each spec** (`specs/NNNN-spec-<slug>/spec.md`) and its
   `clarifications.md` if present. Read the constitution (`AGENTS.md`) for
   architecture guardrails and tech stack constraints.

3. **Research decision-relevant technical unknowns.** If architecture decisions depend on uncertain codebase facts, delegate bounded inspection of the affected modules, extension points, interfaces, tests, and dependencies. Delegate open API, version, or compatibility questions against official, version-appropriate documentation. Do not perform a broad codebase survey when relevant facts are already established. Use evidenced findings to inform architecture, risks, and existing Research Notes where appropriate; keep unresolved decisions visible and discuss them with the user.

4. **Determine the next plan ID and slug.**
   - Scan `plans/` in the project root.
   - Next ID = `max(existing plan IDs) + 1`, zero-padded to 4 digits. If `plans/` does not exist, start at `0001`.
   - Slug: lowercase kebab-case. Usually matches the primary spec's slug, but can differ if the plan spans multiple specs. Confirm with the user if unclear.
   - Directory: `plans/NNNN-<slug>/`.

5. **Design the plan.** Work through these sections with the user:
   - **Goal**: one-paragraph summary of what the plan achieves.
   - **Architecture**: how the system is structured — modules, layers, key abstractions, data flow. Include a mermaid diagram if it helps.
   - **Modules / Components**: table of modules, their responsibility, and whether they are new or touch existing code.
   - **Dependencies**: external libraries (with version constraints), internal modules, services.
   - **Risks & Trade-offs**: numbered risks with mitigations.
   - **Open questions**: items that need research or a decision before tasks can be written.
   - **Research notes** (optional): if tech-stack research was done, summarize here or write `research.md` in the plan directory.

6. **Read the template** at `ls-sdd/templates/plan.template.md` (relative to the skills root, typically `~/.agents/skills/`). Substitute:
   - `{{NNNN}}` → plan ID
   - `{{identity}}` → slug
   - `{{YYYY-MM-DDTHH:MM}}` → current timestamp
   - `{{Title}}` → human-readable title
   - `{{spec-id}}`, `{{spec-slug}}` → referenced specs
   - Set `specs: [<id1>, <id2>, ...]` in the frontmatter.

7. **Write the file** to `plans/NNNN-<slug>/plan.md`. Create the directory if it does not exist.

8. **Set initial frontmatter**:
   ```yaml
   status: planned
   specs: [<spec-id-1>, <spec-id-2>, ...]
   parent: null   # or the predecessor plan ID if this is a revision
   ```

9. **Update referenced specs**: set each referenced spec's `status` to `planned`.

10. **Confirm to the user**: print the plan path, ID, slug, and referenced
   specs. Recommend the next step: `ls-sdd-tasklist`.

## Trivial Scale

When the input spec has `scale: trivial`, produce a **minimal plan** rather than a full architecture document:

- **Goal**: one paragraph describing what will be done.
- **Modules / Components**: a brief table or bullet list (one or two rows) — enough for `ls-sdd-tasklist` to generate tasks from.
- **Omit**: Architecture diagrams, Risk & Trade-offs, Open questions, Research notes (unless the user explicitly wants them).

The plan file **must still be created** (with `status: planned`) so that `ls-sdd-tasklist` can locate it in `plans/`. Do not skip this step for trivial scale — the minimal plan is what makes the rest of the pipeline work.

After writing the plan, recommend `ls-sdd-tasklist` directly.

## Revision Flow

If a plan already exists for these specs and the user wants to revise:
- Read the old plan.
- Create a new plan with `parent: <old-plan-id>`.
- Set the old plan's `status` to `superseded`.
- Reference the same specs in the new plan's `specs:` field.

## Language

→ See `ls-sdd` § Language. Write all artifacts in the project's artifact language (from `AGENTS.md`).

## What Not to Do

- Do not include the task list in the plan — that is `ls-sdd-tasklist`.
- Do not re-litigate requirements; if the spec is unclear, send the user back to `ls-sdd-clarify`.
- Do not pick a tech stack that contradicts the constitution (`AGENTS.md`). If the constitution is silent, ask the user.
- Do not invent modules the user did not agree to. Propose, then confirm.
