# AGENTS.md — plugins/session

Scope: the `session` plugin of the `claude-codeasier` marketplace (parent architecture: [../../AGENTS.md](../../AGENTS.md)). It is the only plugin in this repository with executable code, a runtime lifecycle (Claude Code hooks), a persistent data model, and a dedicated test target (the repo-root [../../tests/](../../tests/) suite).

## What it does

Tracks Claude Code sessions via `SessionStart`/`SessionEnd` hooks, maintains a local session index, and archives / trashes / purges session transcripts on demand or deferred to session end.

## Entry points (three, all converging on the same scripts)

1. **Hook entry** — `hooks/hooks.json` registers `SessionStart` and `SessionEnd` matchers (`*`, 10 s timeout) that run `python3 ${CLAUDE_PLUGIN_ROOT}/scripts/session_hook.py on-start|on-end`. The hook payload arrives as JSON on **stdin** (`session_id`, `transcript_path`, `cwd`, `source`, `model`); missing required fields are a no-op with exit code 0.
2. **CLI entry** — `scripts/cc_session.py` with subcommands `archive [session-id|cancel]`, `delete [session-id|cancel] [--mode trash|purge]`, and `setup (archive-dir|trash-dir|show|reset)`.
3. **Skill entry** — the four skills under `skills/` (`archive`, `delete`, `review`, `setup`) are thin wrappers that shell out to `python3 ${CLAUDE_SKILL_DIR}/../../scripts/cc_session.py ...`. All are `disable-model-invocation: true` (slash-command only) and their frontmatter `allowed-tools` is scoped to exactly that command pattern. `skills/review/` is different: it is a prompt skill (no script call) that reads `shared.sop` plus `troubleshoot.sop`/`summary.sop` and resolves transcripts via the index, falling back to `~/.claude/projects/`.

## Module map (`scripts/`, flat imports — `pythonpath`-style, no package)

| File | Responsibility |
|---|---|
| `config.py` | State dir `~/.claude/plugins/session/state/`, `config.json` load/save with defaults merged in: `archiveDir`, `trashDir` (templates containing `${project_slug}`), `defaultDeleteMode = trash`, `archiveCurrentSessionMode = copy`. |
| `index_store.py` | The data model. `session-index.json` (`{"version": 1, "sessions": {id → record}}`), `fcntl.flock` file lock (`session-index.lock`), atomic saves (tempfile + `os.fsync` + rename + dir fsync), quarantine of corrupt JSON to `session-index.json.corrupt-<ts>`, and `reconcile_stale_sessions` (marks sessions left `active` as `ended`). |
| `path_utils.py` | `project_slug` (absolute path with `/` → `-`), `${var}` template expansion, `safe_session_filename` (`<started_at>__<session_id>.jsonl`), `dedupe_destination` (`__1`, `__2` suffixes). |
| `session_hook.py` | Thin dispatcher: `on-start` → reconcile + upsert as `active`; `on-end` → executes pending ops **before** marking `ended` (imports `cc_session.delete_after_end`/`archive_after_end` lazily). |
| `cc_session.py` | CLI and the session state machine (archive/delete/cancel/setup, failure recording, config validation such as "archiveDir and trashDir must be different"). |

## Session state machine (the core invariant)

Statuses: `active` → `ended`, `pending-archive` → `archived`, `pending-delete` → `deleted`; failure paths `failed-archive` / `failed-delete` (with `last_error` recorded); `missing` when the transcript file is gone.

Rules that must be preserved when editing this code:

- An `active` session is never archived/moved immediately: it becomes `pending-archive`/`pending-delete` and the actual file operation happens in `SessionEnd` (`archive_after_end` / `delete_after_end`). Exception: `archiveCurrentSessionMode = copy` copies an active transcript immediately without changing status flow.
- `handle_end` checks pending states **before** writing `ended` — a plain "mark ended" upsert must not clobber a pending operation. (This ordering was the subject of fixes `de0691a`/`e3faca3`.)
- `reconcile_stale_sessions` runs on every `SessionStart` for all sessions except the current one, converting stale `active` entries to `ended` — this is the crash/recovery path; preserve it.
- `delete` refuses a `pending-archive` session ("archive cancel first"); pending ops are cancellable (`archive cancel` / `delete cancel` → back to `active`, clearing `delete_mode` and `last_error`).
- All index mutations happen inside `with locked_index():` — never read-modify-write the index without the lock.
- OSError during archive/delete sets `failed-*` status + `last_error` and re-raises as `SystemExit` with a message.

## External state (outside this repo)

Everything persistent lives in `~/.claude/plugins/session/state/`: `config.json`, `session-index.json`, `session-index.lock`, and quarantine files. Default directories used for transcripts: `~/.claude/projects/${project_slug}/.archive/sessions` and `.../.trash/sessions`. `${project_slug}` must be passed **quoted** in shell commands so the shell does not expand it.

## Platform / compatibility constraints

- `index_store.py` uses `fcntl` → POSIX only (macOS/Linux); do not introduce Windows-only assumptions.
- `now_iso()` keeps the `datetime.now(timezone.utc)` spelling with `# noqa: UP017` deliberately for older-Python compatibility (see commit `2bf6809`); ruff config is otherwise strict (`E,F,W,I,N,UP,B,SIM`, line length 120, target py311).
- Scripts use flat top-level imports (`from index_store import ...`) and are executed in place; there is no package install. `tests/conftest.py` and `pyproject.toml` (`pythonpath = ["plugins/session/scripts"]`) both add `scripts/` to `sys.path` — keep new modules flat in that directory.

## Tests

The suite lives at the repo root ([../../tests/](../../tests/)), not inside this plugin, and covers **only** these scripts (CI requires ≥90 % coverage of `plugins/session/scripts`):

| Test file | Covers |
|---|---|
| `test_cc_session.py` (59 tests) | CLI state machine: archive/delete/cancel/setup, failure modes, config validation. |
| `test_session_hook.py` (15 tests) | `handle_start`/`handle_end`, stale-session reconciliation, pending-op execution at end. |
| `test_index_store.py` (22 tests) | Locking, atomic save, corrupt-index quarantine, lookups. |
| `test_config.py` (7 tests) | Defaults/merge/save. |
| `test_path_utils.py` (16 tests) | Slug, template expansion, filenames, dedupe. |

Both hook and CLI test modules isolate state by patching `config.state_dir` **and** `index_store.state_dir` to `tmp_path` — mirror that pattern for any new state access. Focus runs: `pytest tests/test_index_store.py`, `pytest tests/test_cc_session.py -k pending`.

## What changes together

- A new session status or pending operation: `cc_session.py` (state machine) + `session_hook.py` `handle_end` dispatch + `tests/test_cc_session.py` / `tests/test_session_hook.py`.
- A new skill: `skills/<name>/SKILL.md` with `allowed-tools: Bash(python3 ${CLAUDE_SKILL_DIR}/../../scripts/cc_session.py *)` and any new subcommand in `cc_session.py build_parser()`; also update `README.md` here and, if it changes plugin behavior, the description in `.claude-plugin/plugin.json` and the marketplace entry in `../../.claude-plugin/marketplace.json`.
- Local dev loop: `claude --plugin-dir ./plugins/session` (from repo root), then `/reload-plugins`.
