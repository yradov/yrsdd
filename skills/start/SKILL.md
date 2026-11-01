---
name: start
description: >
  Use to begin a new feature or task under yrsdd. Triggers on "start {task}", "new feature",
  "yrsdd {task}", "/yrsdd:start", "начни фичу", "новая задача". Creates the project settings file
  on first use, proposes a level (lite / mvp / full), asks the user to confirm it, creates
  docs/features/<slug>/state.md, and hands off to analyze.
argument-hint: "<slug from the roadmap | short task description> [--level lite|mvp|full]"
---

# yrsdd:start

Entry point for **one step** (a "feature" below is a roadmap step). Creates
`docs/features/<slug>/` and fixes the level. Shared rules:
[`../_shared/levels.md`](../_shared/levels.md), [`../_shared/state.md`](../_shared/state.md),
[`../_shared/settings.md`](../_shared/settings.md).

## Protocol

1. **Settings.** If `.claude/yrsdd.md` is missing, create it with the defaults from
   `../_shared/settings.md` (body: one line per key explaining it). Ask **one** question first if
   the user's message language is not Russian: "Language for documents?" and store it as
   `artifact_language`. Never overwrite an existing file. Say in one line that it exists and that
   `/yrsdd:config` changes it.
2. **Slug and roadmap.** Create `docs/` and `docs/features/` if missing.
   - Argument is a slug (or step number) that exists in `docs/roadmap.md`: use that step. Its
     description is the task text (copy it verbatim into the first log line, with the Source),
     its `Size` gives the suggested level. If any step in `Deps` is not `shipped`, warn and ask
     (Continue anyway / Start the dependency first). Set the step's status to `in-progress` in the
     roadmap and bump `updated`.
   - No argument and a roadmap exists: list the steps with status `idea` whose deps are all
     `shipped` and ask which one to start (`AskUserQuestion`).
   - Argument is a free-form description (no roadmap, or no matching step): derive a slug
     (lowercase kebab-case, 2–4 words), show it, and work without a roadmap. Suggest
     `/yrsdd:roadmap` once if the task looks like several features.
   If `docs/features/<slug>/` already exists and its `stage != done`, stop and point to
   `/yrsdd:resume <slug>`.
3. **Level.** Use `--level` if given, else `default_level` from settings if it is not `ask`.
   Otherwise take the step's size (`XS`/`S` → lite, `M` → mvp, `L`/`XL` → full) or infer from the
   description, and confirm with `AskUserQuestion` (one question, recommended option first, each
   option with the artifacts it implies):
   - `lite` — small change, under ~1 day.
   - `mvp` — real feature, a few days, one contributor.
   - `full` — risky or long-lived: several modules, a contract, a data migration.
   Signals for `full`: schema or public API change, security or money, several modules, unclear
   requirements. Signals for `lite`: one file or function, no new behaviour contract, throwaway.
4. **Project facts.** Read `CLAUDE.md` / README / the manifest once. Detect the test, lint and type
   commands (per `settings.md`), print them, and store non-trivial ones in `.claude/yrsdd.md` if
   the `cmd_*` keys were empty and detection was ambiguous.
5. **Branch.** If `branch_strategy: feature` and the current branch is the default one, create
   `feature/<slug>` (tell the user). Do not touch uncommitted work: if the tree is dirty, ask.
6. **Create `state.md`** with `stage: analyze`, the level, `current_task: null`, a first log line
   with the original task description verbatim, and `Next action: /yrsdd:analyze <slug>`.
7. **Handoff** (format in `../_shared/state.md`). Next: `/yrsdd:analyze <slug>`; `/clear` optional.

## Anti-patterns

- Creating a `full` folder for a one-line change. Offer a lower level when in doubt.
- Skipping the level question because "it is obvious". The level changes what gets written.
