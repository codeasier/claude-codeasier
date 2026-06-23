# `tasks.md` Template and Guidance

Use `tasks.md` to turn requirements into an implementation plan with progress checkboxes. It should be executable in dependency order.

## Template

```markdown
# Tasks

- [ ] Task 1: <Foundation or discovery task>
  - [ ] SubTask 1.1: <Concrete action>
  - [ ] SubTask 1.2: <Concrete action>

- [ ] Task 2: <Implementation task>
  - [ ] SubTask 2.1: <Concrete action>
  - [ ] SubTask 2.2: <Concrete action>

- [ ] Task 3: <Verification task>
  - [ ] SubTask 3.1: <Run relevant tests, lint, typecheck, build, or manual check>
  - [ ] SubTask 3.2: <Update checklist based on verified outcomes>

# Task Dependencies
- Task 2 depends on Task 1
- Task 3 depends on Task 2
```

## Guidance

- Tasks should be concrete enough for another Agent to execute without re-planning the feature.
- Keep tasks scoped to the requirements in `spec.md`.
- Include verification tasks explicitly when validation is known.
- Use dependencies to clarify ordering and parallelizable work.
- Leave all checkboxes unchecked during spec writing unless the task describes spec authoring work already completed.
- Do not include unrelated cleanup or opportunistic refactors unless the spec requires them.
