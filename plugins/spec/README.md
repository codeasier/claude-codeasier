# spec

Spec-driven development workflow for writing and executing approved spec packages.

## Included skills

- `/spec:write <change description>`
- `/spec:run <change-id>`

## Workflow

1. Run `/spec:write <change description>` to create or update a spec package under `.claude/specs/<change-id>/`.
2. Review the generated `spec.md`, `tasks.md`, and `checklist.md` files.
3. Run `/spec:run <change-id>` after approving the spec package.

## Spec package layout

```text
.claude/specs/<change-id>/
├── spec.md
├── tasks.md
└── checklist.md
```

## Local development

```bash
claude --plugin-dir ./plugins/spec
```

## Install from local marketplace

```text
/plugin marketplace add .
/plugin install spec@claude-codeasier
/reload-plugins
```
