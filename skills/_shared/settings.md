# Settings — `.claude/yrsdd.md` in the user's project

> Reference only, not a skill. Created by `/yrsdd:start` on first use, tuned by `/yrsdd:config`.
> Skills read it at the start and never overwrite existing values.

```md
---
artifact_language: ru        # language of the DOCUMENTS the plugin writes (any language tag)
default_level: ask           # ask | lite | mvp | full
max_review_rounds: 3         # review rounds per task before asking the human
auto_commit: per_task        # per_task | off
branch_strategy: feature     # feature (create feature/<slug> if on the default branch) | current
model_test_author: sonnet    # haiku | sonnet | opus | inherit | full model id
model_implementer: sonnet
model_reviewer: opus
model_debugger: opus
cmd_test: ""                 # empty = autodetect
cmd_lint: ""
cmd_typecheck: ""
---
```

## `artifact_language`

- Applies to **prose in documents the plugin creates**: `brief.md`, `spec.md`, `design.md`,
  `state.md` prose, `_log/`, `_review/`, `changelog.md`, task cards, review findings.
- Does **not** apply to: the plugin's own instructions (always English), code, identifiers,
  commit messages (English), front-matter keys, section headings, verdict literals
  (`PASS`, `CHANGES REQUESTED`, `REVIEW_CLEAN`), file names, statuses.
- Talk to the user in the language of their message. If `artifact_language` differs, still write
  documents in `artifact_language`.
- An existing document keeps its language (never retro-translate).
- Agents get the language in their dispatch prompt: "Write prose in <language>."

## Command detection (when `cmd_*` are empty)

Check in order, use the first hit, print what was detected:
1. `package.json` scripts (`test`, `lint`, `typecheck` / `tsc`), 2. `Makefile` targets,
3. `pyproject.toml` / `pytest.ini` / `ruff` / `mypy`, 4. `go.mod` (`go test ./...`, `go vet ./...`),
5. `Cargo.toml` (`cargo test`, `cargo clippy`), 6. `*.csproj` / `*.sln` (`dotnet test`).
Nothing found → ask the user once and store the answer in `cmd_*`.

## Project instructions win

If the project has a `CLAUDE.md` (or similar) with rules about language, commands, style or
structure, those rules override this plugin's defaults. Read it before writing code.
