# `checklist.md` Template and Guidance

Use `checklist.md` to define acceptance checks that prove the implemented change satisfies the spec.

## Template

```markdown
# Checklist

- [ ] Requirement `<Requirement Name>` is implemented.
- [ ] Scenario `<Scenario Name>` produces the expected observable behavior.
- [ ] Required files or artifacts exist in the expected paths.
- [ ] Relevant tests pass: `<command or manual check>`.
- [ ] Relevant lint, typecheck, build, or validation commands pass: `<command>`.
- [ ] No out-of-scope files or behaviors were changed.
```

## Guidance

- Each checklist item should be independently verifiable.
- Map checklist items back to important requirements and scenarios from `spec.md`.
- Prefer observable outcomes over implementation details.
- Include path checks when the spec requires files to exist at exact locations.
- Include command-based verification when a project has known validation commands.
- Keep checklist items unchecked until the implementation phase verifies them.
