# AGENTS.md — claude-codeasier

This repository is a **Claude Code plugin marketplace**, not an application. It has no build step and no runtime of its own: it is consumed directly by Claude Code via the marketplace mechanism, and the unit of distribution is a *plugin* registered in `.claude-plugin/marketplace.json`.

Marketplace name: `claude-codeasier` (the shorthand `cce` is documentation-only; installs must use the full name).

## Architecture map

| Path | Role |
|---|---|
| `.claude-plugin/marketplace.json` | Single registration point. Every plugin under `plugins/` **must** have an entry here (`source: ./plugins/<name>`) or it is not installable. |
| `plugins/session/` | The only plugin with executable code: Python scripts, `SessionStart`/`SessionEnd` hooks, and a persistent local state/index. Has its own architecture doc: [plugins/session/AGENTS.md](plugins/session/AGENTS.md). |
| `plugins/dev/` | Skills-only plugin (no code): `env-setup`, `issue-resolve`, `issue-review`, `issue-submit`, `pr-followup`, `worktree-clean`. See [plugins/dev/README.md](plugins/dev/README.md). |
| `plugins/docs/` | Skills-only plugin: `governance` (docs audit/fix workflows). See [plugins/docs/README.md](plugins/docs/README.md). |
| `plugins/release/` | Skills-only plugin: `prep` (PyPI-style release readiness in the *host* project — it references `.github/workflows/publish.yml` etc. in the user's repo, not in this marketplace). See [plugins/release/README.md](plugins/release/README.md). |
| `plugins/understand-me/` | Skills-only plugin: `understand-me` thinking-partner workflow (originally from ClawHub, by `guitenbay`). See [plugins/understand-me/README.md](plugins/understand-me/README.md). |
| `plugins/spec/` | Skills-only plugin: `write` and `run` create/execute spec packages under `.claude/specs/<change-id>/` in the host project. See [plugins/spec/README.md](plugins/spec/README.md). |
| `tests/` | pytest suite for the `session` plugin's scripts only (`testpaths` + `pythonpath` in `pyproject.toml`). No other plugin has tests. |
| `pyproject.toml` | Tool configuration only (ruff, black, pytest, basedpyright). There is no `[build-system]`, no package, and nothing is published to PyPI despite the `name`/`version` fields. |
| `.github/workflows/ci.yml` | Lint (ruff + black), typecheck (basedpyright), tests with `--cov=plugins/session/scripts --cov-fail-under=90`. |
| `.github/workflows/release.yml` | Push a tag `v*` → GitHub Release with a git-log changelog. This is the only release mechanism. |

## Plugin anatomy (invariant across all six plugins)

Every plugin directory follows this layout:

- `plugins/<name>/.claude-plugin/plugin.json` — manifest (`name`, `description`, `author`).
- `plugins/<name>/skills/<skill>/SKILL.md` — prompt definition with YAML frontmatter (`description`, `argument-hint`, `allowed-tools`, often `disable-model-invocation: true` so the skill is slash-command-only).
- `plugins/<name>/README.md` — user-facing usage.

Users invoke skills as `/<plugin>:<skill>` (e.g. `/session:archive`, `/spec:write`). Skills-only plugins contain **no executable code** — their entire behavior is prompt text plus tool permissions in frontmatter. Only `session` additionally ships `hooks/hooks.json` and Python under `scripts/`.

## Adding a plugin — what must change together

1. Create `plugins/<name>/` with `.claude-plugin/plugin.json`, `skills/`, and a `README.md`.
2. Register it in `.claude-plugin/marketplace.json` (`plugins` array, `source: ./plugins/<name>`).
3. Add it to the plugin list in `README.md` (both the "Current plugins" list and the "Repository structure" tree, plus the per-plugin README link list).

Omitting step 2 leaves the plugin unreachable; omitting step 3 leaves the docs inconsistent.

## Build / run / test boundaries

- **Run (consumer)**: `/plugin marketplace add codeasier/claude-codeasier` (remote) or `/plugin marketplace add .` (from this repo), then `/plugin install <name>@claude-codeasier` and `/reload-plugins`.
- **Run (development)**: `claude --plugin-dir ./plugins/<name>` loads a single plugin without the marketplace; `/reload-plugins` picks up edits.
- **Test**: `pytest` — configuration lives in `pyproject.toml` (`testpaths = ["tests"]`, `pythonpath = ["plugins/session/scripts"]`; `tests/conftest.py` also inserts that path). Tests exist only for the `session` plugin. Focus one area: `pytest tests/test_session_hook.py`, or `-k <pattern>`.
- **Lint / format / typecheck** (apply only to the Python in `plugins/session/scripts` and `tests/`, per `[tool.ruff]`, `[tool.black]`, `[tool.basedpyright]` include lists): `ruff check .`, `black --check --diff .`, `basedpyright`. Python target is 3.11+ (`requires-python = ">=3.11"`); CI runs 3.12 and enforces ≥90 % coverage on `plugins/session/scripts`.
- **Do not** look for a build/publish pipeline for Python: none exists. Distribution is the marketplace mechanism; versioning is git tags on this repo.

## Conventions

- Default branch is `main`. PRs use `.github/PULL_REQUEST_TEMPLATE.md` (checklist: ruff/black style, basedPyright annotations, pytest, docs updated).
- Plugin SKILL.md content is mixed English and Chinese (e.g. `dev/env-setup`, `docs/governance`, `release/prep` are Chinese; `spec/*` and `session/*` are English). This file and the root README are English.
- For anything involving session hooks, the session index, or the scripts' state machine, read [plugins/session/AGENTS.md](plugins/session/AGENTS.md) first.
