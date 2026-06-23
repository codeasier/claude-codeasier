# `spec.md` Template and Guidance

Use `spec.md` to describe the requested change as requirements and observable behavior. It is the source of truth for implementation scope.

## Template

```markdown
# <Change Title> Spec

## Why
<Explain the user or business problem, current gap, and desired outcome.>

## What Changes
- <Describe the first planned change.>
- <Describe the second planned change.>

## Impact
- Affected specs: <workflow, feature area, or none>
- Affected code: <expected files, modules, or unknown until implementation>
- Risks: <compatibility, migration, security, performance, or none identified>

## ADDED Requirements
### Requirement: <Requirement Name>
The system SHALL <provide new capability or behavior>.

#### Scenario: <Scenario Name>
- **WHEN** <trigger or condition>
- **THEN** <observable outcome>

## MODIFIED Requirements
### Requirement: <Existing Requirement Name>
The system SHALL <updated behavior>.

#### Scenario: <Updated Scenario Name>
- **WHEN** <trigger or condition>
- **THEN** <updated observable outcome>

## REMOVED Requirements
### Requirement: <Removed Requirement Name>
The system SHALL NOT <old behavior or capability>.

#### Scenario: <Removal Scenario Name>
- **WHEN** <old trigger or condition occurs>
- **THEN** <expected replacement behavior or absence of behavior>
```

## Guidance

- Include only sections that apply to the requested change.
- Prefer `SHALL` statements for requirements so implementation and verification are unambiguous.
- Write scenarios in terms of observable behavior, not internal implementation details.
- Keep scope explicit. If an area is intentionally out of scope, state it in `Impact` or the relevant requirement.
- If information is missing, either ask a targeted question before writing or document assumptions clearly.
