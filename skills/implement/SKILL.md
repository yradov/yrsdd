---
name: implement
description: >
  Use to build a planned feature task by task with strict TDD and a review gate. For each task:
  failing test first (RED), minimal code (GREEN), gate (tests, lint, types), then an independent
  review. PASS commits the task and moves to the next; CHANGES REQUESTED sends it back for rework.
  Triggers on "implement {slug}", "build {slug}", "continue {slug}", "/yrsdd:implement".
argument-hint: "<slug> [--task T03]"
---

# yrsdd:implement

The execution loop. It is the same on every level. Shared rules:
[`../_shared/levels.md`](../_shared/levels.md), [`../_shared/state.md`](../_shared/state.md),
[`../_shared/settings.md`](../_shared/settings.md).

## Gate

`tasks.json` exists and parses, and the criteria document exists (`brief.md` or `spec.md`).
Otherwise: "run `/yrsdd:plan <slug>` first". Follow the resume protocol in `../_shared/state.md`:
if `current_task` and `task_phase` are set, continue that phase instead of picking a new task.

## Setup (once per run)

1. Read settings; resolve the commands `cmd_test`, `cmd_lint`, `cmd_typecheck` (detection order in
   `../_shared/settings.md`). Print them as a one-line banner with the level, models and
   `max_review_rounds`.
2. Check the working tree. Uncommitted changes that are not from this feature → ask the user.
   Record `base = HEAD` for the task in the log.
3. Validate `tasks.json`: no dependency cycles, all `deps` exist. Order tasks topologically,
   keeping the plan order where free.

## The loop (per task)

With `--task Txx`: run only that task (its `deps` must be `done`, else say which are not), then
stop after **Record** and hand off. Otherwise pick the first task with `status: todo` whose
`deps` are all `done`. None left, but some are
`blocked` or `todo` → report and stop. None left at all → go to **Finish**.

Set `status: in_progress`, `current_task`, `review_round: 0` in `tasks.json` and `state.md`.

### 1. RED  — `task_phase: red`
Dispatch `yrsdd:test-author` (model `model_test_author`). Prompt contains: task fields, feature
folder, test command, language. Wait for its report.
- `GOOD RED` → continue. Quote the failing line in the log.
- `FALSE-PASS: behaviour already exists` → verify, mark the task `done` with a note, skip to **Record**.
- `BAD RED` after its own retry, or `Gaps` that block the criterion → ask the user (the criterion
  may be unclear); never continue to GREEN on a test that does not fail for the right reason.

### 2. GREEN + GATE  — `task_phase: green`, then `gate`
Dispatch `yrsdd:implementer` (model `model_implementer`) with the task, the failing line and the
gate commands. Its report must say `GREEN: yes` and a clean `Gate`.
- Gate not clean or `GREEN: no` → escalation ladder (stop at the first step that works):
  1. **Retry** the implementer once with the failure output.
  2. **Diagnose.** Dispatch `yrsdd:debugger` (model `model_debugger`) with the failing output,
     the task, the test command and a summary of what was tried. Save its report in `_log/Txx.md`
     (or the state log on lite). Then act on its `Verdict`:
     - `CODE` / `ENVIRONMENT` → re-dispatch the implementer with the root cause and fix plan.
     - `TEST` → the test is wrong: send it back to `yrsdd:test-author` with the evidence, then
       redo GREEN. Never let the implementer edit the test.
     - `CRITERION` / `UNKNOWN` → ask the user now (the spec may be wrong or unclear); do not loop.
  3. **Stronger model.** Still failing → raise the implementer to `model_reviewer`'s model once.
  4. **Give up cleanly.** Still failing → `status: blocked`, record the reason, ask the user
     (split the task / change the criterion / stop). Never edit or loosen the test to get past
     it. `NEEDS-HUMAN` from any agent → ask the user immediately.
- Re-run the gate yourself once (do not trust the report blindly): tests, lint, types.

### 3. REVIEW  — `task_phase: review`
Dispatch `yrsdd:reviewer` (`mode: code`, model `model_reviewer`, scope: this task, diff from the
`base` recorded for it to the working tree). It is a clean context: do not paraphrase the code to
it, only point to the feature folder, task id and base.
- `PASS` → go to **Record**. Low-severity notes go into the task's `_log` (or the state log on lite).
- `CHANGES REQUESTED` → **Rework**.

### 4. REWORK  — `task_phase: rework`
1. `review_round += 1`; `rounds += 1` in `tasks.json`. Write the findings into `_log/Txx.md`
   (or the state log on lite) so a reset does not lose them.
2. If `review_round > max_review_rounds` → stop the loop for this task and ask the user with
   `AskUserQuestion`: **Keep going** (reset the counter) / **Split the task** (edit `tasks.json`)
   / **Accept with known issues** (record them as open questions) / **Stop**.
3. Dispatch `yrsdd:implementer` in rework mode with the findings. It fixes, adds a failing test
   first for any behaviour bug, and reruns the gate.
4. Back to step 3 (REVIEW). Do not skip the re-review.

### 5. Record  — task done
1. Commit (if `auto_commit: per_task`): only this task's files, message in English
   `<type>(<slug>): <what>` + trailers `YR-Task: Txx` and one `YR-AC: AC-xx` per criterion.
   Follow the project's commit rules from `CLAUDE.md` if they differ. Never skip hooks.
2. `tasks.json`: `status: done`. `state.md`: `current_task: null`, `task_phase: null`, log line,
   `Next action`.
3. `mvp` / `full`: write `_log/Txx.md` (template in `../_shared/state.md`).
4. Loop to the next task.

Update `state.md` front matter at **every** phase change, before dispatching the next agent.

## Finish

All tasks `done`:
- Set `stage: review` (`mvp`, `full`) or `stage: ship` (`lite`), log line.
- Handoff (format in `../_shared/state.md`): what was built, tasks with the number of review
  rounds each, commits, gate results. Next: `/yrsdd:review <slug>` (mvp, full) or
  `/yrsdd:ship <slug>` (lite). `/clear` recommended before review: the reviewer reads from disk.

## Stopping early

If the user stops, or the context is nearly full: finish the current phase boundary, write
`state.md` (phase, round, next action), and say exactly how to continue:
`/yrsdd:resume <slug>`.

## Anti-patterns

- Code before a seen-failing test.
- Weakening a test, or moving to the next task on `CHANGES REQUESTED`.
- Letting the implementer commit, or committing files from other tasks.
- Reviewing in the same context that wrote the code: always dispatch the reviewer agent.
- Keeping progress only in the conversation: `state.md` and `tasks.json` are the truth.
