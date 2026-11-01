---
name: ship
description: >
  Use to close a feature after review: re-run the gate, confirm the outcomes really work, write
  the changelog and pull request text, and mark the feature done. Never merges. Triggers on
  "ship {slug}", "prepare PR for {slug}", "/yrsdd:ship".
argument-hint: "<slug>"
---

# yrsdd:ship

Shared rules: [`../_shared/levels.md`](../_shared/levels.md), [`../_shared/state.md`](../_shared/state.md).

## Gate

`stage` is `ship`. `mvp` / `full` need the latest review record with `PASS`. Otherwise name the
skill to run. Open `blocked` tasks → list them and ask whether to ship without them.

## Protocol

1. **Re-run the gate** on `HEAD`: tests, lint, types. Red → stop and hand back to `implement`.
2. **Verify for real.** Tests are not enough: run the app, script or command the feature is about
   (the `run` skill if the project has one) and check each criterion's observable outcome that a
   human can check. List what could not be checked (device, production data) as a checklist in
   the changelog; do not claim it.
3. **Write the documents** in `artifact_language`:
   - `lite`: a summary section in `state.md` (what changed, how to run, known gaps).
   - `mvp` / `full`: `changelog.md` (what and why, criteria satisfied, tasks and commits, known
     gaps, open checklist) and `pr-body.md` (summary, test plan with the commands run, links to
     spec, design and review record). `full` also: list every document touched/needed
     (README, docs, ADR status) and update stale ones.
4. **PR.** Propose the exact command for the forge in use (`gh pr create --title … --body-file
   …`). Run it only if the user says so. PR text ends with the project's attribution rules if
   they exist. Never merge, never push to the default branch.
5. **Close.** `state.md`: `stage: done`, final log line, `Next action: none`. If `docs/roadmap.md`
   has this slug, set its status to `shipped` and bump `updated`. Handoff block, plus the steps
   the roadmap now makes available (deps all `shipped`) with `/yrsdd:start <slug>`.

## Anti-patterns

- Declaring success from green tests alone.
- Hiding the "not verified" checklist.
