---
id: {{NNNN}}
plan: {{plan-id}}
created: {{YYYY-MM-DDTHH:MM}}
status: draft
---

# Tasks: {{Plan Title}}

> Tasks are an ordered, dependency-aware checklist derived from the plan.
> Each task has an ID (`T####`), optional dependencies, target files, and
> acceptance criteria. The agent parses this file to track progress.

## Task List

- [ ] T0001 {{Task title}}
  - depends: -
  - files: src/...
  - acceptance: ...

- [ ] T0002 {{Task title}}
  - depends: T0001
  - files: src/...
  - acceptance: ...

- [P] T0003 {{Parallelizable task title}}
  - depends: T0001
  - files: src/...
  - acceptance: ...

## Conventions

- `- [ ]` open · `- [x]` done · `- [~]` blocked
- `[P]` after the checkbox marks a task as parallelizable.
- `depends:` lists task IDs that must be done first, or `-` for none.
- `files:` lists the files this task will create or modify.
- `acceptance:` states the verifiable condition for "done".
