---
description: Execute an approved spec package from .claude/specs and update task/checklist progress.
argument-hint: <change-id>
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, AskUserQuestion
---

# Spec Run

Use this skill to implement an existing spec package exactly as written, maintain progress in the spec files, and verify completion before responding.

## When to Use

Invoke this skill when the user asks to:

- execute an existing spec
- implement a spec package
- run tasks from `.claude/specs/<change-id>/`
- complete a spec-driven change while updating progress

## Arguments

- `$ARGUMENTS` should contain the target `<change-id>` or enough text to identify exactly one package under `.claude/specs/`.
- If `$ARGUMENTS` is empty, inspect `.claude/specs/`, list available packages, and ask the user which one to run.
- If `$ARGUMENTS` matches multiple packages, list the matches and ask the user to choose one.
- If no matching package exists, stop and ask whether to create one with `/spec:write`.

## Required Inputs

The target spec package must live under:

```text
.claude/specs/<change-id>/
```

Read these files before making implementation changes:

```text
spec.md
tasks.md
checklist.md
```

If any required file is missing, stop and ask the user whether to repair the spec package or choose another change-id.

## Workflow

1. Identify the target `<change-id>` from the user request or existing `.claude/specs` entries.
2. Read `spec.md`, `tasks.md`, and `checklist.md`.
3. Confirm the requested execution scope and dependencies from `tasks.md`.
4. Execute tasks in dependency order.
5. After each completed task or coherent task group, update the matching checkbox in `tasks.md`.
6. Run the verification needed for the changed project, such as tests, lint, typecheck, build, or documented manual checks.
7. Update checklist checkboxes only when the corresponding condition is verified.
8. If verification fails, preserve or add a corrective task in `tasks.md`, fix the issue, and re-run verification.
9. Finish with a concise summary of implemented changes, completed tasks, checklist status, and verification results.

## Execution Rules

- Treat `spec.md` as the source of requirements.
- Treat `tasks.md` as the execution plan and progress record.
- Treat `checklist.md` as acceptance verification.
- Do not implement work that is outside the approved spec unless the user explicitly approves a scope change.
- If implementation reveals the spec is incomplete or contradictory, stop and ask for clarification before continuing.
- Keep task and checklist checkboxes accurate; do not mark items complete before verification.
- Preserve incomplete tasks and failed checklist items for follow-up work.

## Failed Verification Handling

When a checklist item or command fails:

1. Leave the related checklist item unchecked.
2. Add or preserve a task in `tasks.md` that describes the corrective work.
3. Implement the correction if it is within spec scope.
4. Re-run the relevant verification.
5. Mark the task and checklist item complete only after the verification passes.

## Final Response Requirements

The final response should include:

- implemented scope
- files changed
- tasks completed or remaining
- checklist items verified or remaining
- verification commands and results
- any follow-up needed
