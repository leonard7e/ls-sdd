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
