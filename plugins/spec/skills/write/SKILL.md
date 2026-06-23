---
description: Create spec-driven development documents under .claude/specs without implementing code.
argument-hint: <change description>
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion
---

# Spec Write

Use this skill to guide spec-driven planning before implementation. The goal is to create a complete spec package and stop before code changes.

## When to Use

Invoke this skill when the user asks to:

- write a spec
- create spec documents
- plan a feature through a spec workflow
- turn a requested change into `spec.md`, `tasks.md`, and `checklist.md`

## Arguments

- `$ARGUMENTS` is the requested change description.
- If `$ARGUMENTS` is empty, ask the user what change should be specified before writing files.
- If the request includes an explicit `<change-id>`, use it when it is concise kebab-case.
- Otherwise, create a concise kebab-case `<change-id>` from the requested change.
- If `.claude/specs/<change-id>/` already exists, update that package instead of creating a duplicate.

## Output Location

Specs live under:

```text
.claude/specs/<change-id>/
```

Each spec package must contain:

```text
spec.md
tasks.md
checklist.md
```

## References

Reusable templates and explanations live under this skill's relative reference directory:

```text
references/
```

Use these reference files when generating a spec package:

- `references/spec.md`: requirements and observable behavior template
- `references/tasks.md`: implementation plan and dependency template
- `references/checklist.md`: acceptance verification template

The references are examples for authoring `.claude/specs/<change-id>/spec.md`, `.claude/specs/<change-id>/tasks.md`, and `.claude/specs/<change-id>/checklist.md`; do not copy irrelevant sections into a generated spec package.

## Workflow

1. Understand the requested change.
2. Inspect existing `.claude/specs` packages to avoid duplicating an active spec.
3. Select an existing matching `<change-id>` or create a concise kebab-case `<change-id>` that describes the change.
4. Create or update only the allowed files in `.claude/specs/<change-id>/`:
   - `spec.md`
   - `tasks.md`
   - `checklist.md`
5. Keep the spec package implementation-ready, but do not implement it.
6. Ask for approval or next-step confirmation after writing the spec package.

## `spec.md` Requirements

`spec.md` should include:

- title
- why the change is needed
- what changes
- expected impact
- added, modified, or removed requirements
- scenarios that describe observable behavior

Recommended structure:

```markdown
# <Change Title> Spec

## Why

## What Changes

## Impact

## ADDED Requirements
### Requirement: <Requirement Name>
#### Scenario: <Scenario Name>
- **WHEN** ...
- **THEN** ...
```

Use `MODIFIED Requirements` or `REMOVED Requirements` when appropriate.

## `tasks.md` Requirements

`tasks.md` should include checkbox tasks and subtasks that can be executed in dependency order.

Recommended structure:

```markdown
# Tasks

- [ ] Task 1: <Task Name>
  - [ ] SubTask 1.1: <Subtask>

# Task Dependencies
- Task 2 depends on Task 1
```

Tasks should be concrete, verifiable, and scoped to the approved requirements.

## `checklist.md` Requirements

`checklist.md` should include acceptance checks that prove the spec was completed.

Recommended structure:

```markdown
# Checklist

- [ ] <Expected artifact or behavior exists>
- [ ] <Requirement is satisfied>
- [ ] <Verification command or manual check passes>
```

Checklist items should map back to requirements and critical tasks.

## Guardrails

- Do not implement code while using this skill.
- Do not edit files outside `.claude/specs/<change-id>/` unless the user explicitly asks.
- Do not mark implementation tasks complete during spec writing.
- Do not invent approval; finish by telling the user where the spec package is and ask whether to execute it.
- If requirements are ambiguous, ask targeted clarification questions before writing or clearly state assumptions in `spec.md`.
